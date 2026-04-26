---
name: fast-forward
description: Fast-forward the user's master branch (in the main checkout) to the current Claude Code worktree branch tip. Handles stale lock files automatically.
---

# Fast-Forward Workflow

Goal: bring all commits made on the current Claude Code worktree branch into `master` (or whatever the project's main branch is) in the user's main checkout, so files appear in their usual project folder.

This is the standard end-of-task handoff: Claude commits on its worktree branch → this workflow merges those commits into `master` → user sees files in `/path/to/repo/`.

## Steps

Run all of this with as few tool calls as possible. Combine into one or two `Bash` calls where possible.

### 1. Identify the relevant paths

- **Worktree branch** (current branch in this Claude session): `git branch --show-current`
- **Main repo path** (the parent worktree, where the user works): `git worktree list | head -1 | awk '{print $1}'`
- **Main branch name**: typically `master` or `main`. Determine from `git worktree list` (the branch shown next to the main path) — do not assume.

### 2. Attempt the fast-forward

From the main repo path:

```bash
cd <main-repo-path> && git merge --ff-only <worktree-branch>
```

If it succeeds, **skip to step 4**.

### 3. If it fails with a lock-file error

The error looks like: `Unable to create '.git/index.lock': File exists` (or `HEAD.lock`).

Verify the lock is stale before removing:

```bash
ps aux | grep -E '[Gg]it[Hh]ub|[g]it ' | grep -v grep
ls -la <main-repo-path>/.git/*.lock
```

The lock is stale if:
- No live `git` process appears in `ps`
- GitHub Desktop is not running (its background scanning is the most common cause)
- The lock file is 0 bytes and/or has an old timestamp

If stale, remove the offending lock file(s) and retry the merge:

```bash
rm <main-repo-path>/.git/<lockfile>
cd <main-repo-path> && git merge --ff-only <worktree-branch>
```

If a real git process IS running, **stop and ask the user** what they're doing in the other session — do not delete an active lock.

### 4. Push to origin (if a remote is configured)

```bash
cd <main-repo-path> && git remote get-url origin >/dev/null 2>&1 && git push
```

The `git remote get-url origin` check skips the push silently if the repo has no remote (purely local). If `origin` exists, `git push` uploads the just-merged commits.

If the push fails because the upstream branch isn't set, run `git push -u origin <main-branch>` once to set tracking, then plain `git push` works thereafter.

### 5. Verify

```bash
cd <main-repo-path> && git log --oneline -3 && git status
```

Report to the user:
- The new tip commit on the main branch (e.g. `main`)
- That the working tree is clean
- That files are now visible at `<main-repo-path>`
- Whether the push to `origin` succeeded (if a remote is configured)

Keep the report to 2-3 sentences. Use clickable markdown links for any file references.

## Notes

- **Fast-forward only.** Use `--ff-only` so a non-fast-forward situation surfaces as an error rather than silently producing a merge commit. If the merge isn't a fast-forward, stop and ask the user how to proceed.
- **Don't delete the worktree.** That's a separate decision; the user may want to keep working in this session.
- **Push uses default behavior.** Plain `git push` (no flags) pushes only the current branch to its tracked upstream. It will not push other branches like the Claude worktree branch.
