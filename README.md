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
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/terraform-plan.yml
    with:
      env: "dev"
      terraform-version: "1.8.5"
      terraform-working-dir: "terraform"
      terraform-backend-config-file: "backend-dev.tfvars"
      terraform-vars-file: "tf-vars-dev.tfvars"
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

on:
  push:
    branches:
      - main

  deploy-to-dev:
    name: Deploy to Dev in AWS
    permissions:
      id-token: write
      issues: write
      contents: read
    uses: ./.github/workflows/terraform-plan-and-apply.yml
    with:
      env: "dev"
      terraform-version: "1.8.5"
      terraform-working-dir: "terraform"
      terraform-backend-config-file: "backend-dev.tfvars"
      terraform-vars-file: "tf-vars-dev.tfvars"
      terraform-plan-approvers: User1
      minimum-approvals: 1
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.DEV_AWS_ACCOUNT_ACCESS_ROLE }}


    deploy-to-prod:
    name: Deploy to Dev in AWS
    permissions:
      id-token: write
      issues: write
      contents: read
    uses: ./.github/workflows/terraform-plan-and-apply.yml
    with:
      env: "prod"
      terraform-version: "1.8.5"
      terraform-working-dir: "terraform"
      terraform-backend-config-file: "backend-dev.tfvars"
      terraform-vars-file: "tf-vars-dev.tfvars"
      terraform-plan-approvers: User1,User2
      minimum-approvals: 2
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.PROD_AWS_ACCOUNT_ACCESS_ROLE }}
```

</details>

## Acknowledegments

The workflows comprise of different GitHub Actions below. Please see these GitHub Action projects for further information:

- [actions/checkout](https://github.com/actions/checkout)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [hashicorp/setup-terraform@v3](https://github.com/hashicorp/setup-terraform)
- [borchero/terraform-plan-comment@v1](https://github.com/borchero/terraform-plan-comment)
- [trstringer/manual-approval@v1](https://github.com/trstringer/manual-approval)
