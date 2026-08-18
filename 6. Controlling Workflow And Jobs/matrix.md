What if inside a jobs i want to test to run on every OS and language version
example: i want to try ubuntu os with node version 11,12,13 and windows os with node version 11,12,13 will it succeed to build?

we can use strategy matrix, we can store a variable with multiple values and it will do like an hyperparameter tuning with a given value. 
on this example it will try to build a code with ubuntu and node ver 18,19, 20 and window with 18,19, 20. it will create 6 jobs in total. to test this code

this will be helpfull to test our code is it compatible with every environments. 

key in matrix name are up to u. u can use whatever u want.
name: Eleven Workflow
on:
  push:
    branches:
      - main

jobs:
    build:
      strategy:
        matrix: 
          node-version: [18,19,20]
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

but we also can include an combination or exclude a combination.
this means it will include 21 node version WITH ubuntu latest. but it will not test node version 21 with windows latest. and also we can exclude a combination where i exclude node ver 18 and windows latest BUTTT 18 with os ubuntu will be still testing
name: Eleven Workflow
on:
  push:
    branches:
      - main

jobs:
    build:
      strategy:
        matrix: 
          node-version: [18,19,20]
          os: [ubuntu-latest, windows-latest]
          include:
            - node-version: 21
              os: ubuntu-latest
          exclude:
            - node-version: 18
              os: windows-latest
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