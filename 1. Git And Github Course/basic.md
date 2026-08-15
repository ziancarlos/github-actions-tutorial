# Git Basics — Repository, Staging, Commits & History

## 1. Initialize a Directory as a Git Repository

To start using Git inside a directory:

```bash
git init
```

This creates a hidden `.git` directory that stores Git's repository data.

After this, the directory is now a **Git repository**.

---

# 2. Understanding Git's Three Main States

A simple way to understand Git is:

```text
Working Directory
       │
       │ git add
       ▼
Staging Area
       │
       │ git commit
       ▼
Repository
```

### Working Directory

The files you are currently editing.

### Staging Area

The files/changes you have selected to include in the next commit.

### Repository

The collection of commits (code snapshots) that Git has permanently recorded.

---

# 3. Create a Code Snapshot with a Commit

A **commit** is essentially a snapshot of the project at a particular point in time.

Creating a commit normally requires two steps:

```bash
git add <file_name>
git commit -m "commit message"
```

For example:

```bash
git add app.py
git commit -m "add user authentication"
```

The first command chooses what should go into the snapshot.

The second command actually creates the snapshot.

---

# 4. Choose What Goes Into the Next Snapshot

Use:

```bash
git add <file_name>
```

For example:

```bash
git add app.py
```

This moves the current version of `app.py` into the **staging area**.

You can also stage multiple files:

```bash
git add app.py database.py
```

Or stage everything:

```bash
git add .
```

### Important

`git add` **does not create a commit**.

It only tells Git:

> "Include these changes in my next commit."

---

# 5. Actually Create the Snapshot

After staging the changes:

```bash
git commit -m "commit message"
```

Example:

```bash
git commit -m "add user authentication"
```

This creates a new commit containing the changes currently in the staging area.

You can think of it as:

```text
git add
    ↓
Choose changes for snapshot
    ↓
git commit
    ↓
Create snapshot
```

---

# 6. Check the Status of the Repository

Use:

```bash
git status
```

This tells you what is currently happening in your repository.

For example, you may see:

### Untracked Files

```text
Untracked files:
    app.py
```

This means Git knows that the file exists, but it has **not been added to the staging area**.

You can stage it with:

```bash
git add app.py
```

---

### Changes to Be Committed

```text
Changes to be committed:
    modified: app.py
```

This means the changes are currently **staged** and will be included in the next commit.

You can commit them with:

```bash
git commit -m "update app"
```

---

### Changes Not Staged for Commit

```text
Changes not staged for commit:
    modified: app.py
```

This means the file has changes that are **not currently staged**.

For example:

```text
Commit A
   ↓
git add app.py
   ↓
Staged version
   ↓
Modify app.py again
   ↓
Working directory now has additional changes
```

The newly modified changes are not automatically staged.

You would need to run:

```bash
git add app.py
```

again.

---

# 7. View Commit History

Use:

```bash
git log
```

You may see something like:

```text
commit a221d59fb8a6a9c9abfb1357eca5e21192828b0f (HEAD -> master)
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:31:13 2026 +0700

    add authentication

commit 40dd19fae301a9cbdc92e329f4f0bda4e39583fc
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:30:41 2026 +0700

    initial commit
```

Each commit contains information such as:

* **Commit ID** — unique identifier for the commit
* **Author** — who created the commit
* **Date** — when the commit was created
* **Commit message** — description of the changes

---

# 8. Understanding `HEAD -> master`

Consider:

```text
commit a221d59... (HEAD -> master)
```

There are two important concepts here.

### `master`

`master` is the name of the current branch.

The branch points to the latest commit in that branch.

### `HEAD`

`HEAD` represents your **current position in the Git history**.

So:

```text
HEAD -> master
         │
         ▼
    Commit A
```

means:

> HEAD is currently on the `master` branch, and `master` currently points to Commit A.

A more complete example:

```text
HEAD
 │
 ▼
master
 │
 ▼
Commit A
 │
 ▼
Commit B
 │
 ▼
Commit C
```

---

# 9. Go Back to a Previous Commit

You can temporarily move to an older commit using:

```bash
git checkout <commit_id>
```

For example:

```bash
git checkout 40dd19fae301a9cbdc92e329f4f0bda4e39583fc
```

Your working directory will now contain the code as it existed at that commit.

However, this does **not delete the newer commits**.

Git is simply moving `HEAD` to that older commit.

For example:

```text
A ─── B ─── C
          ▲
         HEAD
```

After:

```bash
git checkout B
```

you get:

```text
A ─── B ─── C
      ▲
     HEAD
```

Commit `C` still exists.

---

## Important: Detached HEAD

If you checkout a commit directly:

```bash
git checkout <commit_id>
```

you will usually enter **detached HEAD state**.

This means:

```text
HEAD
 │
 ▼
Commit B

master
 │
 ▼
Commit C
```

