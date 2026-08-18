we have this case where
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 6. Controlling Workflow And Jobs/starting-project/node_modules
          key: node-modules-${{hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm ci
      - name: TestCode
        id: trigger-test
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run test
      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: 6. Controlling Workflow And Jobs/starting-project/test.json


Upload Test Report wont be run when trigger-test is failing. while it does not make sense. we want also to retrieve the report when test is failing or not. using the default code above. the upload test report wont be runned. by that we need to controll workflow or jobs. by default if a steps is failing the rest of the steps forward will not be runned.

default behaviour is TestCode (Success) -> Upload Test Report(Triggered)
TestCode (Not Success) -> Upload Test Report(Not Triggered)

what we want is
TestCode (Success) -> Upload Test Report(Triggered)
TestCode (Not Success) -> Upload Test Report(Not Triggered)


by that we introduce if in every steps
if keywoard can handle boolean value like other programming language and can use && or || sign like every language.

from if statement we can use step context object to get the outcome from steps with what id
https://docs.github.com/en/actions/reference/workflows-and-actions/contexts#steps-context
where we can know for steps before this test

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 6. Controlling Workflow And Jobs/starting-project/node_modules
          key: node-modules-${{hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm ci
      - name: TestCode
        id: trigger-test
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run test
      - name: Upload Test Report
        if:  steps.trigger-test.outcome == 'failure'  
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: 6. Controlling Workflow And Jobs/starting-project/test.json


but this were not enough. cause github actions yes it checks the outcome but it will still ignoring the steps because previous step were failed. (default behaviour). we need to change the default behaviour where it execute when the previous steps is failling.
when we add failure() it will execute when jobs are failing. and it return true or false.
https://docs.github.com/en/actions/reference/workflows-and-actions/expressions#status-check-functions

    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 6. Controlling Workflow And Jobs/starting-project/node_modules
          key: node-modules-${{hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: TestCode
        continue-on-error: true
        id: trigger-test
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run test
      - name: Upload Test Report
        if: failure() &&  steps.trigger-test.outcome == 'failure'  
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: 6. Controlling Workflow And Jobs/starting-project/test.json



failure() returns true when jobs/steps fails
success() returns true when previous steps succeed
always() always return true
cancelled() return true if cancelled workflow

but conditional not only work at steps level but job level
we can add create a report when lint or any children or deploy has failure.

  report:
    if: failure()
    needs: [lint, deploy]
    runs-on: ubuntu-latest
    steps:
      - name: Output Information
        run: |
          echo "Something when wrong"
          echo "${{toJSON(github)}}"


oke with the usecase above when using if it will mark the job as still error when the next step is still running. so there is continue-on-error. 
where if its set on true. when that steps use continue-on-error true an it occurs an error it will still run the test and put steps jobs as success.
unlike if with failure() and conditions

  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 6. Controlling Workflow And Jobs/starting-project/node_modules
          key: node-modules-${{hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: TestCode
        continue-on-error: true
        id: trigger-test
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run test
      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: 6. Controlling Workflow And Jobs/starting-project/test.json


ok next the case where we already cache the dependency but it still run npm install, we can use steps.cache-name.outputs.cache-hit != 'true'
is the cache success to hit? if yes dont run this code
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache Dependenies
        id: cache
        uses: actions/cache@v4
        with:
          path: 6. Controlling Workflow And Jobs/starting-project/node_modules
          key: node-modules-${{hashFiles('6. Controlling Workflow And Jobs/starting-project/package-lock.json')}}
      - name: Install Dependencies
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci
      - name: TestCode
        continue-on-error: true
        id: trigger-test
        working-directory: 6. Controlling Workflow And Jobs/starting-project
        run: npm run test
      - name: Upload Test Report
        uses: actions/upload-artifact@v4
        with:
          name: test-report
          path: 6. Controlling Workflow And Jobs/starting-project/test.json