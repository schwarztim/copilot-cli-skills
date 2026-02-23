# Copilot CLI Skills

A collection of utility skills for [GitHub Copilot CLI](https://githubnext.com/projects/copilot-cli/).

## Skills

### `/fork` — Session Forking

Clones the current Copilot CLI session into a new independent session with full context. Both sessions continue independently without interfering with each other.

**Trigger:** `/fork` or "fork this session"

**What it does:**
1. Generates a fresh UUID for the new session
2. Copies the entire session state (checkpoints, plan, artifacts)
3. Copies the resume command to clipboard

**Requirements:** macOS (`pbcopy`, `uuidgen`)

---

### `/copy` — Clipboard Copy

Copies the most recent substantive response to the macOS clipboard. Strips markdown formatting for clean plain text output.

**Trigger:** `/copy` or "copy the last response"

**What it does:**
1. Takes the last meaningful response (not tool output)
2. Strips markdown syntax (`**`, `##`, `-`, etc.)
3. Pipes clean text to clipboard via `pbcopy`

**Requirements:** macOS (`pbcopy`)

## Installation

```bash
# Clone the repo
git clone git@github.com:schwarztim/copilot-cli-skills.git

# Symlink skills into your Copilot CLI skills directory
ln -sf "$(pwd)/copilot-cli-skills/skills/fork" ~/.copilot/skills/fork
ln -sf "$(pwd)/copilot-cli-skills/skills/copy" ~/.copilot/skills/copy
```

Or copy directly:

```bash
cp -r skills/fork ~/.copilot/skills/
cp -r skills/copy ~/.copilot/skills/
```

## Adding New Skills

Each skill lives in its own directory under `skills/` with a `SKILL.md` file that defines:

- **Frontmatter** — name, description, compatibility metadata
- **When to Use** — trigger phrases and conditions
- **Execution Steps** — what the skill does
- **Critical Rules** — guardrails and constraints

## License

MIT
