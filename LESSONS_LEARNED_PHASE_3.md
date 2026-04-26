# Lessons Learned — Phase 3: Creating Files via GitHub's Web UI

Notes from creating a file directly on GitHub through the browser — the *other* direction of the local-remote loop. Useful for quick edits, fixing typos, or working from a machine that doesn't have your repo cloned.

## 1. The web UI does have a file editor — and it's full git

The "Add file" → "Create new file" flow on GitHub isn't a lightweight workaround. The commit it produces is a real git commit, with author, message, parent SHA, and a place in the history graph identical to one made from your terminal. Anything you can do locally — view, blame, revert — works the same way on a web-created commit.

## 2. There's no "create folder" button — slashes do it

This catches everyone the first time. GitHub's web UI doesn't have a separate "New Folder" action. Instead, when you type a `/` in the filename field, GitHub interprets the prefix as a folder path:

```
say-anything/blah-blah.md
```

The moment you type the slash, the field visibly splits: `say-anything` becomes a navigable folder segment and `blah-blah.md` is the actual filename. Chain more slashes for nested folders (`a/b/c/file.md`). The directory is created as part of the same commit as the file.

You can't create an empty directory via the web UI for the same reason git can't track empty directories — git tracks files, not folders. The folder exists because a file inside it does.

## 3. Web commits go to a branch — pick the right one

The "Commit changes" panel offers two destinations:

- **Commit directly to `main`** — fast, fine for trivial edits and tests, no review
- **Create a new branch and start a pull request** — for changes that should be reviewed before landing

For learning or solo work, direct-to-main is fine. For collaborative or production code, the PR path is the more common pattern.

## 4. The commit message field works exactly like local commits

Same conventions apply. Short imperative summary on the first line, optional extended description in the body. GitHub even pre-fills a default message (`Create blah-blah.md`) but you can — and usually should — replace it with something more specific.

## 5. After a web commit, your local clone is "behind"

The moment you commit on GitHub, your local repo no longer has the latest history. Locally, `git status` will *still* show "up to date" until you `git fetch` — git only knows what it knew at the last sync. The new commit becomes visible after:

```bash
git fetch       # learn what's new
git status      # now shows "behind 1"
git pull        # bring the file down
```

This isn't broken behavior — it's a feature. Git is decentralized; it doesn't poll the remote. You ask for news when you want it.

## 6. When to use the web UI vs your terminal

| Situation | Best tool |
| --- | --- |
| Fix a typo in a README | Web UI — faster than cloning |
| Quick prose edit on a doc | Web UI |
| Edit while away from your dev machine | Web UI |
| Multi-file change | Local clone |
| Anything that needs testing before commit | Local clone |
| Renames, moves, deletes across many files | Local clone |
| Creating a brand-new file in an existing repo | Either, but local is usually faster once cloned |

## TL;DR

- "Add file" → "Create new file" creates real git commits via the browser
- Type `/` in the filename to create folders inline
- Choose "commit directly to main" for solo/learning work; use the PR path for collaboration
- After a web commit, your local clone is behind until you `git fetch` and `git pull`
- Web UI is great for quick edits; local clone is better for anything multi-file or testable
