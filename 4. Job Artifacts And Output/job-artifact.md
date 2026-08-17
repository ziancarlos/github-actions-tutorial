Understanding Artifact
Basically artifact is a object made or used by humans

So in building and deploying projects normally we need 
1. build and store the binary file
2. deploy the project with stored binary file

and build and deploy has in 2 different jobs
so we need to store the artifacts or binary file
and deploy the project where we get those artifacts

so inside jobs we can use this
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Install Dependency
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm ci
      - name: Test Code
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm run build
      - name: Upload Artifacts
        uses: actions/upload-artifact@v4
        with: 
          name: dist-files
          path: |
            4. Job Artifacts And Output/starting-project/dist
use actions to upload an artifact where we can specify which path we want to store
and when we run this in the workflow runner we can get to download the specified file.