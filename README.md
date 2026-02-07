# Ralph

![Ralph](ralph.webp)

Ralph is an autonomous AI agent loop that runs [Claude Code](https://docs.anthropic.com/en/docs/claude-code) repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

[Read my in-depth article on how I use Ralph](https://x.com/ryancarson/status/2008548371712135632)

## Quick Start

Prerequisites: [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (`npm install -g @anthropic-ai/claude-code`), `jq`, a git repo for your project.

```bash
# 1. Install skills
git clone https://github.com/foxytanuki/ralph.git
cp -r ralph/skills/prd ~/.claude/skills/
cp -r ralph/skills/ralph ~/.claude/skills/
cp -r ralph/skills/ralph-init ~/.claude/skills/

# 2. Initialize (run in your project)
cd /path/to/your-project
```
```
/ralph-init
```
```
# 3. Create & convert PRD
/prd [your feature description]
/ralph convert tasks/prd-[feature-name].md to prd.json

# 4. Run
./scripts/ralph/ralph.sh
```

That's it. Ralph will loop through each user story, implement it, and commit.

## How It Works

1. **`/ralph-init`** sets up `scripts/ralph/` in your project (`ralph.sh`, `CLAUDE.md`, `progress.txt`)
2. **`/prd`** generates a structured PRD with clarifying questions and quality review
3. **`/ralph`** converts the PRD to `prd.json` with prioritized user stories
4. **`ralph.sh`** spawns fresh Claude Code instances in a loop, one story per iteration

Each iteration picks the highest priority incomplete story, implements it, runs quality checks, commits, and updates `prd.json`. When all stories pass, Ralph exits.

### Each Iteration = Fresh Context

The only memory between iterations is:
- Git history (commits from previous iterations)
- `progress.txt` (learnings and context)
- `prd.json` (which stories are done)

### Key Design Principles

- **Small stories** — Each story must fit in one context window. "Add a filter dropdown" is good. "Build the entire dashboard" is too big.
- **CLAUDE.md updates** — Ralph writes learnings to `CLAUDE.md` files so future iterations (and humans) benefit from discovered patterns and gotchas.
- **Feedback loops** — Typecheck, tests, and CI must stay green. Broken code compounds across iterations.
- **Review checkpoints** — Stories with `reviewAfter: true` trigger a code review before continuing.
- **Stop condition** — When all stories have `passes: true`, Ralph outputs `<promise>COMPLETE</promise>` and the loop exits.

## Key Files

| File | Purpose |
|------|---------|
| `ralph.sh` | Bash loop that spawns fresh Claude Code instances |
| `CLAUDE.md` | Prompt template for Claude Code |
| `prd.json` | User stories with `passes` status |
| `progress.txt` | Append-only learnings for future iterations |
| `skills/` | Skills for PRD generation, conversion, and init |

## Flowchart

[![Ralph Flowchart](ralph-flowchart.png)](https://snarktank.github.io/ralph/)

**[View Interactive Flowchart](https://snarktank.github.io/ralph/)**

## Discord Notifications

```bash
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..." ./scripts/ralph/ralph.sh
# or
./scripts/ralph/ralph.sh --webhook "https://discord.com/api/webhooks/..."
```

Also configurable via `RALPH_WEBHOOK_URL` environment variable.

## Debugging

```bash
cat prd.json | jq '.userStories[] | {id, title, passes}'  # story status
cat progress.txt                                            # learnings
git log --oneline -10                                       # history
```

## Alternative Setup Methods

### Plugin Marketplace

If you prefer using Claude Code's plugin system instead of copying skills manually:

```
/plugin marketplace add /path/to/ralph
/plugin install ralph-skills@ralph-marketplace
```

Skills installed this way are namespaced (e.g., `/ralph-skills:prd`).

### Manual File Copy (without skills)

If you only want the runner without installing skills:

```bash
mkdir -p scripts/ralph
cp /path/to/ralph/ralph.sh scripts/ralph/
cp /path/to/ralph/CLAUDE.md scripts/ralph/
chmod +x scripts/ralph/ralph.sh
```

Note: This only sets up the loop runner. `/prd` and `/ralph` skills will not be available — you'll need to create `prd.json` manually (see `prd.json.example`).

## Archiving

Ralph automatically archives previous runs when you start a new feature (different `branchName`). Archives are saved to `archive/YYYY-MM-DD-feature-name/`.

## References

- [Geoffrey Huntley's Ralph article](https://ghuntley.com/ralph/)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
