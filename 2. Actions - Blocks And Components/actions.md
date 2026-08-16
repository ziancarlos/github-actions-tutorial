Actions
- In Github Workflows are called actions. but they truely meant workflow
- Action Truely means a custom application that perform a (typically complex) frequently a repeated task
- Eg: Downloading github repo to a runner machine
- Alternative of actions are command run command inside workflow of yaml file
- normally run command define a simple command
- Better use actions on more complex cases
- In case like takes code from github repository and puts into github actions runner. yes we can use run command and try to git clone or pull the repository. but it will take several steps. on the other hand we can use https://github.com/marketplace/actions/checkout actions name checkout where it automatically helps us getting out repository inside our workflow runners.


When creating a workflows with checkout
instead of run keywoard in the steps we use uses keywoard to use a actions
in uses there is with keywoard to configure the actions. but if we need a simple task just use without use keywoard  
name: Test Workflow
on: workflow_dispatch
jobs: 
  first-job: 
    runs-on: ubuntu-latest
    steps:
      - name: LS
        uses: actions/checkout@v3


or another example when we need node.js or any programming language actions we can use action
to install node.js/. the with keywoard is to install what node.js version we want. on the other hand now u know how to configure the actions 
name: Test Workflow
on: workflow_dispatch
jobs: 
  first-job: 
    runs-on: ubuntu-latest
    steps:
      - name: Get Repository
        uses: actions/checkout@v3
      - name: Install Node
        uses: actions/setup-node@v3
        with: 
          node-version: 18  

