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


but we also can pass a input to an reusable workflow using inputs -> any_name with property description, default and type
name: Reusable Workflow
on: 
  workflow_call:
    inputs:
      file_name: 
        description: file_name
        default: dist
        type: string
jobs: 
  info:
    runs-on: ubuntu-latest
    steps:
      - name: Echoing
        uses: actions/download-artifact@v3
        with:
          name: ${{inputs.file_name}}
      - name: echo
        run: ls -a

while using it inside the reusable workflow is just to call inputs.<<variable_name>> object

to pass an input from a jobs is using with keywoard with the variable_name
  deploy:
    needs: [build]
    uses: ./.github/workflows/twelve-workflow.yml
    with:
      file_name: dist-result
  report:
    if: failure()
    needs: [lint, deploy]
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something when wrong"
          echo "${{toJSON(github)}}"
