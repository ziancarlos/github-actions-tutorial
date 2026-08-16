# GitHub Actions: Actions, Jobs, Dependencies, and Context

## 1. What Does "Action" Mean?

There is some confusing terminology in GitHub Actions:

- In GitHub, the overall automation process is called a **Workflow**.
- An **Action** actually means a reusable custom application that performs a specific task, typically a complex or frequently repeated task.
- Example: Downloading/checking out the GitHub repository onto a runner machine.

### Actions vs `run`

An alternative to using an Action is executing a command with the `run` keyword inside a workflow YAML file.

- `run` → Usually used for simple commands or scripts.
- `uses` → Used to use an existing Action, especially when the task is more complex or reusable.

For example, we could theoretically download our repository using shell commands such as `git clone`, but this would require handling the cloning process ourselves.

Instead, we can use the existing `checkout` Action:

https://github.com/marketplace/actions/checkout

The `checkout` Action automatically gets our repository's code onto the GitHub Actions runner.

---

# 2. Using Actions With `uses`

When using an Action inside a step, instead of the `run` keyword, we use the `uses` keyword.

Example:

```yaml
name: Test Workflow

on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3
```

Here:

```yaml
uses: actions/checkout@v3
```

means:

- `actions` → The owner/organization of the Action.
- `checkout` → The Action name.
- `v3` → The Action version.

---

# 3. Configuring Actions With `with`

Some Actions can be configured using the `with` keyword.

For example, if we want to install Node.js, we can use the `setup-node` Action.

```yaml
name: Test Workflow

on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24
```

Here:

```yaml
with:
  node-version: 24
```

configures the Action to install/use Node.js version 24.

### General Idea

```text
Step
│
├── name
│
├── uses
│   └── Which Action to use
│
└── with
    └── How to configure the Action
```

Not every Action requires `with`.

If an Action does not need configuration, we can simply use:

```yaml
uses: actions/checkout@v3
```

---

# 4. Working With Child Directories

What if our project is inside a child directory instead of the repository root?

For example:

```text
Repository
│
└── 2. Actions - Blocks And Components
    │
    └── starting-project
        ├── package.json
        └── package-lock.json
```

We might initially try:

```yaml
- name: Change Directory
  run: cd 2. Actions - Blocks And Components/starting-project

- name: Install Dependency
  run: npm ci
```

However, the directory change does **not persist between separate `run` steps**.

Each step runs independently.

Therefore, we should specify `working-directory` on the steps that need to work inside that directory.

Example:

```yaml
name: Test Workflow

on: workflow_dispatch

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24

      - name: Install Dependency
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm ci

      - name: Run Test
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm test
```

The important idea:

```text
run: cd ...
    ↓
Only affects that step
    ↓
Step finishes
    ↓
Directory change is gone
```

Therefore:

```yaml
working-directory: path/to/project
```

is useful when a step needs to execute inside a specific directory.

---

# 5. What Happens When a Test Fails?

GitHub Actions automatically tracks whether each step and job succeeds or fails.

For example:

```yaml
- name: Run Test
  run: npm test
```

If `npm test` fails:

```text
npm test
   ↓
Test fails
   ↓
Step fails
   ↓
Job fails
   ↓
Workflow is marked as failed
```

GitHub also provides the logs from the failed step so developers can investigate the problem.

## How Do We Handle a Failed Deployment/Code Change?

If the failure was caused by a bad code change, one possible approach is to revert the commit that introduced the problem.

Example:

```bash
git revert <previous_commit_id>
git add <file_name>
git commit -m "Revert problematic change"
git push origin <branch_name>
```

Note:

`git revert` creates a new commit that reverses the changes from an earlier commit.

---

# 6. Multiple Jobs

A workflow can contain multiple jobs.

Simply add another job under the `jobs` section.

Example:

```yaml
name: Second Workflow

on: push

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24

      - name: Install Dependency
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm ci

      - name: Run Test
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm test

  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24

      - name: Install Dependency
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm ci

      - name: Build Project
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm run build

      - name: Deploy
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: echo "deploying.."
```

