# Understanding Job Outputs

## 1. What Is an Output?

A GitHub Actions **output** is a way to pass a small piece of information, usually a string, from one step/job to another.

It is **not intended for large files**.

Examples of information that can be passed as an output:

* File name
* Version number
* Status
* ID
* Reference name
* Other small pieces of information

For example:

```text
Build Job
    │
    └── output: "hello-world.js"
                ↓
            Deploy Job
                │
                └── uses "hello-world.js"
```

---

# 2. Creating an Output

To create a job output, there are two parts:

1. Create an output inside a step using `$GITHUB_OUTPUT`
2. Expose that step output as a **job output**

Example:

```yaml
build:
  needs: [test]
  runs-on: ubuntu-latest

  outputs:
    output-name: ${{ steps.publish.outputs.reference-name }}
    script-file: ${{ steps.publish.outputs.script-file }}

  steps:
    - name: Get Code
      uses: actions/checkout@v4

    - name: Install Dependency
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm ci

    - name: Build Code
      working-directory: 4. Job Artifacts And Output/starting-project
      run: npm run build

    - name: Publish Javascript File Name
      id: publish
      working-directory: 4. Job Artifacts And Output/starting-project
      run: |
        echo "script-file=hello-world.js" >> "$GITHUB_OUTPUT"
        echo "reference-name=value" >> "$GITHUB_OUTPUT"

    - name: Upload Artifacts
      uses: actions/upload-artifact@v4
      with:
        name: dist-files
        path: |
          4. Job Artifacts And Output/starting-project/dist
```

---

# 3. `$GITHUB_OUTPUT`

The important part is:

```bash
echo "script-file=hello-world.js" >> "$GITHUB_OUTPUT"
```

This creates a **step output** named:

```text
script-file
```

with the value:

```text
hello-world.js
```

Another example:

```bash
echo "reference-name=value" >> "$GITHUB_OUTPUT"
```

creates:

```text
reference-name = value
```

### Important

The output must be written to:

```text
$GITHUB_OUTPUT
```

For example:

```bash
echo "name=value" >> "$GITHUB_OUTPUT"
```

This tells GitHub Actions:

> Create an output named `name` with the value `value`.

---

# 4. Why Do We Need `id`?

Notice this step:

```yaml
- name: Publish Javascript File Name
  id: publish
```

The `id` allows us to reference the outputs created by this step.

For example:

```yaml
${{ steps.publish.outputs.script-file }}
```

Breaking it down:

```text
steps
  ↓
publish
  ↓
outputs
  ↓
script-file
```

So:

```yaml
${{ steps.publish.outputs.script-file }}
```

means:

> Get the `script-file` output from the step whose ID is `publish`.

---

# 5. Creating Job Outputs

A step output is not automatically available as a job output.

We expose it at the job level:

```yaml
build:
  outputs:
    output-name: ${{ steps.publish.outputs.reference-name }}
    script-file: ${{ steps.publish.outputs.script-file }}
```

The structure is:

```text
Step Output
    ↓
steps.publish.outputs.script-file
    ↓
Job Output
    ↓
build.outputs.script-file
```

So:

```yaml
outputs:
  script-file: ${{ steps.publish.outputs.script-file }}
```

means:

> Take the `script-file` output from the `publish` step and expose it as a job output called `script-file`.

---

# 6. Using an Output in Another Job

To use an output from another job, the receiving job needs to specify that it depends on the producing job.

Example:

```yaml
deploy:
  needs: [test, build]
```

Then the output can be accessed using:

```yaml
${{ needs.build.outputs.script-file }}
```

Example:

```yaml
deploy:
  needs: [test, build]
  runs-on: ubuntu-latest

  steps:
    - name: Get Build Artifacts
      uses: actions/download-artifact@v4
      with:
        name: dist-files

    - name: Output File Name
      run: echo "${{ needs.build.outputs.script-file }}"

    - name: List Files
      run: ls -a
```

---

# 7. Understanding `needs`

This:

```yaml
deploy:
  needs: [test, build]
```

means:

```text
test ───────┐
            │
            ↓
          deploy
            ↑
            │
build ──────┘
```

The `deploy` job depends on both `test` and `build`.

It also allows `deploy` to access outputs from those jobs.

For example:

```yaml
${{ needs.build.outputs.script-file }}
```

means:

```text
needs
  ↓
build
  ↓
outputs
  ↓
script-file
```

---

# 8. Output vs. Artifact

This is an important distinction.

## Output

Used for **small pieces of information**:

```text
version = 1.2.0
script-file = hello-world.js
reference-name = production
```

Example:

```yaml
${{ needs.build.outputs.script-file }}
```

Think:

```text
"Here is some information you need."
```

---

## Artifact

Used for **files and folders**:

```text
dist/
├── index.html
├── app.js
└── assets/
```

Example:

```yaml
uses: actions/upload-artifact@v4
```

and later:

```yaml
uses: actions/download-artifact@v4
```

Think:

```text
"Here are the actual files you need."
```

---

# 9. Complete Concept

A build job can produce **both outputs and artifacts**.

```text
                    Build Job
                       │
              ┌────────┴────────┐
              │                 │
              ↓                 ↓
          Job Output          Artifact
              │                 │
              │                 │
       "hello-world.js"       dist/
              │                 │
              ↓                 ↓
          Deploy Job
              │
       ┌──────┴──────┐
       ↓             ↓
    Output        Artifact
       │             │
       ↓             ↓
  File name       Actual files
```

So:

```text
Output
→ small information

Artifact
→ actual files/folders
```

---

# 10. Quick Syntax Reference

### Create step output

```bash
echo "name=value" >> "$GITHUB_OUTPUT"
```

### Give the step an ID

```yaml
- name: Publish
  id: publish
```

### Access step output

```yaml
${{ steps.publish.outputs.name }}
```

### Expose it as a job output

```yaml
outputs:
  output-name: ${{ steps.publish.outputs.name }}
```

### Access job output from another job

```yaml
${{ needs.build.outputs.output-name }}
```

### The receiving job needs the producing job

```yaml
deploy:
  needs: build
```

or:

```yaml
deploy:
  needs: [test, build]
```

---

# 11. Mental Model

Remember the flow:

```text
Step
  │
  │ $GITHUB_OUTPUT
  ↓
Step Output
  │
  │ steps.<step-id>.outputs.<output-name>
  ↓
Job Output
  │
  │ needs.<job-name>.outputs.<output-name>
  ↓
Another Job
```

The key idea is:

> **`$GITHUB_OUTPUT` creates the step output, `jobs.<job>.outputs` exposes it as a job output, and `needs.<job>.outputs` lets another job consume it.**
