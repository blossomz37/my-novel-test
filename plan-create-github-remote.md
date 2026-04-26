# Plan: Create a GitHub remote for `my-novel-test` and push existing history

## Context

Right now `my-novel-test` is a purely local git repo — every commit lives on the user's laptop and nowhere else. Goals:

1. **Backup** — losing the laptop currently means losing the repo
2. **A place to push from anywhere** — opens the door to GitHub Desktop, the GitHub web UI, future collaboration, etc.
3. **Educational** — the user wants to walk through the *first time you ever connect a local repo to a remote*, with each step explained

This is the natural next step after learning the local git loop. The session has already produced a strong local history (9 commits, including a beginner guide), and pushing that history to GitHub means the lessons-learned artifact is also preserved off-machine.

User answers from clarifying questions:
- **Visibility:** Public
- **Branch name:** Rename local `master` → `main` before pushing (matches modern GitHub default)

## Pre-conditions (already verified)

- `gh` CLI installed at `/opt/homebrew/bin/gh` (v2.78.0)
- Authenticated as GitHub user **`blossomz37`** with `repo` scope
- No `origin` remote currently configured
- Working trees clean on both `~/projects/my-novel-test/` (currently `master` @ `f4a6218`) and the Claude worktree

## What's about to happen — concepts first

A **remote** is just a named URL pointing at another copy of your repo. The conventional name for "the main remote you push to" is `origin`. After this plan runs, `git push` from your machine will upload your commits to GitHub, and `git pull` will download anyone else's.

Four steps:

1. **Rename the local branch** `master` → `main` so it matches what GitHub will use as the default branch. This is purely a local rename; no remote exists yet.
2. **Create the GitHub repo** using `gh repo create`. This is the point where a new, empty repo appears at `github.com/blossomz37/my-novel-test`.
3. **Connect the local repo to the remote** by adding `origin`. After this, your local repo "knows" where to send pushes.
4. **Push the existing history** to `origin/main`, so all 9 commits land on GitHub.

`gh repo create` can do steps 2, 3, and 4 in one shot with the right flags — but the plan splits them so the educational walkthrough is clear.

## Steps

All commands run from `~/projects/my-novel-test/` (the main checkout). The Claude worktree is left alone; it stays on its `claude/unruffled-cohen-755056` branch and isn't pushed to GitHub.

### Step 1 — Rename `master` to `main` (local only)

```bash
cd ~/projects/my-novel-test
git branch -m master main
```

`-m` is the rename flag. Because no remote exists yet, this is a clean, no-consequence operation. `git branch` will now show `* main`.

### Step 2 — Create the GitHub repo (no push yet)

```bash
gh repo create my-novel-test \
  --public \
  --description "Learning git fundamentals, with notes for authors using Claude Code."
```

What this does:
- Creates an empty repo at `github.com/blossomz37/my-novel-test`
- Public visibility
- Adds a short description (helpful for future-you and anyone who finds it)
- Does **not** auto-add a README/.gitignore/license (we already have our own)
- Does **not** push yet (so we can do step 3 explicitly for learning purposes)

### Step 3 — Add `origin` and verify

```bash
git remote add origin https://github.com/blossomz37/my-novel-test.git
git remote -v
```

`git remote -v` should show:

```
origin  https://github.com/blossomz37/my-novel-test.git (fetch)
origin  https://github.com/blossomz37/my-novel-test.git (push)
```

Two lines because remotes are bidirectional — one URL for fetching, one for pushing (almost always the same URL).

### Step 4 — Push `main` and set upstream tracking

```bash
git push -u origin main
```

Breakdown:
- `push` — upload commits
- `-u` (short for `--set-upstream`) — record `origin/main` as the upstream for local `main`. After this, plain `git push` and `git pull` will know what to do without arguments.
- `origin main` — push branch `main` to remote `origin`

### Step 5 — Verify on GitHub and locally

```bash
git branch -vv          # local branch should show "[origin/main]" tracking info
git log --oneline       # confirm history is intact
gh repo view --web      # opens the repo in your browser
```

## Branch hygiene notes (no action needed unless asked)

- The Claude worktree branch `claude/unruffled-cohen-755056` will **not** be pushed. It's a Claude-internal branch; remote isn't its place.
- Once you're comfortable with the local→remote loop, the worktree itself can be cleaned up later with `git worktree remove .claude/worktrees/unruffled-cohen-755056` from the main checkout.

## What you'll have learned by the end

- The conceptual difference between a local repo and a remote
- That `origin` is just a conventional name, not magic
- Why `-u` matters on the first push (upstream tracking)
- That GitHub's default branch is `main`, not `master`, and how to rename
- That `gh repo create` can do everything in one command, but breaking it apart makes the moving parts visible

## Verification

Plan is successful when:
- `git remote -v` shows `origin` pointing at the GitHub URL
- `git branch -vv` shows `main` tracking `origin/main`
- `gh repo view --web` opens to a populated repo with all 9 commits visible
- The README, GIT-QUICKSTART, LESSONS_LEARNED, etc. all render on the GitHub repo page

## Critical files

No files are modified by this plan — it's all git config and a remote operation. The plan file itself is relocated to the workspace root after approval.
