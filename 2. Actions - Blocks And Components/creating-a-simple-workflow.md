# Creating A Simple Workflow

In your **GitHub Repository**:

**Actions → Configure a workflow yourself**

> **Note:** The workflow file name does not represent anything. The actual workflow name is defined using the `name` property.

---

## 1. Create The Workflow Name

Use the `name` keyword to define the workflow's name.

This name will be shown in the **GitHub Actions UI**.

```yaml
name: First Workflow
```

---

## 2. Determine The Trigger

Use the `on` keyword to determine **when the workflow should be triggered**.

### Manual Trigger

If you want to manually trigger the workflow from the **GitHub Actions UI**:

```yaml
on: workflow_dispatch
```

### Push Trigger

If you want the workflow to run when someone pushes code to a specific branch:

```yaml
on:
  push:
    branches:
      - main
```

For example:

```yaml
name: First Workflow

on: workflow_dispatch
```

This means the workflow can be triggered **manually from the GitHub Actions UI**.

---

## 3. Create Jobs

Remember:

> **One Workflow can have multiple Jobs.**

The `jobs` property contains the Jobs that belong to the Workflow.

For example:

```yaml
name: First Workflow

on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubuntu-latest
```

### What is `runs-on`?

`runs-on` determines **where the Job will run** — specifically, which runner/environment will execute the Job.

```yaml
runs-on: ubuntu-latest
```

means:

> Run this Job on a GitHub-provided machine running Ubuntu.

Other examples:

```yaml
runs-on: windows-latest
```

```yaml
runs-on: macos-latest
```

So the structure is:

```text
Workflow
│
└── Jobs
    │
    ├── first-job
    │
    ├── second-job
    │
    └── third-job
```

---

## 4. Create Steps

Remember:

> **One Job can have multiple Steps.**

Each Step generally has two important properties:

* `name` → The title/description of the Step
* `run` → The command that should be executed

Example:

```yaml
steps:
  - name: Print Greeting
    run: echo "Hello World"

  - name: End
    run: echo "Bye Bye"
```

Here:

```yaml
- name: Print Greeting
  run: echo "Hello World"
```

means:

> Create a Step called **Print Greeting** and execute `echo "Hello World"`.

And:

```yaml
- name: End
  run: echo "Bye Bye"
```

means:

> Create a Step called **End** and execute `echo "Bye Bye"`.

---

# Complete Workflow

Putting everything together:

```yaml
name: First Workflow

on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Print Greeting
        run: echo "Hello World"

      - name: End
        run: echo "Bye Bye"
```

### Overall Structure

```text
Workflow
│
├── name
│
├── on
│   └── Trigger
│
└── jobs
    │
    └── first-job
        │
        ├── runs-on
        │
        └── steps
            ├── Step 1
            │   ├── name
            │   └── run
            │
            └── Step 2
                ├── name
                └── run
```

**The key hierarchy to remember:**

> **Workflow → Jobs → Steps**

## Manual Workflow Trigger

When using the `workflow_dispatch` trigger, the workflow can be **manually triggered from the GitHub repository**.

Example:

```yaml
on:
  workflow_dispatch:
```

To manually trigger it:

1. Go to the **Actions** tab in the GitHub repository.
2. On the left sidebar, find and select the **workflow name**.
3. Click **Run workflow**.
4. Select the branch if required.
5. Click **Run workflow** again.

> **Note:** You manually trigger the **workflow**, not individual jobs. Once the workflow is triggered, GitHub Actions runs the jobs defined inside that workflow according to their configuration.

```text
GitHub Repository
      │
      └── Actions
            │
            └── Workflow Name
                  │
                  └── Run workflow
                        │
                        ▼
                    Workflow
                        │
                  ┌─────┴─────┐
                  ▼           ▼
                Job 1       Job 2
                  │           │
                Steps       Steps
```
