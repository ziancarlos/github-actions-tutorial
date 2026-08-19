# Reusable Workflows

A reusable workflow allows us to define a workflow once and call it from another workflow.

Instead of defining all steps directly:

```yaml
jobs:
  deploy:
    steps:
      ...
```

we can put the deployment logic into another workflow and call it using the `uses` keyword:

```yaml
jobs:
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml
```

When using a reusable workflow with `uses`, we don't define `steps` inside that job because the steps are already defined inside the reusable workflow.

---

# 1. Creating a Reusable Workflow

A reusable workflow uses:

```yaml
on:
  workflow_call:
```

Example:

```yaml
name: Reusable Workflow

on:
  workflow_call:

jobs:
  info:
    runs-on: ubuntu-latest

    steps:
      - name: Echoing
        run: echo "Deploying And Uploading"
```

Then another workflow can call it:

```yaml
jobs:
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml
```

---

# 2. Passing Inputs

We can define inputs under `workflow_call`.

```yaml
name: Reusable Workflow

on:
  workflow_call:
    inputs:
      file_name:
        description: File name
        default: dist
        type: string

jobs:
  info:
    runs-on: ubuntu-latest

    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with:
          name: ${{ inputs.file_name }}

      - name: Echo
        run: ls -a
```

Inside the reusable workflow, we access the input using:

```yaml
${{ inputs.file_name }}
```

The calling workflow passes the input using `with`:

```yaml
jobs:
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml

    with:
      file_name: dist-result
```

The flow is:

```text
Calling Workflow
      |
      | with:
      |   file_name: dist-result
      ↓
Reusable Workflow
      |
      ↓
inputs.file_name
      |
      ↓
dist-result
```

---

# 3. Passing Secrets

If the reusable workflow needs sensitive information, use `secrets` instead of normal inputs.

Define the secret under `workflow_call`:

```yaml
on:
  workflow_call:
    inputs:
      file_name:
        description: File name
        default: dist
        type: string

    secrets:
      secret_name:
        required: true
```

Then the calling workflow passes the secret using `secrets`:

```yaml
jobs:
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml

    with:
      file_name: dist-result

    secrets:
      secret_name: ${{ secrets.SECRET_NAME }}
```

Inside the reusable workflow, access it using:

```yaml
${{ secrets.secret_name }}
```

Remember:

```text
with:
  → Inputs

secrets:
  → Secrets
```

Sensitive values should be passed using the `secrets` mechanism rather than normal inputs.

---

# 4. Returning Outputs from a Reusable Workflow

A reusable workflow can also return a value to the workflow that called it.

There are three levels involved:

```text
Step Output
    ↓
Job Output
    ↓
Reusable Workflow Output
    ↓
Calling Workflow
```

---

# 5. Step Output

First, create an output inside a step.

```yaml
- name: Result Output
  id: step-result
  run: echo "return_name=success" >> "$GITHUB_OUTPUT"
```

The step has:

```yaml
id: step-result
```

and creates:

```text
return_name = success
```

We can access this output using:

```yaml
${{ steps.step-result.outputs.return_name }}
```

So:

```text
steps.step-result.outputs.return_name
                ↓
             success
```

---

# 6. Job Output

A step output cannot directly become a reusable workflow output.

First, expose the step output as a job output:

```yaml
jobs:
  info:
    runs-on: ubuntu-latest

    outputs:
      outcome: ${{ steps.step-result.outputs.return_name }}

    steps:
      - name: Result Output
        id: step-result
        run: echo "return_name=success" >> "$GITHUB_OUTPUT"
```

Now the relationship is:

```text
Step Output
steps.step-result.outputs.return_name
                ↓
              success
                ↓
Job Output
jobs.info.outputs.outcome
                ↓
              success
```

---

# 7. Reusable Workflow Output

Now expose the job output through `workflow_call.outputs`.

```yaml
on:
  workflow_call:

    outputs:
      return_name:
        description: Return name
        value: ${{ jobs.info.outputs.outcome }}
```

This tells GitHub Actions:

