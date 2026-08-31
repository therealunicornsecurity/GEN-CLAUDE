---
name: contracts
description: Interface Contracts — identify a tool's external output surfaces (DB schema, machine API, exported artifacts, log formats) and declare/improve their docs/contracts/<repo>/<contract>/ manifests. In a TOOL repo this is REQUIRED tool-authoring; in the kit meta-repo it is improve-only (the kit never invents a tool's contracts). RULES.md §11.
---

# Interface Contracts

A **contract** is one external file/language/structure a tool exposes to outside **ingestors**.
**The contract EXISTS the moment the surface exists** — a DB schema, a machine API, an exported
artifact is a contract whether or not a `contract.yml` has been written yet. The manifest merely
*declares* it. So an **empty `docs/contracts/<repo>/` almost never means "nothing to declare"** —
it means real surfaces are undeclared.

## Know which context you are in

- **In a TOOL repo → IDENTIFY + DECLARE (required).** The tool OWNS its surfaces and MUST
  declare each one. Writing `docs/contracts/<repo>/<surface>/` here is the tool **authoring its
  own interface** — the entire point of the system, *not* a violation.
- **In the kit meta-repo → IMPROVE-ONLY (never invent).** The kit hosts + indexes contracts; it
  MUST NOT fabricate a contract on a tool's behalf. Here `/contracts` only helps *improve* a
  contract a tool already declared (e.g. while reviewing or aggregating). **"The kit never
  authors" is scoped to THIS context** — it never means "don't declare."

## Step 0 — identify the surfaces (FIRST, in a tool repo)

Enumerate what the tool emits that an external ingestor consumes. **Default surfaces to look for:**

- **output store / DB schema** — a SQLite/Postgres schema a GUI, scheduler, or sibling tool reads
- **machine / HTTP API** — an OpenAPI / proto / JSON response shape a client depends on
- **exported record files / result artifacts** — a report JSON, an export bundle, a pipeline manifest
- **log / capture formats** — anything an external parser consumes

For each real surface **lacking** a `docs/contracts/<repo>/<surface>/`, **scaffold it** (copy the
`contract.yml` scaffold → `docs/contracts/<repo>/<surface>/contract.yml`, fill it in, add a
**sanitized** sample) — a tool-author action. Only then validate + publish.

## Folder — `docs/contracts/<repo>/<contract>/`

- `contract.yml` — the manifest (below)
- `samples/` — version-tagged example files (**synthetic / sanitized — RULES.md §5**)
- `v<N>/` — optional retained older versions (each self-contained)

## `contract.yml`

```yaml
repo: <owning tool>
contract: <kebab surface noun — output-db | report-json | log-format | http-api | ...>
version: "<contract semver, independent of the tool VERSION>"
kind: filetype | language | structure
format: json | jsonl | csv | sqlite | proto | openapi | binary | ...
description: <what it is + which ingestor depends on it>
ingestors: [<known consumers>, ...]     # or [unknown]
produces: <module/command that emits it>
source_ref: [<file(s) whose change means the contract changed>]
checksum: "sha256:<over source_ref — drives /check_contracts currency>"
samples: [{path: samples/v1/<file>, version: "1.0.0"}]
updated: "<YYYY-MM-DD>"
supersedes: "<older version, optional>"
```

## Rules (RULES.md §11)

- **Declare every externally-consumed surface** — a real surface with no contract is a coverage
  gap, and (in a tool repo) declaring it is *your* job.
- **Currency:** keep the contract current with any surface change — bump `version`, refresh the
  `checksum`, add a fresh sample. `/check_contracts` FAILS on a stale checksum.
- **Samples (RULES.md §5):** synthetic or sanitized only — never real secrets or PII.
- **Versioning:** semver — MINOR for additive, MAJOR for breaking; retain breaking predecessors
  under `v<N>/`.
- **Naming:** `<contract>` names the *surface*, not the repo. Name the `ingestors` if known.
- **"Never invent" is kit-scoped:** only the kit meta-repo is barred from fabricating a tool's
  contracts. A tool identifying and declaring its own surfaces is required.
