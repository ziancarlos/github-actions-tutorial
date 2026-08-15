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

# 13. `git reset --hard` — Move the Branch Back

Another way to go back is:

```bash
git reset --hard <commit_id>
```

For example:

```bash
git reset --hard 40dd19fae301a9cbdc92e329f4f0bda4e39583fc
```

Suppose you have:

```text
A ─── B ─── C ─── D
```

Then:

```bash
git reset --hard B
```

moves the branch back:

```text
A ─── B
      ▲
    master
```

The branch no longer points to `C` or `D`.

Those commits may become unreachable and eventually be garbage-collected by Git.

### Important

`git reset --hard` can also discard **uncommitted changes in your working directory**.

Therefore, be careful when using it.

It is generally not something you want to casually use on commits that have already been shared with other people.

---

# 14. `git revert` vs `git reset --hard`

| Command            | What it does                                    |        Original commit remains? | Creates new commit? |
| ------------------ | ----------------------------------------------- | ------------------------------: | ------------------: |
| `git revert`       | Creates a new commit that undoes another commit |                             Yes |                 Yes |
| `git reset --hard` | Moves branch pointer backwards                  | No longer reachable from branch |                  No |

### Simple rule

Use:

```bash
git revert
```

when you want to **undo a change while preserving history**.

Use:

```bash
git reset --hard
```

when you intentionally want to **move the branch back and discard the commits after that point**.

---

# 15. The Basic Git Workflow

The most important workflow to remember is:

```text
                git add
Working Directory ────────> Staging Area
                                  │
                                  │ git commit
                                  ▼
                             Repository
                                  │
                                  │ git log
                                  ▼
                            Commit History
```

In practice:

```bash
# 1. Create repository
git init

# 2. Check status
git status

# 3. Stage changes
git add .

# 4. Create snapshot
git commit -m "initial commit"

# 5. View snapshot history
git log
```

Then, whenever you make more changes:

```bash
git add .
git commit -m "describe the changes"
```

---

# 16. The Core Concept

Think of Git like taking **multiple snapshots of your project**:

```text
Snapshot 1
    │
    ▼
Snapshot 2
    │
    ▼
Snapshot 3
    │
    ▼
Snapshot 4
```

Each commit is a snapshot of the project's state at that point in time.

The important distinction is:

```text
git add
    ↓
"Which changes should be included?"

git commit
    ↓
"Create the snapshot."

git log
    ↓
"Show me the snapshots."

git checkout
    ↓
"Let me move to/view a different point in history."

git revert
    ↓
"Create a new snapshot that undoes an old snapshot's changes."

git reset --hard
    ↓
"Move the branch back to an older snapshot."
```
