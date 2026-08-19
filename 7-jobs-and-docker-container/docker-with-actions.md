# GitHub Actions: Containers and Services

## Containerizing Jobs

In GitHub Actions, we can containerize a job by specifying a Docker image with the `container` keyword.

This allows us to wrap all the steps of the job inside the specified Docker container.

For example:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container: node:18
```

This means the `test` job runs inside a container created from the `node:18` image.

We can also define environment variables for the job:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    container: node:18
    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb+srv
      MONGODB_CLUSTER_ADDRESS: cluster0.13dgjsx.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
```

The environment variables under the job-level `env` are available to the steps running inside the container.

## Mental Model

### With Container

```text
GitHub Actions Runner (Ubuntu)
        ↓
Docker Container
        ↓
Node 18
        ↓
Your code / job steps
```

### Without Container

```text
GitHub Actions Runner (Ubuntu)
        ↓
Node 18 installed on the runner
        ↓
Your code / job steps
```

The important difference is that `container: node:18` does not simply mean "use Node 18."

It means:

> Run the entire job inside the `node:18` Docker environment.

With `setup-node`, we are changing the Node version on the existing GitHub Actions runner.

With `container`, we are changing the environment in which the job executes.

## Example

```yaml
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
      MONGODB_CONNECTION_PROTOCOL: mongodb+srv
      MONGODB_CLUSTER_ADDRESS: cluster0.13dgjsx.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }}
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        id: cache
        uses: actions/cache@v4
        with:
          path: 7-jobs-and-docker-container/starting-project/node_modules
          key: node-modules-${{ hashFiles('7-jobs-and-docker-container/starting-project/package-lock.json') }}

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
```

---

# Services in GitHub Actions

GitHub Actions also allows us to create additional containers for services that our job needs.

For example:

```yaml
services:
  mongodb:
    image: mongo:latest
    env:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
```

This creates a separate MongoDB container.

The job now has two containers:

```text
GitHub Actions Runner
│
├── Job Container: node:18
│   ├── Checkout
│   ├── npm ci
│   ├── npm start
│   └── npm test
│
└── Service Container: mongo:latest
    └── MongoDB
```

The job container is where our application and test steps run.

The service container provides an external dependency, such as MongoDB, Redis, PostgreSQL, RabbitMQ, etc.

## Service Container Environment Variables

Environment variables defined under the service belong to the service container:

```yaml
services:
  mongodb:
    image: mongo:latest
    env:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
```

These are different from the environment variables of the job:

```yaml
env:
  MONGODB_USERNAME: root
  MONGODB_PASSWORD: example
  PORT: 8080
```

The MongoDB container receives:

```text
MONGO_INITDB_ROOT_USERNAME=root
MONGO_INITDB_ROOT_PASSWORD=example
```

while the application/job container receives:

```text
MONGODB_USERNAME=root
MONGODB_PASSWORD=example
PORT=8080
```

---

# Connecting to a Service Container When Using a Job Container

When the job itself runs inside a container, the service can be accessed using the service name as the hostname.

For example:

```yaml
services:
  mongodb:
    image: mongo:latest
```

The service name is:

```text
mongodb
```

Therefore, the application can use:

```yaml
env:
  MONGODB_CONNECTION_PROTOCOL: mongodb
  MONGODB_CLUSTER_ADDRESS: mongodb
```

The important idea is:

```text
Job Container
    │
    │ hostname: mongodb
    ↓
MongoDB Service Container
```

This is similar to Docker Compose, where containers can communicate using the service name as the hostname.

For example, in Docker Compose:

```yaml
services:
  app:
    ...

  mongodb:
    ...
```

The application can communicate with MongoDB using:

```text
mongodb
```

GitHub Actions provides similar networking behavior for job containers and service containers.

## Full Example: Job Container + MongoDB Service

```yaml
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
      mongodb:
        image: mongo:latest
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        id: cache
        uses: actions/cache@v4
        with:
          path: 7-jobs-and-docker-container/starting-project/node_modules
          key: node-modules-${{ hashFiles('7-jobs-and-docker-container/starting-project/package-lock.json') }}

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
```

Mental model:

```text
GitHub Actions
│
└── Test Job
    │
    ├── node:18 container
    │   └── Application + Tests
    │
    └── mongo:latest service container
        └── Test Database
```

The service container exists for the lifetime of the GitHub Actions job.

When the job finishes, the service container is removed.

This makes service containers especially useful for CI and integration testing.

---

# Services Without a Job Container

We can also use service containers without defining a job container.

For example:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    services:
      mongodb:
        image: mongo:latest
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
        ports:
          - 27017:27017
```

In this case, the steps run directly on the GitHub Actions runner rather than inside a job container.

Because there is no job-container network where the service can simply be referenced by its service hostname, we can expose the service port to the runner:

```yaml
ports:
  - 27017:27017
```

Then the application can connect through:

```text
127.0.0.1:27017
```

For example:

```yaml
env:
  MONGODB_CONNECTION_PROTOCOL: mongodb
  MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017
  MONGODB_USERNAME: root
  MONGODB_PASSWORD: example
  PORT: 8080
```

## Full Example: Runner + MongoDB Service

```yaml
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

    env:
      MONGODB_CONNECTION_PROTOCOL: mongodb
      MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017
      MONGODB_USERNAME: root
      MONGODB_PASSWORD: example
      PORT: 8080

    services:
      mongodb:
        image: mongo:latest
        env:
          MONGO_INITDB_ROOT_USERNAME: root
          MONGO_INITDB_ROOT_PASSWORD: example
        ports:
          - 27017:27017

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Cache Dependencies
        id: cache
        uses: actions/cache@v4
        with:
          path: 7-jobs-and-docker-container/starting-project/node_modules
          key: node-modules-${{ hashFiles('7-jobs-and-docker-container/starting-project/package-lock.json') }}

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
```

Mental model:

```text
GitHub Actions Runner
│
├── Your Node application
│
└── MongoDB Service Container
        ↑
        │
   Port 27017 exposed
        │
        ↓
127.0.0.1:27017
```

---

# Job Container vs Service Container

## Job Container

```yaml
container: node:18
```

Purpose:

> Where the GitHub Actions job steps run.

Contains:
- Your code
- Node 18
- npm
- Test commands
- Application process

## Service Container

```yaml
services:
  mongodb:
    image: mongo:latest
```

Purpose:

> A dependency that the job needs.

Contains:
- MongoDB
- Database
- Other required services

The key distinction:

```text
container:
    Where the job runs

services:
    Dependencies that the job needs
```

Service containers are especially useful for CI and integration testing because they are temporary and are removed when the GitHub Actions job finishes.