# Git Quickstart

A friendly map of local git for beginners. Focuses on the daily loop and the mental model — not every flag.

## The mental model

Git has **three areas**. Every command moves files between them.

```
working tree  →  staging area  →  repository
   (edits)        (git add)       (git commit)
```

- **Working tree** — your files on disk. Where you edit.
- **Staging area** (also called the *index*) — a holding pen for changes you intend to commit next.
- **Repository** — the permanent history of commits.

A **commit** is a snapshot of the staging area, plus a message and an author. **HEAD** is a pointer to "where you are" in history. A **branch** is just a named pointer to a commit.

Once those four ideas click — three areas, commit, HEAD, branch — git stops feeling random.

## The daily loop

```bash
git status                  # what's changed?
git diff                    # show unstaged edits
git add <file>              # stage a file
git diff --staged           # show what's about to be committed
git commit -m "message"     # record the snapshot
git log --oneline           # see history
```

Tip: run `git status` constantly. It tells you which area things are in and usually suggests the next command.

## Undoing things (in order of safety)

| Goal | Command | Notes |
| --- | --- | --- |
| Unstage a file (keep edits) | `git restore --staged <file>` | Safe |
| Discard unstaged edits | `git restore <file>` | Loses edits — be sure |
| Fix the last commit's message | `git commit --amend` | Only before pushing |
| Add a forgotten file to last commit | `git add <file> && git commit --amend --no-edit` | Only before pushing |
| Undo last commit, keep changes staged | `git reset --soft HEAD~1` | Safe |
| Undo last commit, keep changes in working tree | `git reset HEAD~1` | Safe |
| Nuke last commit and its changes | `git reset --hard HEAD~1` | **Destructive** |

Rule of thumb: if a command has `--hard` or `-f` in it, pause and read twice.

## Branching

Branches let you work on something without disturbing the main line.

```bash
git switch -c feature-x     # create and switch to a new branch
git switch main             # go back to main
git switch feature-x        # back to your feature
git merge feature-x         # while on main, pull feature-x's commits in
git branch                  # list local branches
git branch -d feature-x     # delete a merged branch
```

Visualize the topology:

```bash
git log --oneline --graph --all
```

This one command makes branching click. Run it after every merge.

## Stashing

Need to switch contexts mid-edit without committing?

```bash
git stash                   # shelve current changes
git stash pop               # bring them back
git stash list              # see all shelved sets
```

## .gitignore basics

Patterns, one per line. Glob-style:

```
*.log           # ignore all .log files
build/          # ignore the build directory
secret.env      # ignore one specific file
!keep.log       # but DO track this one (negation)
```

Files already tracked are not affected by adding them later — you'd need to `git rm --cached <file>` first.

## Useful inspection commands

```bash
git log --oneline -10                # last 10 commits, terse
git log --oneline --graph --all      # branch topology
git show <sha>                       # what changed in a specific commit
git diff <branch1>..<branch2>        # what differs between two branches
git blame <file>                     # who last touched each line
```

## What's next (beyond local)

When you're ready to go past local-only:

- `git clone <url>` — copy a remote repo
- `git remote -v` — list remotes
- `git fetch` — download remote changes without merging
- `git pull` — fetch + merge
- `git push` — upload your commits
- **Merge conflicts** — what happens when two branches edit the same lines; you resolve them by hand, then `git add` and `git commit`
- `git rebase` — rewrite history to keep it linear; powerful, easy to misuse, learn it after merge feels comfortable

## Three habits worth building early

1. **Commit often, in small logical units.** A commit should do one thing. Easier to review, easier to revert.
2. **Write the message for future-you.** "Fix bug" tells you nothing in six months. "Fix off-by-one in chapter word-count tally" does.
3. **`git status` before every action.** It costs nothing and prevents most "wait, what just happened" moments.
