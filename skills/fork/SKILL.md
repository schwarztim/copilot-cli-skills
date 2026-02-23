---
name: fork
description: Forks the current session into a new independent session with full context. Creates a copy of the session state under a new ID so both sessions can continue independently without interfering. Use when the user says "/fork" or asks to fork the current session.
license: MIT
compatibility: macOS (requires pbcopy, uuidgen)
metadata:
  author: Tim Schwarz
  version: "2.0"
---

# Fork Session

Creates a **new independent session** by cloning the current session state under a fresh UUID. The new session has full context from the original but won't interfere with it. Copies the resume command for the NEW session to clipboard.

## When to Use

- User types `/fork` or says "fork this session"
- User wants to branch off into a new conversation with full context
- User wants to continue work in a separate terminal without conflicting with the current session

## Execution Steps

1. **Extract the current session ID** from the `<session_context>` block in the system prompt. The session folder path contains the UUID (e.g., `~/.copilot/session-state/69dc2041-92d6-4bc0-89a4-fbd8c0e2aa5b`).

2. **Generate a new session ID and clone the state** using bash:
```bash
NEW_ID=$(uuidgen | tr '[:upper:]' '[:lower:]')
CURRENT_DIR="$HOME/.copilot/session-state/<CURRENT_SESSION_ID>"
NEW_DIR="$HOME/.copilot/session-state/$NEW_ID"
cp -R "$CURRENT_DIR" "$NEW_DIR"
echo "copilot --resume $NEW_ID --yolo" | pbcopy
echo "$NEW_ID"
```

3. **Tell the user** what was created:
```
Forked session: <NEW_ID>
Copied to clipboard:
copilot --resume <NEW_ID>

Original session <CURRENT_ID> is unchanged.
```

## Critical Rules

- Always generate a NEW UUID — never reuse the current session ID
- Copy the full session-state directory so the new session has all checkpoints, plan.md, and files/
- Always use `pbcopy` to copy the NEW session's resume command
- Always display both the new and original session IDs so the user knows they're independent
- The current session ID comes from the system prompt's session_context, NOT from asking the user
