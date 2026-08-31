# Doc Alignment Procedure

**Trigger**: "run doc alignment procedure" (or `/doc_alignment`).

Run the markdown sweep **as a workflow**. Output is a table — **no guessing, no
commentary**. Alignment is a **GATE**: report skew, then proceed only on
operator confirmation.

## Intake — change context (operator runs, Claude consumes)

Claude does NOT run git (RULES.md §5). The operator pastes the output of:

```bash
git --no-pager log -10 --stat
```

The last 10 commits (full messages + modified files) scope what changed recently
→ which code/specs to re-check first.

## What counts as "code" (repo-relative)

The procedure aligns docs against the repo's **source / executable artifacts**.
Define the code surface before the sweep: `src/` and `tests/`.

## Mutation policy — read-only sweep, gated changes

The sweep (steps 0–8) is **READ-ONLY**. Its only product is the **REPORT** (step 9).
**Nothing** is deleted, merged, or fixed, and **no** file is removed, until the
operator confirms the report table.
- code↔doc fixes follow **Procedure A** (doc lands in the same commit as the code).
- never edit pipelines / keys / baselines / decision files to "align" them —
  **diagnose + propose only**.

## Steps (sequential — run as a workflow)

0. **INVENTORY** — read EVERY `*.md` (no sampling — **READ ALL MD FILES**). Tag
   each: `code-spec` | `roadmap` | `decision-record` | `index` | `readme`.
1. **REF doc→code** — every spec names its current code file; the path resolves
   to the right module. Flag misses.
2. **REF code→doc** — every spec'd code file points back to its spec. Flag the gap
   on either side.
3. **AGREEMENT (no skew)** — doc and code say the same thing: the doc reflects
   current behaviour; the code has nothing the doc omits. Drift flagged both ways.
   *"Same thing" = behavioural agreement, not merely a reference existing.*
4. **FRESHNESS** — flag stale docs (renamed/removed modules, dead APIs, superseded
   designs).
5. **DEDUP** — merge docs covering the same subject (propose).
6. **PRUNE** — delete obsolete/superseded docs (propose).
7. **NEW WORK** — everything in the last-10-commits diff (code + tests) is
   documented (spec + roadmap); nothing landed undocumented.
8. **HYGIENE** — flag temp scripts, unused fixtures/data, orphan test files NOT
   referenced by code or tests; confirm every needed test fixture **is** committed.
9. **REPORT** — emit the tables below.

**DECISION FILES ARE HISTORY — never align them.** `DECISIONS.md` / `decisions*` /
roadmap decision logs are append-only and legitimately hold old code patterns
("decisions just stack"). Never realign, dedup, or delete them. Exempt from
steps 1–6.

**The document index is itself a doc.** `docs/index.md` carries the `index` tag in
step 0 — verify it lists every doc and is current (line counts, descriptions,
"Last updated"). A missing or stale index row is skew.

## Rules

- Read ALL md, no sampling.
- Bidirectional spec↔code.
- "Same thing" = behavioural agreement, not just a reference existing.
- Decision files exempt.
- Output = table; no guessing, no commentary.
- Run the sweep as a workflow.
- Never edit pipelines / keys / baselines / decisions to align them (diagnose + propose).
- code↔doc fixes follow Procedure A (doc in the same commit as code).
- Repo cleanup is part of the procedure.

## Output

### Alignment

| Doc | doc→code ref | code→doc back-ref | In sync? | Action | Why |
|-----|--------------|-------------------|----------|--------|-----|

### Repo hygiene

| File | Type | Used by | Action |
|------|------|---------|--------|

### Verdict

- docs aligned?
- skew remaining?
- repo clean?
- undocumented new work?