You are no longer currently positioned on the `master` branch.

To return to the branch:

```bash
git checkout master
```

Or, if your branch is named `main`:

```bash
git checkout main
```

---

# 10. Does `git log` Delete the Newer Commits?

No.

If you checkout an older commit, the newer commits are **not deleted**.

For example:

```text
A ─── B ─── C
```

After:

```bash
git checkout B
```

the history still contains:

```text
A ─── B ─── C
```

You are simply viewing the project from the perspective of commit `B`.

When you return to `master`:

```bash
git checkout master
```

you will see commit `C` again.

### Important Correction

`git log` shows the commits reachable from your **current position**.

So if you are on an older commit, newer commits may not appear in `git log` from that position.

That does **not** mean Git deleted them.

---

# 11. Reverting a Commit

Sometimes you want to undo the changes introduced by a previous commit.

There are two common approaches:

### Option 1 — Manually Make the Opposite Changes

You can manually modify the files and create a new commit:

```bash
git add .
git commit -m "remove authentication"
```

This creates a new snapshot containing your manual changes.

---

# 12. `git revert` — Safely Undo a Commit

Use:

```bash
git revert <commit_id>
```

For example:

```bash
git revert a221d59fb8a6a9c9abfb1357eca5e21192828b0f
```

`git revert` does **not delete the original commit**.

Instead, Git creates a **new commit that reverses the changes introduced by the target commit**.

For example:

```text
A ─── B ─── C
```

If `C` introduced a feature that you want to remove:

```bash
git revert C
```

Git creates:

```text
A ─── B ─── C ─── D
                ▲
                │
          revert C
```

Commit `C` still exists.

Commit `D` simply reverses the changes introduced by `C`.

### Why is this commonly used?

Because it preserves the history.

This is especially useful when commits have already been pushed to a shared remote repository.

---

# 13. `git reset` — Move the Branch Back

`git reset` moves the current branch to another commit.

For example:

```text
A ─── B ─── C
          ↑
        HEAD
```

If you run:

```bash
git reset B
```

the branch moves back to `B`:

```text
A ─── B
      ↑
    HEAD
```

The important part is **what happens to the changes from commit `C`**.

That depends on the reset option.

---

## `git reset --soft`

Use:

```bash
git reset --soft <commit_id>
```

`--soft` moves the branch back to the specified commit, but **keeps the changes from the removed commits in the staging area**.

Example:

```text
Before:

A ─── B ─── C
          ↑
        HEAD
```

Run:

```bash
git reset --soft B
```

Now:

```text
A ─── B
      ↑
    HEAD

Staging Area:
Changes from C ✅
```

The commit `C` is no longer part of the current branch, but its changes are still staged.

### When is `--soft` useful?

When you think:

> "I made a commit, but I want to undo the commit and modify what I'm going to commit."

For example, you accidentally included `.env`:

```text
Commit C
├── app.py
├── requirements.txt
└── .env ❌
```

You can undo the commit while keeping the changes:

```bash
git reset --soft HEAD~1
```

Then fix what should be committed and create the commit again.

---

## `git reset --mixed`

If you run:

```bash
git reset <commit_id>
```

without specifying an option, Git uses `--mixed` by default.

Example:

```bash
git reset --mixed B
```

This:

* Moves the branch back to `B`
* Keeps the changes in the working directory
* Removes them from the staging area

```text
A ─── B
      ↑
    HEAD

Working Directory:
Changes from C ✅

Staging Area:
Changes from C ❌
```

So you would need to run `git add` again before committing.

---

## `git reset --hard`

Use:

```bash
git reset --hard <commit_id>
```

`--hard` moves the branch back **and discards the changes from the commits after that point from the working directory and staging area**.

Example:

```text
Before:

A ─── B ─── C
          ↑
        HEAD
```

Run:

```bash
git reset --hard B
```

Now:

```text
A ─── B
      ↑
    HEAD
```

The changes introduced by `C` are removed from:

```text
Working Directory ❌
Staging Area       ❌
```

### When is `--hard` useful?

When you think:

> "I don't want the changes after this commit anymore. Just put my project back exactly as it was."

⚠️ Be careful with `--hard` because it can permanently discard uncommitted changes.

---

# 14. `--soft` vs `--mixed` vs `--hard`

| Command         | Branch     | Staging Area        | Working Directory |
| --------------- | ---------- | ------------------- | ----------------- |
| `reset --soft`  | Moves back | Changes kept staged | Changes kept      |
| `reset --mixed` | Moves back | Changes unstaged    | Changes kept      |
| `reset --hard`  | Moves back | Changes discarded   | Changes discarded |

A simple way to remember:

```text
--soft
"Undo the commit, but keep everything staged."

--mixed
"Undo the commit, but unstage the changes."

--hard
"Undo the commit and throw the changes away."
```

