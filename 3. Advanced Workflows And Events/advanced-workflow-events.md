Event Activity Types & Filter

Event -> Push, Pull Request

But This Events Has More Detail Event Activity Types
Such As
Pull (Event) -> Opened, Closed, Edited
but remember the checkout code inside the workflow will use pr branch from merge with pr branch to so this is the code that will be checkout with.
to specify a branch we can specify inside the checkout using with keywoard
eg:
  pull_request:
    types:
      - opened

where inside types: we can have multiplt activity types

but inside pull rwequest we can specify the event with branches. where it will trigger the workflow if target branch as specified eg:
  pull_request:
    types:
      - opened
    branches:
      - main

where if target branch is main dev -> main then it will trigger what if deva-> devb. it will not triggered the workflow. on the other hand remember to specify the checkout if u need a certain code. unless the code checkouted will be merged from source to target 

Filters 
Push (Event) -> Filter Based On What Branch?
on the other hand push event will trigger when we are pusing. but we can specify a certain branch to trigger the workflow. eg:
  push:
    branches:
      - main
      - 'dev-*'
      - 'feat/**'
the first will trigger if ur pushing to main branch
second will trigger if ur pushing to dev branch with postfix -anycharacterwithoutslash
third will trigger ur pushing to feat branch with postfix/and/any/other/slashes

inside push filters activity
  push:
    branches:
      - main
      - 'dev-*'
      - 'feat/**'
    paths-ignore:
      - './.github/workflows/*'
we can add paths or paths ignore 
1. paths ignore -> where if a specified dir/file has changes ignore the workflow. other than the specified paths trigger the workflow
2. paths -> where if a specified dir/file has changes trigger workflow. other than the specified paths dont trigger the workflow

branch that are used in push is branch that are being triggered if main the workflow checkout will use main


cancelling an workflow can be done through github repository -> action -> which workflow -> choose cancel workflow


skiping a workflow -> also can be done through commit message with this
git add <<file_name>>
git commit -m "comments description [skip ci]"
git push origin <<branch_name>>

with additional postfix of in commit message [skip ci] -> the workflows can be skipped