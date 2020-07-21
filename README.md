# Workflow Status Action 

This action returns the workflow status (Success, Cancelled, Failure), in case of failure it also returns the failed job name and the failed job step name, also a color hex code is returned  for each one of the workflow statuses (it's optional may be used for slack intergration).
This action should run in the final job of the workflow (it doesn't report on the job it's currently running, doesnt report itself).
the job assumes workflow success, and changes its status on first job that report's a "cancel" or "failure" state as a final conclustion.

## Inputs

| Name             | Required | Description                                                                                            |
| :---             |   :---:  | :---                                                                                                   |
| workflow_name    | required | The name of the workflow we are curently running. Default: `"<placeholder-workflow-name>"`.            |
| github_run_id    | required | A unique number for each run within a repository. This number does not change if you re-run the workflow run. Default: `"<placeholder-github_run_id>"`.                                                                                                         |
| github_repository| required | he owner and repository name. e.g, pixellot/Hello-World. Default: `"<placeholder-github-repository>"`. |        


## Outputs

| Name                  | Description                                                                               |
| :---                  | :---                                                                                      |
| workflow_result       | The result of the current workflow run (Success, Cancelled, Failure).                     |
| failed_job            | The name of the job that was failed (only if workflow conclusion is Failure).             |
| failed_step           | The name of the step that was failed (only if workflow conclusion is Failure).            | 
| notification_color    | The color for A slack notification (Green - Success, Yelllow - Cancelled, Red - Failure). |

## Example usage
```yaml
# ci-WORKFLOW-yaml

name: Node.js CI
on:
  push:
    branches: [ master ]    
jobs:

test-slack:
  name: slack-test
  runs-on: ubuntu-latest
  needs: []
  if: always()
  steps:
  
  - name: Workflow Status 
    id: workflow-status
    uses: pixellot/workflow-status@master
    with:
      workflow_name:  ${{ github.workflow }}
      github_run_id: ${{ github.run_id }}
      github_repository: ${{ github.repository }}

  - name: Slack Notification
        uses: rtCamp/action-slack-notify@master
        env:
          SLACK_WEBHOOK: '${{ secrets.SLACK_URL }}'
          SLACK_CHANNEL: 'github-action-slack'
          SLACK_COLOR: '${{ steps.workflow-Status.outputs.notification_color }}'
          SLACK_ICON: https://github.githubassets.com/images/modules/logos_page/Octocat.png?size=48
          SLACK_MESSAGE: "Workflow *${{ steps.workflow-Status.outputs.workflow_result }}*\nJob: ${{ steps.workflow-Status.outputs.failed_job }}\nStep: ${{ steps.workflow-Status.outputs.failed_step }}"
          SLACK_TITLE: 'Status:'
          SLACK_USERNAME: GitHub Action
          SLACK_FOOTER: '${{ github.workflow }}#${{ github.run_number }}'    
    
```
