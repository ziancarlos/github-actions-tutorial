in actions we can create an reusable workflows

with the on is workflow_call. just create a workflow as usual with on as workflow_call

name: Reusable Workflow
on: workflow_call
jobs: 
  info:
    runs-on: ubuntu-latest
    steps:
      - name: Echoing
        run: echo "Deploying And Uploading"

by that we can just call the workflow in a jobs using uses keywoard without defining steps: contoh
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml