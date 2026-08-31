---
disable-model-invocation: true
---
# Restore Chat

Restore committed chat sessions from `.claude/sessions/` into the Claude Code
projects folder so a new Claude Code session can pick up the history.

## What it does

1. Detect the current workspace (absolute path of the repo)
2. Compute the Claude Code encoded path: `/` → `-`
3. Source: `.claude/sessions/*.jsonl` in the repo
4. Destination: `~/.claude/projects/<encoded-path>/`
5. Report how many files were copied

This works regardless of where the repo was cloned — the destination path is
computed from `pwd`, not hardcoded.

## Shell

```bash
WORKSPACE=$(pwd)
ENCODED=$(echo "$WORKSPACE" | tr '/' '-')
SRC="$WORKSPACE/.claude/sessions"
DEST="$HOME/.claude/projects/$ENCODED"

if [ ! -d "$SRC" ]; then
    echo "ERROR: No .claude/sessions/ folder in this repo"
    echo "Nothing to restore."
    exit 1
fi

COUNT=$(find "$SRC" -name '*.jsonl' 2>/dev/null | wc -l)
if [ "$COUNT" -eq 0 ]; then
    echo "No session files in .claude/sessions/"
    exit 0
fi

mkdir -p "$DEST"
cp "$SRC"/*.jsonl "$DEST"/

echo "✓ Restored $COUNT session file(s) to $DEST"
echo ""
echo "Start Claude Code from this directory — it should find the sessions:"
echo "  cd $WORKSPACE && claude"
```

## Rules

- Restore is idempotent — re-running overwrites existing local sessions
- If Claude Code is already running, restart it so it picks up the restored files
