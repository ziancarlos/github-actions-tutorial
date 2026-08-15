# Events (Workflow Triggers)

**Events** determine **when a workflow should be triggered**.

## 1. Repository-Related Events

These events are triggered by activities related to the repository.

* `push`

  * Triggered when code is pushed.
  * **Mostly used.**

* `pull_request`

  * Triggered by Pull Request activity.

* `create`

  * Triggered when a branch or tag is created.

* `fork`

  * Triggered when the repository is forked.

* `issues`

  * Triggered by issue activity.

* `issue_comment`

  * Triggered when someone comments on an issue or Pull Request.

* `watch`

  * Triggered when someone watches the repository.

* `discussion`

  * Triggered by GitHub Discussion activity.

* **Many more events are available.**

---

## 2. Other Events

* `workflow_dispatch`

  * Manually trigger a workflow from the **GitHub Actions UI**.

* `repository_dispatch`

  * Trigger a workflow through a **REST API request**.
  * An external system can call the API to trigger the workflow.

* `schedule`

  * Trigger a workflow based on a **CRON schedule**.

* `workflow_call`

  * Allows a workflow to be **called by another workflow**.
  * Useful for reusable workflows.

---

# Events On Several Types

You can configure a workflow to trigger on **multiple event types**.

For example:

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```

This workflow can be triggered by:

```text
                    ┌── push to main
                    │
Workflow Trigger ───┼── pull request to main
                    │
                    └── manual trigger
```

So `on` does **not have to contain only one event**. A single workflow can respond to **multiple types of events**.
