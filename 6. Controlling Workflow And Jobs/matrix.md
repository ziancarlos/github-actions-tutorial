# GitHub Actions — Matrix Strategy

## Testing Multiple OS and Language Versions

If we want to test our application on every OS and language version combination, we can use a matrix strategy.

For example, we want to test:

* Ubuntu with Node.js 18, 19, and 20
* Windows with Node.js 18, 19, and 20

We can use `strategy.matrix` to define the different values.

```yaml
name: Eleven Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    strategy:
      matrix:
        node-version: [18, 19, 20]
        os: [ubuntu-latest, windows-latest]

    runs-on: ${{ matrix.os }}

    steps:
      - name: Get Code
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm ci

      - name: TestCode
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run build
```

The matrix creates every possible combination of the values.

```text
Node versions:
18, 19, 20

OS:
Ubuntu, Windows
```

Therefore:

```text
Ubuntu + Node 18
Ubuntu + Node 19
Ubuntu + Node 20

Windows + Node 18
Windows + Node 19
Windows + Node 20
```

There are:

```text
3 Node versions × 2 operating systems = 6 jobs
```

So GitHub Actions will create 6 separate jobs to test whether the code can successfully build in each environment.

This is useful for checking whether our code is compatible with different environments.

---

# Matrix Keys

The names inside `matrix` are up to us.

For example:

```yaml
matrix:
  node-version: [18, 19, 20]
  os: [ubuntu-latest, windows-latest]
```

We could instead use:

```yaml
matrix:
  node: [18, 19, 20]
  operating-system: [ubuntu-latest, windows-latest]
```

Then we would reference them using:

```yaml
runs-on: ${{ matrix.operating-system }}
```

and:

```yaml
node-version: ${{ matrix.node }}
```

The matrix key names are therefore user-defined.

---

# Adding Specific Combinations with `include`

We can use `include` to add a specific combination that isn't part of the original matrix.

For example:

```yaml
matrix:
  node-version: [18, 19, 20]
  os: [ubuntu-latest, windows-latest]

  include:
    - node-version: 21
      os: ubuntu-latest
```

The original matrix creates:

```text
Ubuntu + Node 18
Ubuntu + Node 19
Ubuntu + Node 20

Windows + Node 18
Windows + Node 19
Windows + Node 20
```

Then `include` adds:

```text
Ubuntu + Node 21
```

So we now have 7 jobs.

It does NOT add:

```text
Windows + Node 21
```

because we specifically included only:

```yaml
- node-version: 21
  os: ubuntu-latest
```

---

# Removing Specific Combinations with `exclude`

We can use `exclude` to remove a specific combination.

For example:

```yaml
exclude:
  - node-version: 18
    os: windows-latest
```

The original matrix would create:

```text
Ubuntu + Node 18
Ubuntu + Node 19
Ubuntu + Node 20

Windows + Node 18
Windows + Node 19
Windows + Node 20
```

After excluding:

```text
Windows + Node 18
```

we get:

```text
Ubuntu + Node 18
Ubuntu + Node 19
Ubuntu + Node 20

Windows + Node 19
Windows + Node 20
```

Notice that:

```text
Ubuntu + Node 18
```

is still tested.

`exclude` removes only the specific combination that matches both values.

---

# Complete Example with `include` and `exclude`

```yaml
name: Eleven Workflow

on:
  push:
    branches:
      - main

jobs:
  build:
    strategy:
      matrix:
        node-version: [18, 19, 20]
        os: [ubuntu-latest, windows-latest]

        include:
          - node-version: 21
            os: ubuntu-latest

        exclude:
          - node-version: 18
            os: windows-latest

    runs-on: ${{ matrix.os }}

    steps:
      - name: Get Code
        uses: actions/checkout@v3

      - name: Install Node
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm ci

      - name: TestCode
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run build
```

The final combinations are:

```text
Ubuntu + Node 18   → Test
Ubuntu + Node 19   → Test
Ubuntu + Node 20   → Test
Ubuntu + Node 21   → Test  ← include

Windows + Node 18  → Skip   ← exclude
Windows + Node 19  → Test
Windows + Node 20  → Test
```

Total:

```text
7 original/include combinations
- 1 excluded combination
= 6 jobs
```

# Main Idea

```text
matrix
  ↓
Define multiple possible values
  ↓
GitHub creates combinations
  ↓
Each combination becomes a separate job
  ↓
include → add specific combinations
  ↓
exclude → remove specific combinations
```

This is conceptually similar to hyperparameter combinations in experimentation: you define a set of possible values, and the system runs the process for each combination.