By default, jobs run **in parallel**.

```text
Workflow
   │
   ├── Test Job   ──────┐
   │                    │ Parallel
   └── Deploy Job ──────┘
```

---

# 7. Running Jobs Sequentially With `needs`

What if we want `deploy` to wait until `test` finishes?

Use the `needs` keyword.

```yaml
deploy:
  runs-on: ubuntu-latest
  needs: [test]
```

Complete example:

```yaml
name: Second Workflow

on: push

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24

      - name: Install Dependency
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm ci

      - name: Run Test
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: [test]

    steps:
      - name: Get Repository
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: 24

      - name: Install Dependency
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm ci

      - name: Build Project
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: npm run build

      - name: Deploy
        working-directory: 2. Actions - Blocks And Components/starting-project
        run: echo "deploying.."
```

Now the flow becomes:

```text
Workflow
   │
   ↓
 Test Job
   │
   │ must finish successfully
   ↓
Deploy Job
```

If the required job fails:

```text
Test Job
   │
   ❌ Failed
   │
   ↓
Deploy Job
   │
   └── Not executed
```

`needs` can contain one or multiple jobs.

Example:

```yaml
needs: [test, build]
```

This means the current job must wait for both `test` and `build`.

---

# 8. Multiple Workflow Triggers

The `on` keyword can contain multiple event triggers.

For example:

```yaml
on: [push, workflow_dispatch]
```

This means the workflow can be triggered by:

- `push`
- `workflow_dispatch`

Equivalent idea:

```text
                ┌── Push
                │
Workflow ───────┤
                │
                └── Manual Trigger
```

---

# 9. GitHub Actions Context

GitHub Actions provides information about the current workflow run through **contexts**.

This information can be useful when we need to access metadata such as:

- Which repository triggered the workflow.
- Which branch triggered the workflow.
- Which commit triggered the workflow.
- Which event triggered the workflow.
- Information about the workflow.
- Information about the current job.
- Environment-related information.

Official documentation:

https://docs.github.com/en/actions/reference/workflows-and-actions/contexts

---

# 10. Accessing Context With `${{ }}`

GitHub Actions uses the `${{ }}` syntax to evaluate expressions and access GitHub Actions context.

Without `${{ }}`, GitHub Actions treats the value as normal text.

Example:

```yaml
run: echo "${{ github.repository }}"
```

This accesses the repository name from the `github` context.

---

# 11. Using `toJSON()`

`toJSON()` can be used to convert context data into a JSON-formatted string.

For example:

```yaml
name: Output Information

on: workflow_dispatch

jobs:
  info:
    runs-on: ubuntu-latest

    steps:
      - name: Output Github Context
        run: echo "${{ toJSON(github) }}"
```

This outputs the available information inside the `github` context as JSON.

Example conceptually:

```text
{
  "repository": "ziancarlos/github-actions-tutorial",
  "ref": "...",
  "sha": "...",
  "event_name": "workflow_dispatch",
  ...
}
```

This is useful when learning or debugging GitHub Actions because we can inspect the metadata available during a workflow run.

---

# Key Concepts To Remember

```text
Workflow
│
├── Trigger
│   ├── push
│   ├── pull_request
│   ├── workflow_dispatch
│   └── ...
│
└── Jobs
    │
    ├── Job 1
    │   ├── runs-on
    │   └── Steps
    │       ├── uses
    │       └── run
    │
    └── Job 2
        ├── needs
        └── Steps
```

### Quick Summary

- **Workflow** → The entire automation process.
- **Action** → A reusable application/task used inside a workflow step.
- **`uses`** → Use an existing Action.
- **`with`** → Configure an Action.
- **`run`** → Execute a shell command/script.
- **`working-directory`** → Specify which directory a step should execute in.
- **Multiple jobs** → Run in parallel by default.
- **`needs`** → Make a job wait for another job.
- **`on`** → Define workflow triggers.
- **`${{ }}`** → Evaluate GitHub Actions expressions and access context.
- **`toJSON()`** → Convert context data into a JSON string.