# .claude/ Documentation Index

Update after every doc change.

| File | Description | Lines |
|------|-------------|-------|
| `CLAUDE.md` | Workspace law — GEN-CLAUDE rules for Claude Code | - |
| `commands/procedure_a.md` | TDD workflow (8 steps) | - |
| `commands/procedure_b.md` | Code quality review (9 steps) | - |
| `commands/new_tool.md` | Scaffold a new repo via GEN-CLAUDE.sh | - |
| `commands/freeze.md` | Snapshot procedure | - |
| `commands/tag.md` | Version bump + changelog + git tag | - |
| `commands/doc_alignment.md` | Doc↔code alignment sweep (gated) | - |
| `commands/check_contracts.md` | Validate interface contracts (§11) | - |
| `commands/export_contracts.md` | Publish contracts upstream to the kit (§11) | - |
| `commands/backup_chat.md` | Snapshot the Claude session to `.claude/sessions/` | - |
| `commands/restore_chat.md` | Restore a snapshotted session | - |
| `agents/spec_writer.md` | Write `docs/spec/` for a module | - |
| `agents/test_writer.md` | Generate failing unit test from spec | - |
| `agents/code_reviewer.md` | Procedure B steps 6-7 analysis | - |
| `agents/security_auditor.md` | Isolated §5 security audit (pairs with `/security_scan`) | - |
| `skills/commit_format/SKILL.md` | `/commit_format` — commit type reference | - |
| `skills/review/SKILL.md` | `/review` — structured code review | - |
| `skills/refactor/SKILL.md` | `/refactor` — systematic refactoring | - |
| `skills/test_gen/SKILL.md` | `/test_gen` — generate test suites | - |
| `skills/perf_benchmark/SKILL.md` | `/perf_benchmark` — before/after benchmark | - |
| `skills/deps_audit/SKILL.md` | `/deps_audit` — CVE + license gate | - |
| `skills/security_scan/SKILL.md` | `/security_scan` — grep RULES.md §5 forbidden patterns (review step 6) | - |
| `skills/security_review/SKILL.md` | `/security_review` — invokes the official /security-review; runs with /security_scan (review step 6) | - |
| `skills/supply_chain_audit/SKILL.md` | `/supply_chain_audit` — dep malware check (gated) | - |
| `skills/contracts/SKILL.md` | `/contracts` — help improve interface contracts | - |
| `templates/module_spec.md` | `docs/spec/` skeleton | - |
| `hooks/PreToolCall` | Block reads/writes outside workspace + git/gh/curl/wget Bash | - |
| `hooks/PostToolCall` | Warn when secrets appear in tool output | - |
| `hooks/Stop` | Reminder to run tests before closing | - |
| `hooks/Notification` | Append events to `.claude/logs/events.log` | - |
| `settings.json` | Wires the four hooks + denies git/gh/curl/wget in Bash | - |
