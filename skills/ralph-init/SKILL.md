---
name: ralph-init
description: "Initialize Ralph in the current repository. Auto-detects existing setup. Triggers on: setup ralph, init ralph, initialize ralph, ralph init."
user-invocable: true
---

# Ralph Setup

Initialize the Ralph autonomous agent system in the current repository. Auto-detects if already initialized.

---

## Step 0: Detect Existing Installation

Before doing anything, check if Ralph is already set up:

```bash
# Check for ralph directory and key files
ls scripts/ralph/ralph.sh scripts/ralph/CLAUDE.md 2>/dev/null
```

### If BOTH files exist:

Ralph is already initialized. Report the status:

1. Show: "Ralph is already initialized in this project."
2. Check for existing `prd.json`: `ls scripts/ralph/prd.json 2>/dev/null`
   - If exists: show the current project name and branch from prd.json
   - If not: note that no PRD is configured yet
3. Check for existing `progress.txt`: show last few lines if it has content
4. **Suggest next steps** based on state:
   - No prd.json → "Run `/prd` to create a PRD for your feature"
   - prd.json exists with incomplete stories → "Run `./scripts/ralph/ralph.sh` to continue"
   - All stories complete → "All stories are done. Create a new PRD with `/prd` for the next feature"

**Do NOT re-download or overwrite existing files.**

### If files are MISSING:

Proceed to Step 1 (fresh installation).

---

## Step 1: Install Ralph

Execute these commands to set up Ralph:

```bash
# Create the ralph directory
mkdir -p scripts/ralph

# Download Ralph files from GitHub
curl -fsSL https://raw.githubusercontent.com/foxytanuki/ralph/main/ralph.sh -o scripts/ralph/ralph.sh
curl -fsSL https://raw.githubusercontent.com/foxytanuki/ralph/main/CLAUDE.md -o scripts/ralph/CLAUDE.md

# Make executable
chmod +x scripts/ralph/ralph.sh

# Create initial progress file
echo "# Ralph Progress Log" > scripts/ralph/progress.txt
echo "Initialized: $(date)" >> scripts/ralph/progress.txt
echo "---" >> scripts/ralph/progress.txt
```

---

## Step 2: Confirm and Guide

After installation completes, confirm success and guide the user:

1. Verify files exist: `ls -la scripts/ralph/`
2. Show: "Ralph initialized successfully."
3. **Guide to next step:** "Run `/prd` to create a PRD for your feature."

---

## Directory Structure

After setup, you'll have:

```
your-project/
├── scripts/
│   └── ralph/
│       ├── ralph.sh        # Run this to start Ralph
│       ├── CLAUDE.md       # Claude Code instructions
│       ├── prd.json        # Your PRD (create with /prd → /ralph)
│       └── progress.txt    # Progress log (auto-managed)
```

---

## Notes

- The `prd.json` file is NOT downloaded — you create it for your specific feature
- Workflow: `/ralph-init` → `/prd` → `/ralph` → `./scripts/ralph/ralph.sh`
- Discord webhook notifications can be configured via `RALPH_WEBHOOK_URL` environment variable
