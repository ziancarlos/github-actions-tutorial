# Environment Variables & Secrets in GitHub Actions

## 1. Environment Variables

Creating environment variables in GitHub Actions is important because workflows often need configuration values such as:

* Port numbers
* Database names
* API URLs
* Environment-specific configuration
* Other application settings

GitHub Actions allows us to define environment variables at different scopes.

Common scopes are:

1. Workflow level
2. Job level
3. Step level

---

# 2. Workflow-Level Environment Variables

An environment variable can be defined at the workflow level:

```yaml
name: Eight Workflow

on:
  push:
    branches:
      - main
      - dev

env:
  MONGO_DB_NAME: gha_demo
```

Because `MONGO_DB_NAME` is defined at the workflow level, it is available to jobs throughout the workflow.

Conceptually:

```text
Workflow
│
├── MONGO_DB_NAME
│
├── test
│
└── deploy
```

The workflow-level variable can therefore be accessed by both `test` and `deploy`.

---

# 3. Job-Level Environment Variables

An environment variable can also be defined at the job level:

```yaml
jobs:
  test:
    env:
      PORT: 300
```

This means `PORT` belongs to the `test` job.

Conceptually:

```text
Workflow
│
├── MONGO_DB_NAME
│
├── test
│   └── PORT
│
└── deploy
```

The `test` job can access both:

```text
MONGO_DB_NAME
PORT
```

But `deploy` can only access:

```text
MONGO_DB_NAME
```

It cannot access the `PORT` environment variable defined specifically inside `test`.

---

# 4. Important Scope Rule

**Job-level environment variables only exist within that job.**

For example:

```yaml
jobs:
  test:
    env:
      PORT: 300
```

`PORT` is available inside:

```text
test
├── step 1
├── step 2
└── step 3
```

but not inside:

```text
deploy
```

unless `PORT` is separately defined there or passed through another mechanism.

---

# 5. Accessing Environment Variables

Environment variables can be accessed using the `env` context:

```yaml
${{ env.VARIABLE_NAME }}
```

For example:

```yaml
- name: Echo
  run: echo "${{ env.MONGO_DB_NAME }}"
```

If:

```yaml
env:
  MONGO_DB_NAME: gha_demo
```

the expression:

```yaml
${{ env.MONGO_DB_NAME }}
```

resolves to:

```text
gha_demo
```

---

# 6. Example: Workflow-Level and Job-Level Variables

```yaml
name: Eight Workflow

on:
  push:
    branches:
      - main
      - dev

env:
  MONGO_DB_NAME: gha_demo

jobs:
  test:
    env:
      PORT: 300

    runs-on: ubuntu-latest

    steps:
      - name: Get Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}

      - name: Install Dependency
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm ci

      - name: Test Code
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm run test
```

Here:

```text
Workflow-level:
MONGO_DB_NAME

Job-level:
PORT
```

---

# 7. Secrets

Environment variables are useful for configuration, but we should **not hard-code sensitive information** inside the workflow.

For example, do not do this:

```yaml
env:
  MONGODB_PASSWORD: my-super-secret-password
```

The workflow file is part of the repository and may be visible to people who have access to the repository.

Instead, sensitive information should be stored as a **GitHub Secret**.

Examples:

```text
MONGODB_USERNAME
MONGODB_PASSWORD
API_KEY
DATABASE_PASSWORD
```

---

# 8. Creating Repository Secrets

To create a repository-level secret:

```text
GitHub Repository
      ↓
Settings
      ↓
Secrets and variables
      ↓
Actions
      ↓
New repository secret
```

Then provide:

```text
Name:
MONGODB_PASSWORD

Secret:
your-password
```

The secret is stored by GitHub rather than directly inside the workflow YAML.

---

# 9. Accessing Secrets

Secrets can be accessed through the `secrets` context:

```yaml
${{ secrets.SECRET_NAME }}
```

For example:

```yaml
env:
  MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
```

This means:

```text
GitHub Secret
     ↓
secrets.MONGODB_PASSWORD
     ↓
environment variable
     ↓
MONGODB_PASSWORD
```

---

# 10. Example Using Secrets

```yaml
name: Eight Workflow

on:
  push:
    branches:
      - main
      - dev

env:
  MONGO_DB_NAME: ${{ secrets.MONGO_DB_NAME }}

jobs:
  test:
    env:
      MONGODB_CLUSTER_ADDRESS: ${{ secrets.MONGODB_CLUSTER_ADDRESS }}
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: ${{ secrets.PORT }}

    runs-on: ubuntu-latest

    steps:
      - name: Get Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}

      - name: Install Dependency
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm ci

      - name: Run Server
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm start & npx wait-on http://127.0.0.1:$PORT

      - name: Test
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm run test

      - name: Echo
        run: echo "${{ env.MONGODB_USERNAME }}"
```

