# Lessons Learned — Phase 4: Pulling Remote Changes & Keeping Worktrees in Sync

Notes from closing the remote→local loop: getting GitHub-side commits down to the local clone, and keeping multiple worktrees aligned so future work doesn't blow up.

## 1. `git fetch` and `git pull` do different things

These get conflated, but they're not the same operation:

- **`git fetch`** — download new commit info from the remote into your local repo's metadata. **Working files do not change.** Your local branches still point to where they pointed before.
- **`git pull`** — `git fetch` + `git merge` rolled together. Working files DO change.

If you're not sure what the remote has, fetch first and inspect. If you trust the situation, pull directly.

```bash
git fetch                      # safe: nothing in your working tree changes
git log HEAD..origin/main      # see what's about to come in
git pull                       # commit to the merge
```

## 2. `git status` lies until you fetch

Your local git only knows what the remote looked like at the last sync. If someone else (or you, on another machine, or via the GitHub web UI) pushes commits, your local `git status` will still cheerfully say "up to date with origin/main" — because *as far as it knows*, it is.

`git fetch` is what makes `git status` honest again. After fetching:

```
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
```

That message only appears post-fetch. Build the habit: fetch first, *then* check status.

## 3. "Behind", "ahead", and "diverged" are the three states

After fetching, `git status` will tell you one of:

- **"Up to date"** — local and remote match
- **"Ahead by N"** — you have N local commits not on remote (push will fast-forward remote)
- **"Behind by N"** — remote has N commits not local (pull will fast-forward local)
- **"Have diverged"** — both sides have commits the other doesn't (true merge or rebase needed)

The first three are simple. Divergence is where you actually have to think — and it's the case branches usually drift into when multiple people are pushing or you forget to pull before working.

## 4. Pulling into the *current* branch only

`git pull` updates only the branch you're on right now. Other branches in your repo — including Claude Code worktree branches — remain at whatever commit they pointed to before. Branches don't auto-update each other.

This is by design but it has a real consequence: after pulling `main`, any feature/worktree branch that started from `main` is now behind `main` too.

## 5. Sync your worktree branch at the START of a session

This is the most useful practical habit from this phase, especially with Claude Code's worktree model:

```bash
# At the top of a Claude Code session, in the worktree:
git fetch
git merge origin/main          # or: git rebase origin/main
```

Why: a Claude worktree branch is created from `main` at the moment the session starts. If `main` has moved on since (web UI commits, other sessions, other machines), your worktree branch is *stale* before the session even begins. Building new commits on a stale base means the end-of-session merge into `main` won't be a clean fast-forward — you'll get a real merge or a divergence to resolve.

The mirror image of the end-of-session pattern:

| Direction | When | Command |
| --- | --- | --- |
| `main` → worktree | Start of session, get up to date | `git fetch && git merge origin/main` (in worktree) |
| Worktree → `main` | End of session, ship the work | `git merge --ff-only claude/<name>` then `git push` (in main checkout) |

## 6. `git pull` defaults work because of the `-u` you set earlier

Plain `git pull` with no arguments works because back when we did the *first* push, we used `git push -u origin main`. The `-u` flag set `origin/main` as the upstream of local `main`. After that, both `git pull` and `git push` know what to fetch from / push to without arguments. This is one of the quiet payoffs of doing the first push correctly.

## 7. Fast-forwards happen in both directions

Throughout this project we've fast-forwarded:
- Worktree branch tip → `main` (end of Claude task)
- `origin/main` → local `main` (after a remote commit)
- `main` → worktree branch (start of new session)

Same operation, different directions. A fast-forward is just "advance the branch pointer to a descendant commit, no merge commit needed." It works any time the source branch is a strict descendant of the target branch.

## 8. `git pull --ff-only` is a useful safety belt

```bash
git pull --ff-only
```

Tells git: only do the pull if it can be done as a fast-forward. If the remote has diverged from your local (someone else pushed *and* you have unpushed local commits), the pull stops and tells you, instead of creating a merge commit you didn't ask for. Many people configure this as their default:

```bash
git config --global pull.ff only
```

After this, plain `git pull` becomes safer — you'll always know if real merging is needed, never have it sneak in.

## TL;DR

- `git fetch` = download info, no file changes. `git pull` = fetch + merge.
- `git status` is stale until you fetch — build the habit of fetching first
- Branches don't auto-update each other; pulling `main` does not update your worktree branch
- At the start of a Claude session, sync the worktree branch: `git fetch && git merge origin/main`
- This is the mirror of the end-of-session fast-forward — together they make a clean two-way loop
- Set `pull.ff only` globally to avoid surprise merge commits
