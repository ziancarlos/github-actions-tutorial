Creating an environment variables in github actions is very important

in github actions we can define an env variables in workflow level or job level
In this case we have created an 1 env variables in workflow level and 1 env variable in env level


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
      - name: get code
        uses: actions/checkout@v3
      - name: Cache Dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: deps-node-modules-${{hashFiles('**/package-lock.json')}}
      
      - name: Install Dependency
        working-directory: 4. Job Artifacts And Output/starting-project
        run: npm ci