---

# 11. GitHub Secret Masking

When GitHub Actions prints a secret value in the logs, GitHub attempts to mask the secret.

For example, instead of:

```text
my-super-secret-password
```

the log may show:

```text
***
```

This helps prevent secrets from being accidentally exposed through workflow logs.

However, **do not assume that secrets are impossible to leak**.

You should still:

* Never intentionally print secrets.
* Never hard-code secrets into workflow files.
* Avoid passing secrets into commands where they can be exposed.
* Be careful when using third-party actions.
* Treat secrets as sensitive information.

---

# 12. Environments

GitHub Actions also supports **Environments**.

Environments are useful when you have different deployment environments such as:

```text
test
staging
production
```

Each environment can have its own:

* Secrets
* Variables
* Protection rules
* Deployment controls

For example:

```text
test
├── MONGODB_USERNAME
├── MONGODB_PASSWORD
└── MONGO_DB_NAME

staging
├── MONGODB_USERNAME
├── MONGODB_PASSWORD
└── MONGO_DB_NAME

production
├── MONGODB_USERNAME
├── MONGODB_PASSWORD
└── MONGO_DB_NAME
```

The names can be the same while the actual values are different.

---

# 13. Creating an Environment

To create an environment:

```text
GitHub Repository
      ↓
Settings
      ↓
Environments
      ↓
New environment
```

For example, create:

```text
test
```

Then you can configure environment-specific variables and secrets.

---

# 14. Using an Environment in a Job

A job can specify which environment it belongs to:

```yaml
jobs:
  test:
    environment: test
```

Example:

```yaml
jobs:
  test:
    environment: test

    env:
      MONGODB_CLUSTER_ADDRESS: ${{ secrets.MONGODB_CLUSTER_ADDRESS }}
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: ${{ secrets.PORT }}

    runs-on: ubuntu-latest
```

Now the job uses the configuration associated with the `test` environment.

---

# 15. Example Environment Structure

Imagine we have three environments:

```text
GitHub Repository
│
├── test
│   ├── MONGODB_USERNAME
│   ├── MONGODB_PASSWORD
│   └── PORT
│
├── staging
│   ├── MONGODB_USERNAME
│   ├── MONGODB_PASSWORD
│   └── PORT
│
└── production
    ├── MONGODB_USERNAME
    ├── MONGODB_PASSWORD
    └── PORT
```

Then:

```yaml
jobs:
  test:
    environment: test
```

uses the `test` environment.

And:

```yaml
jobs:
  deploy:
    environment: production
```

could use the `production` environment.

This allows the same workflow to work with different environments without putting different credentials directly into the YAML file.

---

# 16. Important Difference: Environment Variables vs Secrets

### Environment Variable

Used for normal configuration:

```yaml
env:
  PORT: 3000
  MONGO_DB_NAME: gha_demo
```

### Secret

Used for sensitive values:

```yaml
env:
  MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
```

Think:

```text
Normal configuration
        ↓
Environment Variable

Sensitive configuration
        ↓
GitHub Secret
```

---

# 17. Important Difference: Repository Secrets vs Environment Secrets

### Repository Secret

Available to workflows/jobs that have access to the repository secret:

```text
Repository
└── Secrets
    ├── API_KEY
    └── MONGODB_PASSWORD
```

### Environment Secret

Associated with a specific environment:

```text
test
└── Secrets
    └── MONGODB_PASSWORD

production
└── Secrets
    └── MONGODB_PASSWORD
```

The same secret name can have different values depending on the environment.

---

# 18. Complete Mental Model

```text
GitHub Actions
│
├── Workflow-level env
│   └── Available throughout the workflow
│
├── Job-level env
│   └── Available only inside that job
│
├── Repository Secrets
│   └── Sensitive values stored securely
│
└── Environments
    ├── test
    │   └── Environment-specific secrets/variables
    │
    ├── staging
    │   └── Environment-specific secrets/variables
    │
    └── production
        └── Environment-specific secrets/variables
```

The key concepts are:

> **`env` = configuration values.**

> **`secrets` = sensitive values that should not be hard-coded.**

> **Job-level `env` only applies to that job.**

> **Environment = a named deployment context such as `test`, `staging`, or `production`, which can have its own secrets, variables, and protection rules.**
