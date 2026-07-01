# Lecture 5 — Version Control & Git

Official notes: https://missing.csail.mit.edu/2026/version-control/

**Big idea:** Git takes **snapshots** of your project over time, keeps the full
**history**, and lets people collaborate. A repo's history is a **chain of
snapshots**, each pointing back to its parent.

> Sections marked **(reference)** we didn't drill hands-on yet — they're here so
> the notes are complete.

## Mental model

- A **commit** = a snapshot (a save point) of your *whole project* at one moment.
- Each commit has a unique **ID** (a fingerprint computed from its contents — so
  it's unique *and* tamper-evident).
- **Pointers (refs)** you'll see in `git log`:
  - `main` / `master` = **your local** branch (the tip of a line of commits).
  - `HEAD` = **"you are here"** — the commit/branch you're currently on.
  - `origin/main` = **GitHub's copy** of main (`origin` = nickname for the remote).
  - When you commit, the branch pointer **jumps forward** to the new snapshot.
  - When `main` is ahead of `origin/main`, you have **unpushed commits**.

## The three areas + core workflow

```
Working directory  →  Staging area  →  Repository
   (you edit)          (git add)        (git commit = snapshot)
```

- **Working directory** = your real files. **Staging area** = a box for the
  changes you want in the *next* snapshot. **Repository** = sealed history.
- Why the separate `git add`? So you choose *exactly* what goes in each commit.

```bash
git status                 # what changed? what's staged?
git diff                   # show unstaged changes (line by line; + = added)
git diff --staged          # show what's IN the staging box
git add FILE               # stage a file (put it in the box)
git commit -m "message"    # seal the box into a snapshot
git log --oneline          # compact history, one line per commit
```

`git status` saying **"nothing to commit, working tree clean"** = everything's
committed; your files match the latest snapshot.

## Branches — parallel timelines

A **branch** is a movable named pointer to a commit — a separate line of work.
Use it to experiment/build a feature without disturbing your stable line.

```bash
git branch                 # list branches (* = current)
git switch -c NAME         # create a new branch AND switch to it
                           #   (older form: git checkout -b NAME)
git switch NAME            # switch to an existing branch
git branch -d NAME         # delete a merged branch (cleanup)
```

A commit on one branch only affects that branch. Switching branches changes what
your files look like — because each branch is its own timeline.

## Merging

Bring one branch's work into another. Switch to the branch you want to merge
*into*, then merge the other in:

```bash
git switch main
git merge experiment
```

- **Fast-forward:** if `main` had no new commits, Git just slides its pointer
  forward. No new commit created.
- **Merge commit:** if both branches diverged (both made commits), Git combines
  them into a new *merge commit*.

## Merge conflicts

Happen when **both branches change the same line** differently. (Changes on
*different* lines merge automatically — that's how you avoid conflicts.)

Git pauses (prompt shows `MERGING`) and puts markers in the file:
```
<<<<<<< HEAD          your current branch's version
Color: green
=======               divider
Color: red
>>>>>>> change-to-red the incoming branch's version
```

**Resolve in 4 steps:**
1. Edit the file so it says what you want — **delete all three marker lines**.
2. `git add FILE`  — tells Git "resolved."
3. `git commit -m "..."`  — finalizes the merge.
4. `git status` should be clean again; `MERGING` disappears.

## Undo — fixing mistakes (reference)

| Command | Undoes |
|---------|--------|
| `git restore FILE` | discard **unstaged** changes to a file (revert it to last commit) |
| `git restore --staged FILE` | unstage a file (take it out of the box, keep the edits) |
| `git commit --amend` | fix the **last** commit (message or add forgotten changes) |
| `git reset --soft HEAD~1` | undo last commit, **keep** changes staged |
| `git reset --hard HEAD~1` | undo last commit and **throw away** changes (dangerous!) |
| `git revert COMMIT` | make a **new** commit that undoes an old one (safe for shared history) |
| `git stash` / `git stash pop` | temporarily shelve changes, then bring them back |
| `git reflog` | a log of everywhere HEAD has been — your safety net to recover "lost" commits |

Rule of thumb: `revert` is safe for history others have (it adds a commit);
`reset --hard` rewrites/destroys and should be used carefully on shared branches.

## Remotes — the GitHub side (reference)

You've been doing these all night:

```bash
git clone URL              # copy a remote repo to your machine
git push                   # upload your commits to the remote (GitHub)
git pull                   # download + merge others' commits from the remote
git fetch                  # download remote changes WITHOUT merging yet
git remote -v              # show your remotes (origin = your GitHub repo)
```

`origin` = the nickname for your remote. `origin/main` = your local record of
where the remote's main branch was last time you synced.

## .gitignore (reference)

A file listing patterns Git should **ignore** (never track) — build outputs,
secrets, OS junk:
```
node_modules/
*.log
.DS_Store
.env
```

## Handy

```bash
git log --oneline --graph --decorate --all   # visualize history as a graph
git show COMMIT                               # see what a commit changed
git blame FILE                                # who last changed each line
```
