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

to use a environment variables we can access env object inside ${{}} to read the env variables
from that we can access workflow scope env variables or job scope env variables.

BUT KEEP IN MIND JOB LEVEL ENV VARIABLES ONLY CAN BE ACCESS BY THE JOB INSIDE ONLY NOT ANOTHER JOB. 
unless it will be blank.

  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Echo 1
        run: echo "${{env.MONGODB_USERNAME}}"
      - name: Echo 2
        run: echo "${{env.MONGO_DB_NAME}}"


On the other hand u will not save ur env variables like db_password inside a workflow hard code code. so we must use secrets. to create a secret in repository layer we goto repo settings -> secrets and variables -> actions -> New Repository Secrets

After Creating it we can consume inside the workflow with object called ${{secrets.SECRET_NAME}}

with example below
name: Eight Workflow
on: 
  push: 
    branches:
    - main
    - dev
env:
  MONGO_DB_NAME: ${secrets.MONGO_DB_NAME}
jobs:
  test:
    env:
      MONGODB_CLUSTER_ADDRESS: ${secrets.MONGODB_CLUSTER_ADDRESS}
      MONGODB_USERNAME: ${secrets.MONGODB_USERNAME}
      MONGODB_PASSWORD: ${secrets.MONGODB_PASSWORD}
      PORT: ${secrets.PORT}
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
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm ci
      - name: Run Server
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Test
        working-directory: 5. Environment Variables And Secret/starting-project
        run: npm run test
      - name: Echo
        run: echo "${{env.MONGODB_USERNAME}}"
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Echo 1
        run: echo "${{env.MONGODB_USERNAME}}"
      - name: Echo 2
        run: echo "${{env.MONGO_DB_NAME}}"