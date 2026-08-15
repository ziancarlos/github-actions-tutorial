Events (Workflow Triggers)

1. Repository Related
    - push (Mostly used)
    - pull_request
    - create
    - fork
    - issues
    - issue_comment
    - watch
    - discussion
    - many more
2. Other
    - workflow_dispatch
        - Manually trigger workflow (github ui)
    - repository_dispatch
        - REST API request trigger workflows (given a link to call the api)
    - schedule
        - By CRON 
    - workflow_call
        - Called by others worklfows