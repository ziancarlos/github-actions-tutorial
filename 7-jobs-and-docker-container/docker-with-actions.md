Ok in actions we can containerize our jobs
for just specifying what base image we want? we can wrap the steps inside the docker container

we also can pass an env variables inside container by simply adding env keywoard

by that test jobs are running inside node container with version 18
jobs:
  test:
    environment: test
    runs-on: ubuntu-latest
    container: 
        node:18
        env: 
    env: 
      MONGODB_CONNECTION_PROTOCOL: mongodb+srv
      MONGODB_CLUSTER_ADDRESS: cluster0.13dgjsx.mongodb.net
      MONGODB_USERNAME: ${{secrets.MONGODB_USERNAME}} 
      MONGODB_PASSWORD: ${{secrets.MONGODB_PASSWORD}}
      PORT: 8080
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 7-jobs-and-docker-container/starting-project/node_modules
          key: node-modules-${{hashFiles('7-jobs-and-docker-container/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 7-jobs-and-docker-container/starting-project
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: Run Server
        working-directory: 7-jobs-and-docker-container/starting-project
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run Test
        working-directory: 7-jobs-and-docker-container/starting-project
        run: npm test

Mental Mode
WITH CONTAINER

Ubuntu
  ↓
Docker Container
  ↓
Node 18
  ↓
Your code

WITH CONTAINER

Ubuntu
  ↓
Docker Container
  ↓
Node 18
  ↓
Your code

So The whole actions are running inside container that use node ver 18. Not Ubuntu that has version 18 node and run the code inside. it isolate the whole code into a container.

The real benefit 
without container -> it only says run on actions ubuntu with node installed
with container -> i want this same entire environment that runs on NODE ver 18


and we can create an extra services/container inside a jobs. where it will create an docker container for node to run all those jobs. but on the other hand it will create a mongodb service (container) where the test database will be refer to the actions service. remember this is good for testing phase. where this service are going to be exists at the lifetime of github actions jobs. when jobs are done this services are gone.

name: Deployment With Docker
on: 
  push:
    branches:
      - main
      - dev
env:
  MONGODB_DB_NAME: gha_demo
jobs:
  test:
    environment: test
    runs-on: ubuntu-latest
    container: node:18
    env: 
      MONGODB_CONNECTION_PROTOCOL: mongodb
      MONGODB_CLUSTER_ADDRESS: mongodb
      MONGODB_USERNAME: root 
      MONGODB_PASSWORD: example
      PORT: 8080
    services:
      mongodb: <- title here as a host name
        image: mongo:latest
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
    steps:
      - name: Checkout Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 7-jobs-and-docker-container/starting-project/node_modules
          key: node-modules-${{hashFiles('7-jobs-and-docker-container/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 7-jobs-and-docker-container/starting-project
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: Run Server
        working-directory: 7-jobs-and-docker-container/starting-project
        run: npm start & npx wait-on http://127.0.0.1:$PORT
      - name: Run Test
        working-directory: 7-jobs-and-docker-container/starting-project
        run: npm test

same as docker compose connecting/communicate between container/services insdie jobs just use the title as host name it already has the same networks