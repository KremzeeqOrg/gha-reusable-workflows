# GitHub Actions Reusuable Workflows

This repo consolidates reusable workflows.

## Terraform Plan

- [workflow](.github/workflows/terraform-plan.yml)
- Produces a Terraform Plan in relation to a target AWS account.

### Typical Workflow Use Case

This workflow should only be used in relation to a PRs raised against the main branch for your repo. When the Terraform plan is generated, it is added as a comment on your PR. With successive commits, these will be shown as edits upon the first comment posted for the PR.

<details>

<summary>Example GitHub Action Workflow Input Details</summary>

```

on:
  pull_request:
    branches:
      - main
    types: [opened, synchronize, reopened]

  terraform-plan:
    name: "Terraform Plan and Comment on PR"
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/terraform-plan.yml@main
    with:
      environment: dev
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.DEV_AWS_ACCOUNT_ACCESS_ROLE }}
```

</details>

## Terraform Plan and Apply

- [workflow](.github/workflows/terraform-plan-and-apply.yml)
- Produces a Terraform Plan in relation to a target AWS account and raises an Issue in your GitHub repo where you are using the workflow.
- In relation to the Issue, you can add a comment to approve or reject the Terraform plan.
- This serves as an approval gate, where you do not need to manually create GitHub environments at your repo level in order to implement approval gate functionality.
- By default, the workflow will enter a polling state for 72 hours, until it times out, due to there being no approval/rejection.
- If the plan is approved, the workflow recommences and the Terraform plan is applied.

### Typical Workflow Use Case

This workflow could be used following a merge of a PR to the main branch of your repo. You could invoke using the workflow twice to deploy to a target dev and prod environments in AWS.

<details>

<summary>Example GitHub Action Workflow Input Details</summary>

```

jobs:
  tf-plan-and-apply-in-dev:
    name: Terraform Plan and Apply in Dev
    permissions:
      id-token: write
      issues: write
      contents: read
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/terraform-plan-and-apply.yml@main
    with:
      environment: dev
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.DEV_AWS_ACCOUNT_ACCESS_ROLE }}

  tf-plan-and-apply-in-prod:
    name: Terraform Plan and Apply in Prod
    needs: tf-plan-and-apply-in-dev
    permissions:
      id-token: write
      issues: write
      contents: read
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/terraform-plan-and-apply.yml@main
    with:
      environment: prod
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.PROD_AWS_ACCOUNT_ACCESS_ROLE }}

```

</details>

## Environment Variables

In relation to your GitHub environment e.g. for `dev`/ `prod`, set the following variables:

| Variable               | Explanation                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| ENV                    | e.g. `dev` / `prod`                                                                                 |
| MINIMUM_APPROVALS\*    | Number of GitHub approvals needed for `terraform plan`                                              |
| TF_BACKEND_CONFIG_FILE | Backend configuration file for Terraform, required for `terraform init`, e.g. `backend-dev.tfvars`  |
| TF_PLAN_APPROVERS\*    | Specify GitHub user e.g. `user` or users e.g. `user1,user2`, who can approve Terraform Plan         |
| TF_VARS_FILE           | Configuration file for environment variables for Terraform. e.g. `tf-vars-dev.tfvars`               |
| TF_VERSION             | Terraform version e.g. run `terraform --version` to check what you're using locally e.g. 1.8.5      |
| TF_WORKING_DIR         | Directory where all terraform files are kept in relation to the root for your repo e.g. `terraform` |

- only required for Terraform Plan and Approve workflow

Remember, GitHub environments can be set at the repo or organization level. Please see [here](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment) for more info.

## Acknowledegments

The workflows comprise of different GitHub Actions below. Please see these GitHub Action projects for further information:

- [actions/checkout](https://github.com/actions/checkout)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [hashicorp/setup-terraform@v3](https://github.com/hashicorp/setup-terraform)
- [borchero/terraform-plan-comment@v1](https://github.com/borchero/terraform-plan-comment)
- [trstringer/manual-approval@v1](https://github.com/trstringer/manual-approval)
