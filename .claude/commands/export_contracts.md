---
disable-model-invocation: true
---
# Export Contracts

Agent-executed upstream publish of a tool's interface contracts to the kit's store (RULES.md §11).
Contracts flow **upstream** (tool → kit) — this is **not** `GEN-CLAUDE.sh update` (which is
downstream-only). Publishing an interface presupposes it has been **declared**: an empty or
partial `docs/contracts/<repo>/` means surfaces are undeclared, not that there is nothing to
export — declare them first (Step 0 of the tool-side variant, via `/contracts`).

## Pull model — run in the kit meta-repo (recommended)

A tool-repo agent cannot write across the workspace boundary (RULES.md §5), so aggregation runs in
the kit. For each in-scope tool repo (colocated + granted):
1. **Gate** — run `/check_contracts` against the tool's `docs/contracts/<repo>/`; **abort on
   any FAIL** (never publish a stale / invalid / secret-bearing contract). If a tool's contracts
   are missing or incomplete, the kit **does not author them** — report the coverage gap back to
   the tool (its agent runs `/contracts` to declare them), then re-run.
2. **Aggregate** — copy `docs/contracts/<repo>/` verbatim into the kit's `contracts/<repo>/`.
3. **Index** — regenerate `contracts/registry.json`: for every contract record
   `{repo, contract, version, kind, format, ingestors, checksum, samples[], updated}`.
4. **Publish (RULES.md §5)** — output the git commit + push sequence for the operator to run
   (Claude never runs git): `docs(contracts): sync <repo> contracts`.

## Tool-side variant — run in a tool repo

0. **Identify + declare (do this FIRST).** An empty/partial `docs/contracts/<repo>/` usually means
   real surfaces are undeclared, not that there is nothing to export. Enumerate the tool's
   externally-consumed output surfaces — **output / DB schema, machine / HTTP API, exported record
   files / result artifacts, log formats** — and for any real surface lacking a
   `docs/contracts/<repo>/<surface>/`, **scaffold and declare it via `/contracts`** (a tool-author
   action, distinct from kit-authoring — the tool declaring its OWN interface is required).
1. **Validate** — run `/check_contracts`; abort on any FAIL.
2. **Stage** `docs/contracts/<repo>/` and emit the exact kit-side steps for the operator to apply,
   since the agent cannot reach across repos itself.

## Rules

- Refuse to export if `/check_contracts` fails.
- The kit **stores** contracts; it never **authors** them (help-improve only, via `/contracts`,
  when explicitly asked).
- Upstream only — `GEN-CLAUDE.sh update` distributes the *tooling* (these commands, the `/contracts`
  skill, the `contract.yml` scaffold), never the contracts.
