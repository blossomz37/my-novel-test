# Lessons Learned: Git in Claude Cowork & Claude Code

Notes from a learning session using Claude Code on a fresh repo. Specific to how Claude Cowork and Claude Code interact with git — not general git tips.

## 1. Claude works in a worktree, not your main folder

When you start a Claude Code session, the assistant is spawned inside a **git worktree** — a second checkout of your repo, on its own branch — at:

```
/path/to/your-repo/.claude/worktrees/<some-name>/
```

Your usual project folder (`/path/to/your-repo/`) is a separate checkout, on `master`. Both share the same `.git` history, but each has its own working copy of the files.

This is by design: Claude can edit and commit freely without ever touching the files you're working on.

## 2. Files don't appear in your main folder until you merge

The most common "wait, where did my file go?" moment.

When Claude commits on its worktree branch, those commits exist in your repo's history immediately — but the files only appear at `/path/to/your-repo/` after you (or Claude) merge the worktree branch into `master`.

Run this to confirm Claude's commits exist on a sibling branch:

```bash
git log --oneline --all --graph
```

You'll see a single line of commits with the worktree branch ahead of `master`. To bring them in:

```bash
cd /path/to/your-repo
git merge --ff-only claude/<branch-name>
```

After that, the files appear in your main folder.

## 3. You don't need a pull request for local work

A pull request is a **GitHub** concept — a discussion/review wrapper around a merge, used when collaborating through a remote repo.

For solo, local Claude work, `git merge` is the whole story. PRs only enter the picture if:

- You push the worktree branch up to GitHub
- You want a review step before the merge

## 4. Use `--ff-only` for the merge

```bash
git merge --ff-only claude/<branch-name>
```

The `--ff-only` flag tells git: "Only do this if it's a clean fast-forward — don't create a merge commit." If the situation isn't a fast-forward, git stops and tells you, instead of silently producing a merge commit you didn't want.

For Claude's typical worktree workflow, fast-forwards are the normal case.

## 5. Stale lock files happen — quit GitHub Desktop

A recurring annoyance: errors like

```
Unable to create '.git/index.lock': File exists
Unable to create '.git/HEAD.lock': File exists
```

These come from git's locking mechanism, which prevents two git processes from writing to the index at the same time. If a process gets killed mid-operation, the lock file survives and blocks the next git command.

The most common culprit on macOS: **GitHub Desktop scanning your repos in the background.** Even when idle, it can hold or leave behind lock files when Claude Code's worktree is also touching the repo.

Fix:

- Quit GitHub Desktop while you're working with Claude Code
- Or remove the repo from GitHub Desktop's sidebar entirely

## 6. Don't reflexively delete a lock file — verify it's stale first

Before removing `.git/index.lock` or `.git/HEAD.lock`, check that no real git process owns it:

```bash
ps aux | grep '[g]it '
```

If nothing comes back (and GitHub Desktop is closed), the lock is stale and safe to remove:

```bash
rm /path/to/your-repo/.git/index.lock
```

If a real git process IS running, stop and figure out what it's doing — don't kill its lock.

## 7. Add `.claude/` to `.gitignore`

The `.claude/worktrees/` folder lives inside your project directory and will show up as untracked in `git status` forever if you don't ignore it.

Add to `.gitignore`:

```
# Claude Code
.claude/
```

This keeps `git status` clean without committing anything Claude-specific into your project history.

## 8. Plan files should live in your repo, not `~/.claude/plans/`

When Claude Code goes into Plan Mode, it writes the plan file to `~/.claude/plans/<slug>.md` by default — a global config directory you'll never look at again.

Better: relocate plan files to the **root of the project workspace** (next to `README.md` and `AGENTS.md`) and commit them. They become part of the project's history, visible alongside the work, and easy to find later.

## 9. The collaboration flow, in one mental model

The whole Claude Code + git interaction reduces to four steps:

1. Claude works in its worktree on branch `claude/<name>` and commits there
2. You (or Claude) `git merge --ff-only claude/<name>` from your main checkout
3. Files appear at `/path/to/your-repo/`
4. Optionally remove the worktree later: `git worktree remove .claude/worktrees/<name>`

That's it. Once the worktree model clicks, every "weird" Claude Code git behavior makes sense.

## 10. Reusable shortcut: a fast-forward workflow

If you find yourself running the merge step repeatedly, save the steps as a workflow file (e.g. `.agents/workflows/fast-forward.md`) so Claude can execute the entire handoff in one tool call: identify the worktree branch and main repo path, attempt the fast-forward, handle stale locks if needed, verify, and report.

This turns the end-of-session merge from a multi-step exchange into a single command.

## TL;DR

- Claude works on a separate branch in a worktree — files don't appear in your main folder until you merge
- Use `git merge --ff-only claude/<branch>` from your main checkout to bring Claude's work in
- No PR needed for local work
- Quit GitHub Desktop to avoid stale lock files
- Verify locks are stale (no live git process) before deleting them
- Ignore `.claude/` in `.gitignore`
- Keep plan files at the repo root, not in `~/.claude/plans/`
