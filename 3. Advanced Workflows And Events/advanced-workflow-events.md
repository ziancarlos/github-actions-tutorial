# Event Activity Types & Filters

## 1. Events vs. Event Activity Types

A GitHub Actions **event** determines **what kind of GitHub activity** can trigger a workflow.

Common events include:

* `push`
* `pull_request`

However, an event can have more specific **activity types**.

For example:

```yaml
pull_request:
  types:
    - opened
```

Here:

* `pull_request` → **Event**
* `opened` → **Event Activity Type**

A `pull_request` event can have multiple activity types:

```yaml
pull_request:
  types:
    - opened
    - closed
    - edited
```

So, the workflow will trigger when the specified pull request activities occur.

---

# 2. Pull Request Event

A basic pull request trigger:

```yaml
on:
  pull_request:
```

This means the workflow can be triggered by pull request activities.

You can restrict it to specific activity types:

```yaml
on:
  pull_request:
    types:
      - opened
```

The workflow will only trigger when a pull request is **opened**.

You can specify multiple activity types:

```yaml
on:
  pull_request:
    types:
      - opened
      - closed
      - edited
```

---

# 3. Pull Request Branch Filters

For `pull_request`, the `branches` filter refers to the **target/base branch** of the pull request.

Example:

```yaml
on:
  pull_request:
    types:
      - opened
    branches:
      - main
```

This means:

> Trigger the workflow when a pull request is opened **and its target branch is `main`**.

For example:

```text
dev ────────────────┐
                    └── PR → main
```

This **will trigger** the workflow.

But:

```text
dev ────────────────┐
                    └── PR → devb
```

This **will not trigger** the workflow because the target branch is `devb`, not `main`.

### Important

For `pull_request`, remember that:

```yaml
branches:
  - main
```

refers to the **branch being merged into**, not the source branch.

---

# 4. Checkout Code in Pull Request Workflows

The code checked out by the workflow is based on the pull request's merge context by default.

For example:

```text
dev ────────────────┐
                    └── PR → main
```

The workflow is associated with the pull request from `dev` into `main`.

Therefore, do not assume that the default checkout simply means:

```text
checkout → main
```

The checkout action determines the specific ref/commit that is checked out.

If you need to explicitly control what branch/ref is checked out, specify it using the `with` keyword:

```yaml
- name: Checkout
  uses: actions/checkout@v4
  with:
    ref: dev
```

Here:

```yaml
with:
  ref: dev
```

explicitly tells `actions/checkout` which ref to check out.

---

# 5. Push Event

The `push` event is triggered when commits are pushed to the repository.

Basic example:

```yaml
on:
  push:
```

You can restrict the workflow to specific branches:

```yaml
on:
  push:
    branches:
      - main
      - 'dev-*'
      - 'feat/**'
```

These are **branch filters**.

### `main`

```yaml
- main
```

Triggers when pushing to:

```text
main
```

Example:

```bash
git push origin main
```

---

### `'dev-*'`

```yaml
- 'dev-*'
```

Matches a `dev-` prefix followed by characters, without crossing `/`.

Examples:

```text
dev-login
dev-payment
dev-feature
```

But not:

```text
dev/backend
```

---

### `'feat/**'`

```yaml
- 'feat/**'
```

Matches branches beginning with `feat/`, including additional `/` characters.

Examples:

```text
feat/login
feat/payment
feat/user/profile
feat/backend/auth
```

---

# 6. Push Branch vs. Pull Request Branch

The meaning of `branches` depends on the event.

### Push

For `push`:

```yaml
on:
  push:
    branches:
      - main
```

The branch refers to the **branch being pushed to**.

```text
git push origin main
              ↓
         triggers workflow
```

The checkout will normally use the commit associated with that push.

### Pull Request

For `pull_request`:

```yaml
on:
  pull_request:
    branches:
      - main
```

The branch refers to the **target/base branch**.

```text
dev ────────────────┐
                    └── PR → main
                          ↑
                     target branch
```

---

# 7. Path Filters

For events such as `push`, you can also filter based on which files changed.

There are two important filters:

* `paths`
* `paths-ignore`

---

## `paths-ignore`

Example:

```yaml
on:
  push:
    branches:
      - main
      - 'dev-*'
      - 'feat/**'
    paths-ignore:
      - '.github/workflows/**'
```

`paths-ignore` means:

> Do not trigger the workflow when the changes only match the ignored paths.

For example, if the only changed file is:

```text
.github/workflows/test.yml
```

the workflow will be skipped.

But if the commit changes:

```text
src/app.js
```

the workflow can trigger.

### Important

It is better to think of `paths-ignore` as:

> **Ignore the workflow when the changed files are only within these paths.**

It does **not** mean that the workflow is ignored whenever an ignored file appears alongside other non-ignored changes.

---

# 8. `paths`

`paths` does the opposite: it specifies which paths can trigger the workflow.

Example:

```yaml
on:
  push:
    paths:
      - 'src/**'
```

This means:

> Trigger the workflow when relevant changes occur under `src/`.

For example:

```text
src/app.js
src/components/Button.jsx
```

can trigger the workflow.

But changes only to:

```text
README.md
docs/example.md
```

will not trigger this workflow.

---

# 9. `paths` vs. `paths-ignore`

### `paths`

```yaml
paths:
  - 'src/**'
```

Meaning:

> Only changes matching these paths trigger the workflow.

### `paths-ignore`

```yaml
paths-ignore:
  - 'docs/**'
```

Meaning:

> Changes only in these paths should not trigger the workflow.

A simple way to remember:

```text
paths
    ↓
"What changes should trigger?"

paths-ignore
    ↓
"What changes should be ignored?"
```

---

# 10. Cancelling a Workflow

You can manually cancel a running workflow from the GitHub repository.

Go to:

```text
GitHub Repository
      ↓
   Actions
      ↓
Select the workflow
      ↓
Select the running workflow
      ↓
Cancel workflow
```

This stops the currently running workflow.

---

# 11. Skipping a Workflow

You can also skip a workflow using a special commit-message suffix.

Example:

```bash
git add <file_name>

git commit -m "Update documentation [skip ci]"

git push origin <branch_name>
```

The important part is:

```text
[skip ci]
```

When supported by the workflow/event, GitHub will skip the workflow triggered by that commit.

Other commonly supported skip markers include:

```text
[skip ci]
[ci skip]
```

So:

```bash
git commit -m "Update README [skip ci]"
```

can be used when you do not want the commit to trigger CI.

---

# 12. Quick Summary

```text
Event
│
├── push
│   │
│   ├── branches
│   │   └── Which branch is being pushed to?
│   │
│   ├── paths
│   │   └── Which changed paths should trigger?
│   │
│   └── paths-ignore
│       └── Which paths should be ignored?
│
└── pull_request
    │
    ├── types
    │   └── Which PR activity?
    │       ├── opened
    │       ├── closed
    │       └── edited
    │
    └── branches
        └── Which target/base branch?
```

## Key Things to Remember

1. **Event** = what GitHub activity happened.
2. **Activity type** = more specific action within an event.
3. `pull_request.branches` = **target/base branch**.
4. `push.branches` = **branch receiving the push**.
5. `paths` = only matching changed paths can trigger.
6. `paths-ignore` = ignore workflows when changes are only in ignored paths.
7. For PR workflows, understand what ref `actions/checkout` is checking out; use `with.ref` when you explicitly need a particular branch/ref.
8. A workflow can be manually cancelled from **Actions → running workflow → Cancel workflow**.
9. `[skip ci]` can be added to a commit message to skip supported CI workflows.
