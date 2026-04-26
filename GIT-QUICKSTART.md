# Git Quickstart for Authors

A friendly map of local git for writers who already know Word, Scrivener, or Google Docs. Git isn't really a different beast — it's a precise, named version of habits you already have.

## Translation table — author terms ↔ git terms

| What you already do | What git calls it |
| --- | --- |
| Save the document | **commit** — a labeled snapshot of your manuscript |
| The label or note next to a saved version | **commit message** |
| Version History pane (Google Docs / Word) | **`git log`** — list of every snapshot |
| Compare to earlier version | **`git diff`** — what changed |
| Track Changes (pending edits not yet accepted) | **staged changes** waiting in the **staging area** |
| Accept changes | **`git add`** then **`git commit`** |
| Reject changes / revert | **`git restore`** |
| "Manuscript_v2_FINAL_FINAL_actually.docx" | **branch** — a named parallel draft |
| Working from your "main" manuscript file | being on the **`main`** branch |
| Starting an experimental rewrite in a copy | **`git switch -c rewrite-ch3`** — make a new branch |
| Folding the rewrite back into the main draft | **`git merge`** |
| The cursor / "what version am I looking at?" | **HEAD** |
| Files Scrivener excludes from Compile | **`.gitignore`** |
| Shelving a half-done pass to deal with later | **`git stash`** |
| Renaming a save you just made | **`git commit --amend`** |
| Rolling back to last week's version | **`git reset`** or **`git restore`** |

## The mental model

Forget commands for a minute. There are **three places** your words live:

```
your draft  →   pages set aside    →   bound manuscript history
(editing)       (ready to save)         (every version, kept forever)
```

1. **Working tree** — the chapter you're editing right now. Unsaved.
2. **Staging area** — pages you've decided are ready. Not committed yet, but earmarked for the next save. Think of it as the stack on your desk: "these are going in."
3. **Repository** — your bound history. Every commit is a permanent snapshot you can return to.

`git add` moves pages from the desk into the "ready" stack. `git commit` binds that stack into the manuscript and adds a label. That's the entire loop.

A **commit** is one labeled snapshot. **HEAD** points to the snapshot you're currently looking at. A **branch** is just a name attached to a snapshot — like a sticky tab on a particular version.

## The daily writing loop

```bash
git status                  # what have I changed since the last save?
git diff                    # show me my edits
git add chapter-3.md        # set this chapter aside, ready to save
git diff --staged           # what's about to be saved?
git commit -m "Tighten chapter 3 opening; cut Mrs. Ashford's monologue"
git log --oneline           # show me my version history
```

Run `git status` constantly. It always tells you where things are and usually suggests the next move.

## Undoing things (ranked from safe to scary)

| You want to... | Command | Risk |
| --- | --- | --- |
| Take a chapter off the "ready" stack (keep edits) | `git restore --staged chapter-3.md` | Safe |
| Throw away today's edits to a chapter | `git restore chapter-3.md` | **You lose those edits** |
| Reword the label on your last save | `git commit --amend` | Safe (before sharing) |
| Add a forgotten file into the last save | `git add notes.md && git commit --amend --no-edit` | Safe (before sharing) |
| Undo the last save, keep the pages on the "ready" stack | `git reset --soft HEAD~1` | Safe |
| Undo the last save, put the pages back on your desk | `git reset HEAD~1` | Safe |
| Erase the last save AND throw away its edits | `git reset --hard HEAD~1` | **Destructive** |

Rule of thumb: if a command has `--hard` or `-f` in it, pause and read it twice.

## Branches — your "what if" drafts

Suppose you want to try a darker tone in chapter 7 without disturbing your real manuscript. In Word you'd "Save As Manuscript_dark_v1.docx" and pray you remember which is canonical. Git does this cleanly:

```bash
git switch -c darker-ch7    # start a parallel draft, copied from where you are
# ...edit, save, commit on this draft...
git switch main             # jump back to the canonical manuscript
git switch darker-ch7       # back to the experiment
git merge darker-ch7        # (while on main) fold the experiment in
git branch -d darker-ch7    # delete the branch once it's merged
```

Both drafts share the entire history up to the point you branched. You can flip between them instantly — your files actually change on disk to match the branch you're on.

See the structure of all your drafts at once:

```bash
git log --oneline --graph --all
```

This one command makes branching click. Run it after every merge.

## Stashing — "I need to set this pass aside"

You're three paragraphs into a revision when an editor email forces you to dig into chapter 1. You don't want to save the half-done revision yet — but you can't lose it either.

```bash
git stash                   # shelve current edits, working tree goes clean
# ...do the urgent thing, commit it...
git stash pop               # bring the half-done edits back
git stash list              # see everything you've shelved
```

## .gitignore — what NOT to track

Some files don't belong in version history: OS junk, scratch notes, exported PDFs, Scrivener's caches. Tell git to ignore them with a `.gitignore` file:

```
*.pdf           # ignore all PDFs (you re-export them anyway)
.DS_Store       # macOS clutter
exports/        # ignore the whole exports folder
scratch.md      # ignore one specific file
!keep-this.pdf  # but DO track this one (negation)
```

If a file is already tracked, adding it to `.gitignore` won't untrack it — you'd run `git rm --cached <file>` first.

## Inspection — reading your manuscript history

```bash
git log --oneline -10                # last 10 saves, terse
git log --oneline --graph --all      # all drafts, visualized
git show <sha>                       # exactly what changed in one save
git diff main..darker-ch7            # what differs between two drafts
git blame chapter-3.md               # which save last touched each line
```

(`<sha>` is the short string that identifies a commit — you'll see them in `git log`, e.g. `e0d07ca`.)

## When you're ready to go beyond your laptop

So far this is all local. To collaborate, back up to GitHub, or sync between machines, you'll add:

- `git clone <url>` — copy someone else's repo (or your own from GitHub) onto your laptop
- `git remote -v` — list the remote copies your repo knows about
- `git fetch` / `git pull` — download new commits from the remote
- `git push` — upload your commits to the remote
- **Merge conflicts** — what happens when two drafts edited the same paragraph; git pauses and asks you to choose. Resolve by hand, then `git add` + `git commit`.
- `git rebase` — rewrite history to keep it tidy. Powerful, easy to misuse. Learn it once `merge` feels natural.

## Three habits worth building early

1. **Save (commit) often, in small logical units.** One commit, one idea: "Cut chapter 3 opening" is a commit. "Whole-novel rewrite" is not.
2. **Write the label for future-you.** "Edits" tells you nothing in six months. "Fix Millicent's eye color in chapter 4 (was hazel, now grey)" does.
3. **`git status` before every action.** It costs nothing and prevents most "wait, what did I just do" moments.

That's the whole local loop. The rest is practice — and after about a week of using it on a real project, the commands stop feeling foreign and start feeling like Save buttons with better memory.
