---
disable-model-invocation: true
---
# Backup Chat

Snapshot the current Claude Code session into `.claude/sessions/` so it
can be committed and restored on another machine.

## What it does

1. Detect the current workspace (absolute path of the repo)
2. Compute the Claude Code encoded path: `/` → `-`
3. Source: `~/.claude/projects/<encoded-path>/*.jsonl`
4. Destination: `.claude/sessions/<session-id>.jsonl` in the repo
5. Report how many files were copied and the total size

## Shell

```bash
WORKSPACE=$(pwd)
ENCODED=$(echo "$WORKSPACE" | tr '/' '-')
SRC="$HOME/.claude/projects/$ENCODED"
DEST="$WORKSPACE/.claude/sessions"

mkdir -p "$DEST"

if [ ! -d "$SRC" ]; then
    echo "ERROR: No session folder found at $SRC"
    echo "Are you running this from inside the repo?"
    exit 1
fi

COUNT=$(find "$SRC" -name '*.jsonl' | wc -l)
if [ "$COUNT" -eq 0 ]; then
    echo "No session files found in $SRC"
    exit 0
fi

cp "$SRC"/*.jsonl "$DEST"/
SIZE=$(du -sh "$DEST" | cut -f1)

echo "✓ Backed up $COUNT session file(s) ($SIZE) to .claude/sessions/"
echo ""
echo "To commit:"
echo "  git add .claude/sessions/"
echo "  git commit -m 'chore(sessions): backup chat history'"
```

## Rules

- **Review before commit** — session JSONL contains everything, including any
  secrets, tokens, or sensitive paths that were pasted in the chat
- Session files accumulate — consider pruning old sessions periodically
- Never force-push `.claude/sessions/` — these are append-only history
