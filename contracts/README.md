# Contracts store

Aggregated **interface contracts** (RULES.md §11), namespaced by owning repo:
`contracts/<repo>/<contract>/`. **The kit hosts these for ingestor query; it does not author them.**

Populated **upstream** by the agent-executed `/export_contracts` (tool → kit) — **not** by
`GEN-CLAUDE.sh update` (which is downstream-only). Query the machine-readable index at
[`registry.json`](registry.json): `repo → contract → {version, kind, format, ingestors,
checksum, samples}`.

A tool declares its own contracts in its `docs/contracts/<repo>/`; `/check_contracts` gates
currency + sample validity (and RULES.md §5 sample sanitization) before `/export_contracts`
publishes them here.

*(Empty until the first tool runs `/export_contracts`.)*
