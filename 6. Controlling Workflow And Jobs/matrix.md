What if inside a jobs i want to test to run on every OS and language version
example: i want to try ubuntu os with node version 11,12,13 and windows os with node version 11,12,13 will it succeed to build?

we can use strategy matrix, we can store a variable with multiple values and it will do like an hyperparameter tuning with a given value. 
on this example it will try to build a code with ubuntu and node ver 11,12,13 and window with 11,12,13.
name: Eleven Workflow
on:
  push:
    branches:
      - main

jobs:
    build:
      strategy:
        matrix: 
          node-version: [11,12,13]
          os: [ubuntu-latest, windows-latest]
      runs-on: ${{matrix.os}}
      steps:
        - name: Get Code
          uses: actions/checkout@v3
        - name: Install Node
          uses: actions/setup-node@v3
          with:
            node-version: ${{matrix.node-version}}
        - name: Install Dependencies
          working-directory: 6. Controlling Workflow And Jobs/starting-project
          run: npm ci
        - name: TestCode
          working-directory: 6. Controlling Workflow And Jobs/starting-project
          run: npm run build