> Take the `outcome` output from the `info` job and expose it as the reusable workflow output called `return_name`.

The complete chain is:

```text
Step Output
steps.step-result.outputs.return_name
                ↓
Job Output
jobs.info.outputs.outcome
                ↓
Reusable Workflow Output
on.workflow_call.outputs.return_name
```

---

# 8. Complete Reusable Workflow

```yaml
name: Reusable Workflow

on:
  workflow_call:

    inputs:
      file_name:
        description: File name
        default: dist
        type: string

    outputs:
      return_name:
        description: Return name
        value: ${{ jobs.info.outputs.outcome }}

jobs:
  info:
    runs-on: ubuntu-latest

    outputs:
      outcome: ${{ steps.step-result.outputs.return_name }}

    steps:
      - name: Download Artifact
        uses: actions/download-artifact@v4
        with:
          name: ${{ inputs.file_name }}

      - name: Echo
        run: ls -a

      - name: Result Output
        id: step-result
        run: echo "return_name=success" >> "$GITHUB_OUTPUT"
```

---

# 9. Accessing the Output from the Calling Workflow

Now we call the reusable workflow:

```yaml
jobs:
  deploy:
    needs: [build]

    uses: ./.github/workflows/twelve-workflow.yml

    with:
      file_name: dist-result
```

The job calling the reusable workflow is named:

```yaml
deploy:
```

Therefore, we can access the reusable workflow's outputs using the `needs` context:

```yaml
${{ needs.deploy.outputs.return_name }}
```

For example:

```yaml
print-deploy-result:
  needs: [deploy]

  runs-on: ubuntu-latest

  steps:
    - name: Echoing
      run: echo "${{ needs.deploy.outputs.return_name }}"
```

This will output:

```text
success
```

---

# 10. The Important Output Chain

The most important thing to remember is the output chain:

```text
Inside Reusable Workflow

STEP
steps.step-result.outputs.return_name
        ↓
        ↓
JOB
jobs.info.outputs.outcome
        ↓
        ↓
WORKFLOW
workflow_call.outputs.return_name
        ↓
        ↓
CALLING WORKFLOW
needs.deploy.outputs.return_name
```

Visualized:

```text
┌──────────────────────────────────────┐
│ Reusable Workflow                   │
│                                      │
│ Step                                 │
│ steps.step-result.outputs.return_name│
│              ↓                       │
│ Job                                  │
│ jobs.info.outputs.outcome            │
│              ↓                       │
│ workflow_call.outputs.return_name    │
└──────────────┬───────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ Calling Workflow                     │
│                                      │
│ needs.deploy.outputs.return_name     │
│                                      │
│              ↓                       │
│           "success"                  │
└──────────────────────────────────────┘
```

So this:

```yaml
run: echo "${{ needs.deploy.outputs.return_name }}"
```

works because:

1. `deploy` is the job that calls the reusable workflow.
2. The reusable workflow exposes an output called `return_name`.
3. The calling workflow accesses that output through `needs.deploy.outputs.return_name`.

---

# Key Concepts

```text
Reusable Workflow
→ A workflow that can be called by another workflow.

workflow_call
→ Trigger used to make a workflow reusable.

with
→ Used to pass normal inputs.

secrets
→ Used to pass sensitive values.

inputs.<name>
→ Used to access inputs inside the reusable workflow.

secrets.<name>
→ Used to access secrets inside the reusable workflow.

Step Output
→ Created using $GITHUB_OUTPUT.

Job Output
→ Exposes a step output at the job level.

workflow_call Output
→ Exposes a job output to the workflow that called it.

needs.<job>.outputs.<output>
→ Used by the calling workflow to access the reusable workflow's output.
```

The overall concept is:

```text
Calling Workflow
       |
       | with / secrets
       ↓
Reusable Workflow
       |
       | inputs / secrets
       ↓
Steps
       |
       | step output
       ↓
Job Output
       |
       | workflow_call output
       ↓
Calling Workflow
       |
       ↓
needs.deploy.outputs.return_name
```
