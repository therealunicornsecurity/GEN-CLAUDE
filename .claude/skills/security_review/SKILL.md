---
name: security_review
description: GEN-CLAUDE's Procedure B Step 6 AI security gate — invokes Anthropic's OFFICIAL bundled /security-review on the diff, IN ADDITION to the grep-based /security_scan. Not a reimplementation.
version: 2.0.0
tags: [security, review, ai, official, sast]
---

# /security_review — GEN-CLAUDE wrapper around the official `/security-review`

GEN-CLAUDE does **not** reimplement AI security analysis. This skill **invokes Anthropic's
official, bundled [`/security-review`](https://github.com/anthropics/claude-code-security-review)**
(semantic, diff-scoped vulnerability review) and folds its findings into GEN-CLAUDE's Procedure B
Step 6 gate — **in addition to**, never instead of, the grep-based
[`/security_scan`](../security_scan/SKILL.md).

> The analysis engine is Anthropic's and versions with Claude Code. This file is only GEN-CLAUDE's
> *wiring* — when the official skill runs, alongside what, and what blocks. Bump `version`
> when the wrapper policy changes, not when the upstream engine does.

## Step 6 runs BOTH — additional, not either/or

1. `/security_scan` — GEN-CLAUDE's own grep of `RULES.md §5` + `docs/security.md`. Deterministic. **Blocking.**
2. `/security-review` — the **official Anthropic** bundled skill. Semantic AI review of the
   pending diff (injection, auth/authz, data exposure, crypto, input validation, business
   logic, config, supply chain, code execution, XSS). **Additional. A HIGH finding blocks.**

Two independent gates: a pattern our grep knows, and a vulnerability only semantic analysis
catches. Either one firing stops Step 6.

## Invocation

Run the official skill directly:

```
/security-review
```

It reads the `git diff` of pending changes on the current branch and reports findings with
severity (HIGH / MEDIUM / LOW). **GEN-CLAUDE policy:** a HIGH finding is blocking (same weight as a
`/security_scan` hit); MEDIUM/LOW are advisory. Report only — fixes follow Procedure A (test first).

## ⚠️ GEN-CLAUDE constraint — git is blocked by the hook

`/security-review` needs `git diff`, but GEN-CLAUDE's `.claude/hooks/PreToolCall` refuses all `git`.
Two ways to run it inside an GEN-CLAUDE repo:

- **CI mode (recommended).** `templates/security-review.yml` → `.github/workflows/security-review.yml`
  runs the upstream GitHub Action (`anthropics/claude-code-security-review@main`) in CI —
  outside the hook — on every PR. Needs a `CLAUDE_API_KEY` repo secret.
- **Interactive mode.** The developer runs `/security-review` in a session where git is
  permitted (temporarily allow `Bash(git diff:*)`), or hands the agent the diff to review.

## CI form

```yaml
- uses: anthropics/claude-code-security-review@main
  with:
    comment-pr: true
    claude-api-key: ${{ secrets.CLAUDE_API_KEY }}
    # custom-security-scan-instructions: docs/security.md   # feed GEN-CLAUDE's per-language addendum
    # exclude-directories: tests,docs
```

Inputs: `claude-api-key` (required), `comment-pr` (default `true`), `claude-model`
(default `claude-opus-4-1-20250805`), `exclude-directories`, `custom-security-scan-instructions`,
`false-positive-filtering-instructions`. Outputs: `findings-count`, `results-file`.
**Upstream note:** the Action is *not* hardened against prompt injection — review trusted PRs
only; enable "Require approval for all external contributors" on the repo.

## Rules

- Run BOTH `/security_scan` and `/security-review` at Step 6 — additional, not either/or.
- A `/security_scan` hit **OR** a HIGH `/security-review` finding blocks the step.
- GEN-CLAUDE does not fork or reimplement the analysis — it invokes the official bundled skill.
- Report only — no auto-fix. Each fix follows Procedure A.

## Attribution

Engine: Anthropic's official **[`/security-review`](https://github.com/anthropics/claude-code-security-review)**
— bundled with Claude Code; source MIT. This skill is GEN-CLAUDE's integration only: it decides
*when* the official skill runs (Step 6), *alongside what* (`/security_scan`), and *what blocks* (HIGH).
