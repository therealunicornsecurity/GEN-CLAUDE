---
name: supply_chain_audit
description: Supply-chain malware check — cross-reference resolved dependencies against a known-malware / advisory database. Gated review Step 0 sub-check.
version: 1.0.0
tags: [security, dependencies, supply-chain, malware]
---

# /supply_chain_audit

Cross-reference the repo's **resolved** dependencies against a **known-malware /
advisory database** — OSV, the ecosystem advisory feeds (npm / PyPI), the
ecosystem's own audit tool, or a comparable source — to catch known-malicious
packages (typosquats, hijacked releases, backdoored versions). Runs as a
**gated sub-check of the Code Quality Review Step 0 (DEPS)**, alongside
`/deps_audit` (which stays fully offline: CVE + license only). This skill is the
only one that may talk to an external, authenticated, rate-limited API — so when
the chosen database requires a key it is governed by the gate and secrets rules
below.

## ⛔ GATE — ask for the key; no key ⇒ skip

Some advisory databases are open and keyless; others gate access behind an
authenticated, **rate-limited** token (per-token, per-minute; `429` on exceed).
When the chosen source needs a key:

- When the review reaches Step 0, the agent **asks the operator for the API
  key** (available at `secrets/advisory-api-key`? or provide the path).
- **No key ⇒ SKIP.** The check does not run; report `skipped — no advisory key`.
  **Non-blocking** — a missing key never fails or stops the review.
- **Key present ⇒ run**, spending **one request per package** — N lookups per
  ecosystem (see API). Handing over the key for this run **is** the explicit
  per-run authorization (RULES.md §5.1 rule 5); a prior run's authorization does
  not carry over.

A keyless source (e.g. a public OSV query or an offline advisory dump) skips the
gate entirely — run it directly.

## 🔑 Secrets handling (RULES.md §5.1 — absolute)

- The token is a secret, stored at **`secrets/advisory-api-key`** (placeholder
  `<ADVISORY_API_KEY>`).
- **The agent never reads, echoes, logs, or transmits the token** (§5.1 rules
  1–4). A **consumer** (a small script or a hardened container) reads the file
  **by path** and sets the `Authorization: Bearer` header itself. The agent
  passes `secrets/advisory-api-key`, nothing more.
- **Never paste the token into chat, a tool call, a commit, or a log.** If a
  token value appears in the transcript, treat it as **exposed → rotate**.

## API — the common shape

Many advisory databases expose only a **single-resource** endpoint: check one
coordinate, get one verdict — `Authorization: Bearer`, params for report type ·
resource identifier · ecosystem (opt) · version (opt). **No batch and no
list/feed endpoint** — so the real capability is **one call per package**, and
quota is per-token, per-minute (`429`). Confirm the endpoint and parameters
against the chosen database's current docs before running.

### The consumer is a ONE-SHOT static script — `advisory_check.py`

The agent never calls the API directly and never reads the token. It invokes a
committed static script (this skill's asset), which checks **exactly one
package** and returns its verdict as JSON:

```
python3 advisory_check.py --ecosystem npm --name axios --version 1.18.1 \
                          --token-file secrets/advisory-api-key --params-verified
```

- Reads the token **by path** — the agent never sees the value.
- **One package = one query.** No loop, no cache, no throttle *in the script*.
- **Refuses to call until `--params-verified`** (explicit per-run authorization,
  §5.1). `--dry-run` prints the lookup URL with no token and no network.
- Exit: `0` clean · `2` malicious · `3` error · `4` no token (skip).

### The procedure owns the iteration

`/supply_chain_audit` orchestrates around the one-shot script:

1. Parse lockfiles → resolved `(ecosystem, name, version)` per dependency; dedupe.
2. Run `advisory_check.py` **once per dependency**.
3. **Throttle** between runs for the per-minute quota; back off on `429`;
   optionally cache verdicts so a coordinate is checked once.
4. Aggregate: any exit `2` ⇒ BLOCKING finding; exit `4` on the first run
   (no key) ⇒ SKIP the whole check.

**Against a real project this is N lookups — one per resolved dependency** (a
typical npm tree is hundreds of transitive packages). A single-resource database
has no batch/list endpoint, so N calls cannot be collapsed into one; the
procedure paces them under the rate limit. If the database offers a batch/list
endpoint, use it instead.

## Input — resolved dependency coordinates only

Parse **lockfiles** (resolved graph, not top-level declarations):
`package-lock.json` / `pnpm-lock.yaml` / `yarn.lock` (npm), `poetry.lock` /
`requirements.txt` (PyPI), `go.sum` (Go), `Cargo.lock` (Rust), `composer.lock`,
`Gemfile.lock`, … Extract `(ecosystem, name, version)` tuples; dedupe; group by
ecosystem. With an offline/keyless source, coordinates never leave the machine.

## Output

Each match reported as malicious becomes a supply-chain finding:
`finding_type=supply-chain`, `severity` from the database's severity level, the
coordinate in `affected_resources`, the advisory id + report URL in
`references`, the database's tags/description into `description`/`context`.

```
## Supply-chain Audit: <ecosystem>
### Malicious (BLOCKING)
- <ecosystem>:<package>@<version> — <severity> — <tags> — <advisory-id>
### Skipped
- no advisory key — supply-chain check not run (non-blocking)
### Unverified (quota / API error, key was present)
- <N> packages not checked — reason
### Clean
- <N> packages checked, no hits
```

## Rules

- **A malicious dependency is a BLOCKING finding** — it stops the review at
  Step 0. Resolution: remove/replace the package or pin to a known-clean
  version; re-run.
- **No key ⇒ skip, not fail.** Missing key is a reported, **non-blocking** skip.
- **Key present but API error / quota ⇒ `unverified`**, never silent-clean
  (null ≠ safe); the checked portion still counts.
- **One request per package** — N lookups per ecosystem; never a single
  per-ecosystem call.
- **Report only** — the agent does not auto-remove or auto-pin; the operator
  decides.
