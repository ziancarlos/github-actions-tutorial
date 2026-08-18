# Controlling Workflow and Jobs

## 1. Default Behavior When a Step Fails

Consider this workflow:

```yaml
test:
  runs-on: ubuntu-latest
  steps:
    - name: Get Code
      uses: actions/checkout@v3

    - name: Cache Dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: 6. Controlling Workflow And Jobs/starting-project/node_modules
        key: node-modules-${{ hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json') }}

    - name: Install Dependencies
      working-directory: 6. Controlling Workflow And Jobs/starting-project
      run: npm ci

    - name: TestCode
      id: trigger-test
      working-directory: 6. Controlling Workflow And Jobs/starting-project
      run: npm run test

    - name: Upload Test Report
      uses: actions/upload-artifact@v4
      with:
        name: test-report
        path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

By default, GitHub Actions stops executing subsequent steps when a previous step fails.

Therefore:

```text
TestCode succeeds
    ↓
Upload Test Report runs
```

But:

```text
TestCode fails
    ↓
Upload Test Report is skipped
```

This is not ideal for testing because we may want to retrieve the test report even when the tests fail.

---

# 2. Using `if`

GitHub Actions provides the `if` keyword to control whether a step or job should run.

The `if` condition can evaluate boolean expressions and supports operators such as:

```text
&&   → AND
||   → OR
!    → NOT
```

We can also use contexts inside an `if` expression.

For example, GitHub provides the `steps` context:

https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#steps-context

The `steps` context allows us to access information about previous steps by using their `id`.

For example:

```yaml
- name: TestCode
  id: trigger-test
  run: npm run test
```

Because the step has:

```yaml
id: trigger-test
```

we can access its outcome with:

```yaml
steps.trigger-test.outcome
```

---

# 3. Checking the Outcome of a Previous Step

We could try:

```yaml
- name: Upload Test Report
  if: steps.trigger-test.outcome == 'failure'
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

The idea is:

```text
TestCode
   ↓
Check trigger-test outcome
   ↓
failure?
   ↓
Upload report
```

However, this alone is not enough.

Why?

Because GitHub Actions has a default behavior:

> If a previous step fails, subsequent steps are skipped unless their conditions allow them to run despite the failure.

So even though:

```yaml
steps.trigger-test.outcome == 'failure'
```

is true, GitHub Actions can still skip the step because the job is already in a failed state.

---

# 4. `failure()`

To explicitly allow a step to run when a previous step has failed, we can use the `failure()` status check function.

```yaml
if: failure() && steps.trigger-test.outcome == 'failure'
```

`failure()` returns:

```text
true
```

when a previous step or job has failed.

Therefore:

```yaml
- name: Upload Test Report
  if: failure() && steps.trigger-test.outcome == 'failure'
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

This means:

```text
Previous step/job failed
        AND
trigger-test outcome is failure
        ↓
Upload Test Report
```

However, there is another approach that is often simpler for this particular testing use case.

---

# 5. `continue-on-error`

`continue-on-error: true` tells GitHub Actions:

> Even if this step fails, continue executing the following steps.

Example:

```yaml
- name: TestCode
  continue-on-error: true
  id: trigger-test
  working-directory: 6. Controlling Workflow And Jobs/starting-project
  run: npm run test

- name: Upload Test Report
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

Now the behavior becomes:

```text
TestCode succeeds
    ↓
continue
    ↓
Upload Test Report
```

or:

```text
TestCode fails
    ↓
continue-on-error: true
    ↓
continue
    ↓
Upload Test Report
```

This is useful because we want the test report to be uploaded regardless of whether the test itself succeeds or fails.

---

# 6. Difference Between `failure()` and `continue-on-error`

These two mechanisms solve different problems.

## `failure()`

Used when we want to execute something specifically because something failed.

Example:

```yaml
if: failure()
```

Meaning:

```text
Something failed
    ↓
Run this step
```

It is useful for things such as:

```text
Test fails
    ↓
Generate failure report
```

or:

```text
Build fails
    ↓
Send notification
```

---

## `continue-on-error: true`

Used when we want a failed step to **not stop the remaining workflow**.

Example:

```yaml
continue-on-error: true
```

Meaning:

```text
Step fails
    ↓
Don't stop the workflow
    ↓
Continue to next step
```

For our test-report use case, this is appropriate because we want:

```text
Test succeeds
    ↓
Upload report


Test fails
    ↓
Continue anyway
    ↓
Upload report
```

---

# 7. Status Check Functions

GitHub Actions provides several status check functions:

```text
failure()
success()
always()
cancelled()
```

## `failure()`

Returns `true` when a previous step or job has failed.

```yaml
if: failure()
```

Example:

```text
Test
  ↓
FAIL
  ↓
failure() = true
  ↓
Run report step
```

---

## `success()`

Returns `true` when the relevant previous steps/jobs have succeeded.

```yaml
if: success()
```

Example:

```text
Test
  ↓
SUCCESS
  ↓
success() = true
  ↓
Run next step
```

