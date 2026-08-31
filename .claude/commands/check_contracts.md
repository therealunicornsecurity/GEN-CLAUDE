---
disable-model-invocation: true
---
# Check Contracts

Agent-executed validation of interface contracts (RULES.md §11). Run in a **tool repo**
(checks `docs/contracts/<repo>/`) or in **the kit** (checks the aggregated `contracts/` store).
Read-only report + gate — no git, no edits to the contracts themselves.

## Coverage — are all surfaces declared? (check this FIRST)

Enumerate the tool's externally-consumed output surfaces — **output / DB schema, machine / HTTP
API, exported record files / result artifacts, log formats.** Any real surface with **no**
`docs/contracts/<repo>/<surface>/` is a **COVERAGE GAP** — an empty/partial `docs/contracts/`
with real outputs is a gap, **not** a clean pass. Report each gap; in a tool repo the fix is to
declare it via `/contracts` (tool-authoring). The kit never declares it for the tool.

## Per contract folder `docs/contracts/<repo>/<contract>/`

1. **Manifest** — `contract.yml` carries the required fields: `repo`, `contract`, `version`,
   `kind` (filetype|language|structure), `format`, `description`, `ingestors` (list, or
   `[unknown]`), `produces`, `source_ref` (list), `checksum`, `updated`. Reject an unknown
   `kind`/`format`.
2. **Currency** — recompute `sha256` over the concatenated `source_ref` files and compare to
   `contract.yml.checksum`. **Mismatch ⇒ FAIL** (the surface changed without bumping the
   contract). This is how "kept current with any local modification" is enforced.
3. **Samples** — if samples are declared: each file exists, **parses in its declared `format`**
   (json/jsonl/csv/sqlite/proto/… — validate structurally), and its `version` matches the
   contract (or the `v<N>/` subdir it lives in). **Sanitization (RULES.md §5, Security — binding):**
   scan every sample for secret-shaped values (passwords, API keys, tokens, private keys, real
   PII) — a sample MUST be synthetic / sanitized; any live secret is a **BLOCKING** failure.
4. **Versions** — multi-version layout (`v<N>/` subdirs) well-formed; current marked; no orphan.
5. **Naming** — `<contract>` is a kebab **surface** noun (not the repo name); ingestors named
   or explicitly `[unknown]`.

## Output

Per-contract **PASS / FAIL** table with reasons. Any FAIL blocks `/export_contracts`; exit
non-zero so it can gate CI / Procedure B. **Report only** — never edit a contract to "fix" it:
that is the tool author's job. The kit / Claude never authors contracts (help-improve only, via
the `/contracts` skill, when explicitly asked).