---

# 15. `git revert` vs `git reset`

These commands can both be used to undo changes, but they work differently.

### `git revert`

```bash
git revert <commit_id>
```

Creates a **new commit** that reverses the changes from the target commit.

```text
A ─── B ─── C
          ↓
        revert C
          ↓
A ─── B ─── C ─── D
```

Commit `C` still exists.

Commit `D` simply undoes the changes introduced by `C`.

Use this when you want to **preserve the existing history**.

---

### `git reset`

```bash
git reset --soft <commit_id>
```

or:

```bash
git reset --hard <commit_id>
```

moves the branch pointer backwards.

```text
Before:

A ─── B ─── C
          ↑
        HEAD


After reset:

A ─── B
      ↑
    HEAD
```

The commits after `B` are no longer part of the current branch history.

---

# 16. Simple Rule for `revert` vs `reset`

Think:

```text
git revert
    ↓
"I want to undo a commit
but keep the history."

git reset --soft
    ↓
"I want to undo the commit
but keep its changes so I can fix/recommit them."

git reset --hard
    ↓
"I want to go back and
throw away those changes."
```

If the commits have already been pushed and other people are using the branch, `git revert` is generally safer because it does not rewrite the existing history.

---

# 17. Common Scenario — Accidentally Committed `.env`

Suppose you accidentally did:

```bash
git add .
git commit -m "initial commit"
```

and your commit contains:

```text
initial commit
├── app.py
├── requirements.txt
└── .env ❌
```

You realize:

> ".env should not be tracked by Git."

There are **two different situations**.

---

## Situation A — You Don't Care That `.env` Is In the Previous Commit

If you simply want Git to **stop tracking `.env` from now on**, use:

```bash
git rm --cached .env
```

Then add `.env` to `.gitignore`:

```bash
echo ".env" >> .gitignore
```

Then:

```bash
git add .gitignore
git commit -m "stop tracking env file"
```

Now:

```text
Your computer:

.env ✅


Git:

.env ❌
.gitignore ✅
```

The important thing is:

> `git rm --cached .env` does NOT remove `.env` from the previous commit.

It creates a new change that tells Git to stop tracking the file.

The history becomes:

```text
Commit A
├── app.py
├── requirements.txt
└── .env ❌ accidentally included

        ↓

Commit B
├── app.py
├── requirements.txt
└── .gitignore
    .env is no longer tracked
```

---

# 18. Situation B — You Want to Fix the Latest Commit

If the accidental `.env` commit is the **latest commit** and you want to recreate that commit correctly, use `--soft`.

First:

```bash
git reset --soft HEAD~1
```

This means:

> "Remove the latest commit, but keep all of its changes staged."

Now remove `.env` from the staging area:

```bash
git rm --cached .env
```

Add `.env` to `.gitignore`:

```bash
echo ".env" >> .gitignore
```

Then stage the correct files:

```bash
git add .
```

Finally, create the corrected commit:

```bash
git commit -m "initial commit"
```

The result is:

```text
Before:

Commit A
├── app.py
├── requirements.txt
└── .env ❌


After:

Commit A'
├── app.py
├── requirements.txt
└── .gitignore
```

The `.env` file still exists on your computer:

```text
.env ✅
```

but it is no longer included in the commit:

```text
Git
└── .env ❌
```

---

# 19. Why `git reset --soft` Is Useful Here

The reason we use:

```bash
git reset --soft HEAD~1
```

instead of:

```bash
git reset --hard HEAD~1
```

is that we **want to keep the code changes**.

We only made a mistake about **what should be included in the commit**.

```text
--soft

Undo commit
    ↓
Keep changes
    ↓
Fix staging
    ↓
Commit again
```

Whereas:

```text
--hard

Undo commit
    ↓
Discard changes
    ↓
Changes are gone
```

So for the accidental `.env` situation:

```text
"I committed the wrong files,
but I still want my code."

        ↓

git reset --soft HEAD~1
```

Then fix the staging area and commit again.

---

# 20. Important `.env` Warning

If `.env` contains secrets such as:

```env
API_KEY=...
DATABASE_PASSWORD=...
SECRET_KEY=...
```

and the commit has already been pushed to GitHub or another remote, **deleting the file from the latest commit is not enough**.

The secret may still exist in the Git history.

In that situation:

1. Rotate/revoke the exposed secret.
2. Remove the secret from the repository history if necessary.
3. Add `.env` to `.gitignore`.

For example:

```gitignore
.env
```

The important lesson is:

```text
.gitignore
    ↓
Prevents files from being added accidentally

git rm --cached .env
    ↓
Stops an already-tracked file from being tracked

git reset --soft
    ↓
Undo a commit while keeping its changes

git reset --hard
    ↓
Undo a commit and discard its changes

git revert
    ↓
Create a new commit that undoes another commit
```