---

## `always()`

Always returns `true`, regardless of whether previous steps succeeded, failed, or were cancelled.

```yaml
if: always()
```

This can be useful when something should always be executed.

For example:

```yaml
- name: Upload Test Report
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-report
    path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

This says:

```text
SUCCESS → Upload
FAILURE → Upload
CANCELLED → Upload
```

Be careful with `always()` because you may not want a step to run after every possible failure or cancellation.

---

## `cancelled()`

Returns `true` when the workflow/job has been cancelled.

```yaml
if: cancelled()
```

---

# 8. Controlling Jobs

The `if` keyword doesn't only work on steps.

It can also be used on jobs.

For example:

```yaml
report:
  if: failure()
  needs: [lint, deploy]
  runs-on: ubuntu-latest

  steps:
    - name: Output Information
      run: |
        echo "Something went wrong"
        echo "${{ toJSON(github) }}"
```

Here:

```yaml
needs: [lint, deploy]
```

means the `report` job depends on the `lint` and `deploy` jobs.

If one of those jobs fails:

```text
lint ──────── SUCCESS ──┐
                        ├──→ report
deploy ────── FAILURE ──┘
```

Then:

```yaml
if: failure()
```

allows the `report` job to execute.

This is useful for creating a centralized reporting/notification job.

---

# 9. Dependency Cache Example

Another common use case is controlling whether dependencies need to be installed.

Consider:

```yaml
test:
  runs-on: ubuntu-latest

  steps:
    - name: Get Code
      uses: actions/checkout@v3

    - name: Cache Dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: 6. Controlling Workflow And Jobs/starting-project/node_modules
        key: node-modules-${{ hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json') }}

    - name: Install Dependencies
      working-directory: 6. Controlling Workflow And Jobs/starting-project
      if: steps.cache.outputs.cache-hit != 'true'
      run: npm ci

    - name: TestCode
      continue-on-error: true
      id: trigger-test
      working-directory: 6. Controlling Workflow And Jobs/starting-project
      run: npm run test

    - name: Upload Test Report
      uses: actions/upload-artifact@v4
      with:
        name: test-report
        path: 6. Controlling Workflow And Jobs/starting-project/test.json
```

Here, the cache step has:

```yaml
id: cache
```

The cache action provides an output:

```yaml
steps.cache.outputs.cache-hit
```

This tells us whether an exact cache was found.

Therefore:

```yaml
if: steps.cache.outputs.cache-hit != 'true'
```

means:

```text
Cache hit?
    ↓
YES → Don't run npm ci
NO  → Run npm ci
```

---

# 10. Why the Cache Key Uses `package-lock.json`

The cache key is:

```yaml
key: node-modules-${{ hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json') }}
```

The `hashFiles()` function creates a hash based on the contents of `package-lock.json`.

For example:

```text
package-lock.json
       ↓
hashFiles()
       ↓
ABC123
       ↓
node-modules-ABC123
```

If the dependencies don't change:

```text
package-lock.json
       ↓
same hash
       ↓
same cache key
       ↓
cache hit
       ↓
skip npm ci
```

If the dependencies change:

```text
package-lock.json changes
       ↓
different hash
       ↓
different cache key
       ↓
cache miss
       ↓
run npm ci
```

This makes the cache dependent on the dependency definition.

---

# 11. Important Difference: `~/.npm` vs `node_modules`

When caching dependencies, understand the difference between:

```text
~/.npm
```

and:

```text
node_modules/
```

`~/.npm` is npm's package download cache.

```text
~/.npm
    ↓
cached packages
    ↓
helps npm install faster
```

`node_modules` contains the actual installed dependencies:

```text
node_modules/
    ├── eslint/
    ├── react/
    ├── ...
    └── .bin/
        └── eslint
```

`npm ci` installs dependencies and creates `node_modules`.

Therefore, if the intention is:

```text
Cache hit
    ↓
Skip npm ci
```

then the cache needs to contain `node_modules`.

That's why this is appropriate for that strategy:

```yaml
path: 6. Controlling Workflow And Jobs/starting-project/node_modules
```

rather than:

```yaml
path: ~/.npm
```

---

# 12. Overall Concepts

The important concepts are:

```text
if
│
├── Controls whether a step/job runs
│
├── Can use boolean expressions
│
├── Can use && / || / !
│
└── Can access contexts such as steps


steps context
│
└── Allows us to inspect previous steps
    └── steps.<step-id>.outcome


failure()
│
└── Returns true when something has failed


success()
│
└── Returns true when previous execution succeeds


always()
│
└── Always returns true


cancelled()
│
└── Returns true when execution is cancelled


continue-on-error
│
└── Allows the workflow to continue after a step fails
```

The key distinction is:

```text
if: failure()
    → "Run this because something failed."


continue-on-error: true
    → "Even if this fails, don't stop the workflow."
```

For a test-reporting workflow, `continue-on-error: true` is useful when the test failure itself should not prevent the report-upload step from running.
