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