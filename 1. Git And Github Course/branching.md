# 17. Git Branching

Branches allow us to create separate lines of development.

For example:

```text
master
   │
   └── feature-restructure
```

You can work on `feature-restructure` without directly changing the `master` branch.

---

# 18. View Existing Branches

Use:

```bash
git branch
```

Example output:

```text
  feature-restructure
* master
```

The `*` indicates the **branch you are currently working on**.

In this example:

```text
feature-restructure
    ↓
exists, but not currently active

* master
    ↓
currently active branch
```

---

# 19. Delete a Branch

To delete a local branch:

```bash
git branch -D <branch_name>
```

Example:

```bash
git branch -D feature-restructure
```

### Important

You cannot normally delete the branch you are currently working on.

For example, if you are currently on:

```text
* feature-restructure
```

you should first switch to another branch:

```bash
git checkout master
```

Then:

```bash
git branch -D feature-restructure
```

---

# 20. Create a Feature Branch

A common workflow is to create a branch for a new feature:

```bash
git checkout -b feature-restructure
```

This does two things:

1. Creates the `feature-restructure` branch.
2. Switches you to that branch.

You can verify it with:

```bash
git branch
```

Example:

```text
* feature-restructure
  master
```

The `*` tells you that you are currently working on `feature-restructure`.

---

# 21. Commit Changes on a Feature Branch

After creating the branch, make your changes and commit them:

```bash
git add .
git commit -m "add restructure page"
```

Now your Git history might look like:

```text
A ─── B
      \
       C
```

Where:

```text
B = latest commit on master
C = new commit on feature-restructure
```

---

# 22. View the Branch History

You can use:

```bash
git log
```

For example:

```text
commit bdda025f83a392634c1ccb7cb935d56ee4735e86 (HEAD -> feature-restructure)
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 16:30:01 2026 +0700

    add restructure page

commit a221d59fb8a6a9c9abfb1357eca5e21192828b0f (master)
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:31:13 2026 +0700

    initial commit

commit 40dd19fae301a9cbdc92e329f4f0bda4e39583fc
Author: Zian Carlos Wong <ziancrlswong@gmail.com>
Date:   Sat Aug 15 14:30:41 2026 +0700

    initial commit
```

Notice:

```text
bdda025... (HEAD -> feature-restructure)
a221d59... (master)
```

This means:

```text
master
   ↓
a221d59

feature-restructure
   ↓
bdda025
```

The `feature-restructure` branch has an additional commit that `master` does not have.

Therefore:

> `master` is behind `feature-restructure` by one commit.

The branches can be visualized as:

```text
40dd19f ─── a221d59 ─── bdda025
                ↑            ↑
              master    feature-restructure
                             ↑
                            HEAD
```

---

# 23. Merge a Branch Into Another Branch

Suppose you have:

```text
master
   │
   └── feature-restructure
```

and you have finished working on `feature-restructure`.

You now want to merge the feature into `master`.

### Step 1 — Switch to the Branch That Will Receive the Changes

First switch to `master`:

```bash
git checkout master
```

You can verify:

```bash
git branch
```

You should see:

```text
  feature-restructure
* master
```

The `*` tells you that `master` is now the current branch.

---

### Step 2 — Merge the Feature Branch

Run:

```bash
git merge feature-restructure
```

This means:

> "Take the changes from `feature-restructure` and merge them into my current branch."

Because you are currently on `master`, the feature branch is merged **into `master`**.

---

# 24. Complete Feature Branch Workflow

A typical workflow looks like:

```text
1. Create feature branch
        ↓
git checkout -b feature-restructure
        ↓
2. Make changes
        ↓
3. Stage changes
        ↓
git add .
        ↓
4. Create commit
        ↓
git commit -m "add restructure page"
        ↓
5. Switch back to master
        ↓
git checkout master
        ↓
6. Merge feature branch
        ↓
git merge feature-restructure
```

Visually:

```text
Before merge:

A ─── B ─── C
      ↑     ↑
    master  feature-restructure
```

After merging:

```text
A ─── B ─── C
            ↑
          master
```

In this simple case, Git may perform a **fast-forward merge**, because `master` had no new commits after the feature branch was created.

---

# 25. Important Concept — The Branch You Are Currently On Matters

This command:

```bash
git merge feature-restructure
```

does **not** mean:

> "Merge `master` into `feature-restructure`."

It means:

> "Merge `feature-restructure` INTO THE BRANCH I AM CURRENTLY ON."

For example:

```text
git checkout master
git merge feature-restructure
```

means:

```text
feature-restructure
        ↓
        ↓ merge into
        ↓
      master
```

Whereas:

```text
git checkout feature-restructure
git merge master
```

means:

```text
master
   ↓
   ↓ merge into
   ↓
feature-restructure
```

So always remember:

```text
git merge <branch_name>

             ↓
        source branch
             ↓
     ┌───────────────┐
     │ current branch│
     └───────────────┘
             ↑
        receives changes
```

The **current branch receives the changes** from the branch specified in `git merge`.
