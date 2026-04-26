# Plan: Add CLAUDE.md pointing to AGENTS.md

## Context
This repo already has an [AGENTS.md](../../AGENTS.md) at the root. Claude Code looks for `CLAUDE.md` for project-level instructions. To avoid duplicating content (and drift between the two files), `CLAUDE.md` should be a thin pointer that names `AGENTS.md` as the source of truth.

## Change
Create `CLAUDE.md` at the repo root with a short note stating that `AGENTS.md` is the source of truth for this project's agent instructions.

Proposed content:

```markdown
# CLAUDE.md

See [AGENTS.md](AGENTS.md) — it is the source of truth for agent instructions in this project.
```

## Critical files
- New: [CLAUDE.md](../../CLAUDE.md)
- Referenced: [AGENTS.md](../../AGENTS.md)

## Verification
- `ls CLAUDE.md` shows the new file
- `cat CLAUDE.md` shows the pointer note
- `git status` shows one new untracked file
