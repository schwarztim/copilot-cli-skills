---
name: copy
description: Copies the most recent substantive text response from Copilot to the macOS clipboard using pbcopy. Use when the user says "/copy" or asks to copy the last response. Strips markdown formatting and outputs clean plain text.
license: MIT
compatibility: macOS (requires pbcopy)
metadata:
  author: Tim Schwarz
  version: "1.0"
---

# Copy to Clipboard

Copies the last substantive Copilot response to the macOS clipboard using `pbcopy`.

## When to Use

- User types `/copy` or asks to copy the last response
- User wants clean text on clipboard without terminal word-wrap artifacts

## Usage

When invoked, take your most recent substantive text response (not tool output or meta-commentary), and pipe it to `pbcopy` using a bash command. Strip any markdown formatting (headers, bold, bullet symbols) to produce clean plain text. Confirm with "Copied to clipboard."

```bash
printf '%s' '<response text here>' | pbcopy
```

## Notes

- Only copy the actual content, not tool calls or status messages
- Remove markdown syntax (**, ##, -, etc.) for clean plain text
- Use `printf '%s'` not `echo` to avoid trailing newline issues
- Confirm with a short message after copying
