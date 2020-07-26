# Workflow Status Action 

This action returns the workflow status (Success, Cancelled, Failure), in case of failure it also returns the failed job name and the failed job step name, also a color hex code is returned  for each one of the workflow statuses (it's optional may be used for slack intergration).

This action should run as part of the final job in the workflow (it doesn't report on the job it's currently running, doesnt report itself).
the job the action runs should "need" all the other jobs in the workflow, that way it will run after all other jobs have finished.
the job "workflow-status" runs in should rin with "if: always()" clause, that way it will run and report on failure as well.
the job assumes workflow success, and changes its status on first job that report's a "cancel" or "failure" state as a final conclustion.

## Inputs
 
| Name              | Required | Description                                                                                                            |
| :---              |   :---:  | :---                                                                                                                   |
| workflow_name     | required | The name of the workflow we are curently running. Default:      `"${{ github.workflow }}"`.                            |
| github_run_id     | required | A unique number for each run within a repository. This number does not change if you re-run the workflow run. Default:        `"${{ github.run_id }}"`.                                                                                                                              | 
| github_repository | required | The owner and repository name. e.g, pixellot/Hello-World. Default: `"${{ github.repository }}"`.                       |     
| github_token      | required | A token to authenticate on behalf of the GitHub App installed on your repository.           `"${{ github.token }}"`.   |


## Outputs

| Name                  | Description                                                                               |
| :---                  | :---                                                                                      |
| workflow_result       | The result of the current workflow run (Success, Cancelled, Failure).                     |
| failed_job            | The name of the job that was failed (only if workflow conclusion is Failure).             |
| failed_step           | The name of the step that was failed (only if workflow conclusion is Failure).            | 
| notification_color    | The color for A slack notification (Green - Success, Yelllow - Cancelled, Red - Failure). |
| notification_icon     | An icon for A slack notification.                                                         |

## Example usage
```yaml
# ci-WORKFLOW-yaml

name: Node.js CI
on:
  push:
    branches: [ master ]    
jobs:

int:
  name: ESLint
  runs-on: ubuntu-latest
  ...

test:
  name: Coverage
  needs: lint
  strategy:
    matrix:
      node: ['11', '12']
  ...

publish:
  name: Publish Package
  needs: test
  if: startsWith(github.ref, 'refs/tags/v')
  ...

test-slack:
  name: slack-test
  runs-on: ubuntu-latest
  needs: [int, test, publish]                 # Should need all the jobs in the workflow, that way it will run only after all other jobs.
  if: always()                                # this will make sure the job will run and report on failure as well.
  steps:
  
  - name: Workflow Status 
    id: workflow-status
    uses: pixellot/workflow-status@master
    with:
      workflow_name:  ${{ github.workflow }}
      github_run_id: ${{ github.run_id }}
      github_repository: ${{ github.repository }}
      github_token: ${{ github.token }}

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
