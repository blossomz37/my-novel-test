# Lessons Learned — Phase 2: Going from Local to GitHub Remote

Notes from taking a purely local repo public on GitHub for the first time. Focus is on the *one-way trip* from "lives only on your laptop" to "lives on GitHub too" — not on day-to-day push/pull collaboration.

## 1. A remote is just a named URL

The word "remote" sounds technical, but a git remote is nothing more than **a name pointing at a URL** of another copy of your repo. The conventional name for "the main remote" is `origin`. After it's set up, your local repo knows where to send `git push` and where to fetch from.

```bash
git remote add origin https://github.com/<you>/<repo>.git
git remote -v   # confirm
```

`origin` isn't a magic word — it's just a convention. You can name it whatever you want.

## 2. GitHub's default branch is `main`, not `master`

If you started a repo with `git init` years ago, your default branch is probably `master`. Modern GitHub (since 2020) defaults to `main`. Renaming locally before you push avoids divergence:

```bash
git branch -m master main
```

This is purely local — no consequence as long as you haven't pushed yet. After the first push to a new remote, the remote will use `main` automatically.

## 3. `gh repo create` skips the GitHub web UI entirely

Instead of clicking through GitHub's "New Repository" form, the GitHub CLI can do it from your terminal:

```bash
gh repo create my-novel-test \
  --public \
  --description "Short description for the repo card."
```

By default this creates an empty repo (no auto-generated README/license/.gitignore), which is exactly what you want when pushing an existing local history — no merge conflict on the first push.

Other useful flags:
- `--private` for a private repo
- `--source=. --remote=origin --push` to do *everything* (create, add remote, push) in one command — useful once you're comfortable with the moving parts

## 4. The first push needs `-u`

```bash
git push -u origin main
```

The `-u` flag (short for `--set-upstream`) records `origin/main` as the upstream for your local `main`. After that, plain `git push` and `git pull` work without arguments because git knows what they mean. Skipping `-u` on the first push works, but you'll have to type `origin main` every subsequent time.

## 5. Going public means *every line of every file* is now visible

Sounds obvious, but the consequences hit harder than expected:

- **Filepaths** in docs that include `/Users/<your-username>/...` reveal your macOS short name
- **First names or personal references** in plan files or notes
- **Anything in `.env`, config files, or scratch notes** you forgot was tracked

Before going public, do a full-repo grep:

```bash
git grep -nE '/Users/|<your-name>|@gmail|@(api[_-]?key|secret|password)'
```

Replace user-specific paths with placeholders like `~/projects/<repo>/` or `<your-repo-path>/`.

## 6. Commit metadata is part of "going public" too

This is the surprise. Even with squeaky-clean *file content*, every commit carries the **author name and email** baked in:

```bash
git log --format='%an <%ae>'
```

If your `git config user.email` is your real email address, every commit you've ever made publicly attaches that email to your work — searchable, scrapable, and forever.

## 7. GitHub provides a `noreply` email — use it

GitHub gives every account a private alias of the form:

```
<numeric-id>+<username>@users.noreply.github.com
```

Find your numeric ID:

```bash
gh api user --jq '.id'
```

Configure git to use it for all future commits:

```bash
git config --global user.email "<id>+<username>@users.noreply.github.com"
```

Future commits will carry only this address. Your real email stays out of public history.

## 8. Fixing personal email *already* in commit history requires a rewrite

Existing commits can't have their metadata changed without a history rewrite. Two tools:

- **`git filter-repo`** — modern, fast, recommended (install via Homebrew: `brew install git-filter-repo`)
- **`git filter-branch`** — bundled with git, slower, deprecated but works fine for small repos

For email replacement across all commits with `filter-branch`:

```bash
FILTER_BRANCH_SQUELCH_WARNING=1 git filter-branch --env-filter '
if [ "$GIT_AUTHOR_EMAIL" = "old@example.com" ]; then
  export GIT_AUTHOR_NAME="new-name"
  export GIT_AUTHOR_EMAIL="new@example.com"
fi
if [ "$GIT_COMMITTER_EMAIL" = "old@example.com" ]; then
  export GIT_COMMITTER_NAME="new-name"
  export GIT_COMMITTER_EMAIL="new@example.com"
fi
' --tag-name-filter cat -- --all
```

Important detail: rewriting touches **both** "author" (who wrote it) and "committer" (who applied it) — they're separate fields and both can leak emails.

## 9. After a rewrite, force-push and clean up

Two follow-up steps the rewrite doesn't do automatically:

```bash
# Remove the safety backup refs filter-branch leaves behind
git for-each-ref --format='%(refname)' refs/original/ | xargs -n1 git update-ref -d

# Push the rewritten history (this overwrites the old history on GitHub)
git push --force origin main
```

`--force` is destructive: it overwrites the remote with your local history. Use it deliberately and only on branches *you control*. Never force-push to `main` of a repo other people have cloned, unless you've coordinated with them first.

## 10. Force-push has a safer cousin: `--force-with-lease`

```bash
git push --force-with-lease origin main
```

`--force-with-lease` checks that the remote tip matches what your local thinks it should be. If someone else pushed in the meantime, the lease fails and git refuses, protecting you from clobbering their work. Use this for normal force-push situations.

In a *fresh* rewrite scenario like ours, the local `origin/main` ref was itself rewritten by `filter-branch`, so the lease is meaningless and plain `--force` is appropriate. This is one of the few times plain `--force` is the right call.

## 11. The fast-forward workflow grows a new step

Pre-remote, "fast-forward" meant only: merge the worktree branch into `main` locally. Once you have a remote, the workflow grows a final step:

```bash
git push
```

Plain `git push` (no arguments) pushes only the current branch to its tracked upstream. It will *not* push other branches like Claude's worktree branch — which is exactly what you want.

## 12. GitHub keeps orphaned commit objects briefly after a rewrite

Even after force-pushing rewritten history, GitHub retains the old commit objects in internal storage for a period (typically up to 90 days for unreferenced objects). The old SHAs are no longer reachable from any branch, don't appear in the API or web UI commit list, and aren't searchable — but if you've **already shared a specific old SHA URL** elsewhere, it might resolve briefly.

Practical takeaway: scan and rewrite *before* anyone has had time to copy a commit URL. For a freshly-pushed repo, this window is essentially closed by the time the rewrite finishes.

## 13. The `gh` CLI ecosystem rewards getting set up once

After a single `gh auth login`, `gh` handles authentication for every subsequent operation: creating repos, viewing them, querying the API, opening PRs, listing issues. No browser needed for most workflows.

Useful one-liners:

```bash
gh repo view --web                    # open this repo in the browser
gh repo view <owner>/<repo>           # show metadata in the terminal
gh api user --jq '.id'                # your numeric user ID
gh api repos/<owner>/<repo>/commits   # raw API access
```

## TL;DR

- A remote is just a named URL; `origin` is the convention
- Rename `master` → `main` before pushing
- `gh repo create` + `git remote add origin ...` + `git push -u origin main` is the whole first-push flow
- Going public exposes file content **and** commit author metadata
- Switch `git config user.email` to GitHub's `<id>+<user>@users.noreply.github.com` to keep your real email out of future commits
- Use `git filter-branch` (or `git filter-repo`) + `git push --force` to rewrite emails already in history
- Once remote, the fast-forward workflow adds a `git push` at the end
