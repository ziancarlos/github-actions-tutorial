# Workflow, Jobs, and Steps

## 1. Structure

A **code repository** can have multiple workflows.

Each **workflow** can contain one or more jobs.

Each **job** can contain one or more steps.

```text
Code Repository A
│
├── Workflow 1
│   ├── Job 1
│   │   ├── Step 1
│   │   └── Step 2
│   │
│   └── Job 2
│       ├── Step 1
│       ├── Step 2
│       ├── Step 3
│       └── Step 4
│
├── Workflow 2
│   └── Job 1
│       ├── Step 1
│       └── Step 2
│
└── Workflow 3
    └── Job 1
        ├── Step 1
        ├── Step 2
        ├── Step 3
        └── Step 4
```

---

# 2. Key Elements

## Workflow

A **workflow** is an automated process defined in a repository.

1. Attached to a repository.
2. Contains **one or more jobs**.
3. Triggered by **events**.

   * Example: When a `push` happens on branch `main`, trigger the workflow.
4. Defined using a YAML file inside `.github/workflows/`.

Example:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  test:
    # ...

  build:
    # ...
```

---

## Jobs

A **job** is a unit of work inside a workflow.

1. Defines a **runner** where the job will execute.
2. Contains **one or more steps**.
3. Example:

   * When the workflow is triggered → run `test` job and `build` job.
4. Jobs run **in parallel by default**.
5. Jobs can be configured to run **sequentially** using dependencies such as `needs`.
6. Jobs can be **conditional**.

Example:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    # ...

  build:
    runs-on: ubuntu-latest
    # ...
```

By default:

```text
Workflow
   │
   ├── Test Job   ──────┐
   │                    │ Parallel
   └── Build Job  ──────┘
```

---

## Steps

A **step** is an individual action performed inside a job.

1. Can execute a **shell command/script**.
2. Can use **third-party or pre-built actions**.
3. Steps run **sequentially** by default.
4. Steps can be **conditional**.

Example:

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v4

  - name: Install dependencies
    run: npm install

  - name: Run tests
    run: npm test
```

The steps execute in order:

```text
Step 1
  ↓
Step 2
  ↓
Step 3
```

### Overall hierarchy

```text
Repository
    │
    └── Workflow
          │
          ├── Job
          │     ├── Step
          │     ├── Step
          │     └── Step
          │
          └── Job
                ├── Step
                └── Step
```
