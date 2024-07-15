# GitHub Actions Reusuable Workflows

This repo consolidates reusable workflows for GitHub Actions. Example uses are available in these projects:

[fruit-project-infra](https://github.com/KremzeeqOrg/fruit-project-infra)
[fruit-project-api-scraper](https://github.com/KremzeeqOrg/fruit-project-api-scraper)

## Contents

- [Terraform Plan](#terraform-plan)
- [Terraform Plan and Apply](#terraform-plan-and-apply)
- [Serverless Deploy Workflow](#serverless-deploy-workflow)
  - [Tagging Strategy Summary for Docker images](#tagging-strategy-summary-for-docker-images)
- [Pytest Unit Test](#pytest-unit-test)

## Terraform Plan

<details>

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
      aws-iam-role: ${{ secrets.AWS_ACCOUNT_ACCESS_ROLE }}

```

</details>
</details>

## Terraform Plan and Apply

<details>

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
      aws-iam-role: ${{ secrets.AWS_GITHUB_ACTIONS_ACCESS_ROLE }}
      tf-plan-approvers: ${{ secrets.TF_PLAN_APPROVERS }}

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
      aws-iam-role: ${{ secrets.AWS_GITHUB_ACTIONS_ACCESS_ROLE}}
      tf-plan-approvers: ${{ secrets.TF_PLAN_APPROVERS }}
```

</details>
</details>

## Serverless Deploy Workflow

<details>

- [workflow](.github/workflows/serverless-deploy-workflow.yml)
- Based on the environment (`feature`/`dev`/`prod`), the workflow implements a tagging strategy for Docker images, which are built and pushed to AWS ECR. `stage`. This results in a AWS Cloudformation stack being provisioned with resources as specified in the `serverless.yml` file, speciifed in the repo that calls the reusable workflow.
- If you use this workflow, you will need to provision AWS ECR and Docker Hub repos with the same name as the `APP` e.g. `fruit-project-api-scraper`.

### Tagging Strategy Summary for Docker images

<details>

#### feature environment

- Build Docker image with the tag `APP:feature-COMMIT_SHA_SHORT`.
- Push to AWS ECR with the tag `AWS_ECR_REPO/APP:feature-COMMIT_SHA_SHORT`.

#### dev environment

- Build Docker image with the tag `APP:dev-COMMIT_SHA_SHORT`.
- Tag image as `AWS_ECR_REPO/APP:dev-COMMIT_SHA_SHORT` and `AWS_ECR_REPO/APP:latest`.
- Tag image as `DOCKERHUB_USERNAME/APP:dev-COMMIT_SHA_SHORT` and `DOCKERHUB_USERNAME/APP:latest`.
- Push the image to AWS ECR and Docker Hub with all above tags.

#### prod envionment

- Pull the image from Docker Hub with the tag `DOCKERHUB_USERNAME/APP:dev-COMMIT_SHA_SHORT`.
- Tag image as `AWS_ECR_REPO/APP:prod-COMMIT_SHA_SHORT` and `AWS_ECR_REPO/APP:latest`.
- Push the image to AWS ECR with both tags.

</details>

### Typical Workflow Use Cases

The resusable workflow can support a straightforward deployment to an environment.

<details>

<summary>Example GitHub Action Workflow Input Details</summary>

```
  serverless-deploy:
    name: Serverless Deploy
    permissions:
      id-token: write
      contents: read
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/serverless-deploy-workflow.yml
    with:
      environment: dev
    secrets:
      aws-region: ${{ secrets.AWS_REGION }}
      aws-iam-role: ${{ secrets.AWS_ACCOUNT_ACCESS_ROLE }}
      aws-account-id: ${{ secrets.AWS_ACCOUNT_ID }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
      serverless-access-key: ${{ secrets.SERVERLESS_ACCESS_KEY }}
```

It's also possible to use this to provision to an ephemeral (short-lived) environment, based on whether your pr as a `deploy` label. A teardown workflow can support removing the related cloudformation stack, once you have merged your pr.

</details>

<details>

<summary>Example of feature deploy related workflows</summary>

- [serverless feature deploy workflow](https://github.com/KremzeeqOrg/fruit-project-api-scraper/blob/main/.github/workflows/serverless-feature-workflow.yml)
- [serverless teardown workflow](https://github.com/KremzeeqOrg/fruit-project-api-scraper/blob/main/.github/workflows/serverless-feature-teardown.yml)

</details>

</details>
</details>

## Pytest Unit Test

<details>

- [workflow](.github/workflows/pytest-unit-test-workflow.yml)

This workflow can be used to run Pytest unit tests in relation to Python code.

### Typical Workflow Use Case

The Pytest reusable workflow can be used with a matrix, so that Pytests can run concurrently for each directory where tests are specified.

<details>

<summary>Example GitHub Action Workflow Input Details using matrix</summary>

```
jobs:
  pytest:
    name: Pytest
    permissions:
      contents: read
    strategy:
      matrix:
        test_file: ${{ fromJson('["api_mapping_manager", "record_manager", "scraper", "utils"]') }}
    uses: KremzeeqOrg/gha-reusable-workflows/.github/workflows/pytest-unit-test-workflow.yml@main
    with:
      environment: dev
      test_file: ${{ matrix.test_file }}
```

</details>
</details>

## Environment Variables

GitHub environments can be set at the repo or organization level, where environment variables and secrets can be specified. Please see [here](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment) for more info.

<details>

<summary>Variables for Terraform workflows</summary>

In relation to your GitHub environment e.g. for `dev`/ `prod`, manually set the following variables:

| Variable               | Explanation                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------- |
| ENV                    | e.g. `dev` / `prod`                                                                                 |
| MINIMUM_APPROVALS\*    | Number of GitHub approvals needed for `terraform plan`                                              |
| TF_BACKEND_CONFIG_FILE | Backend configuration file for Terraform, required for `terraform init`, e.g. `backend-dev.tfvars`  |
| TF_VARS_FILE           | Configuration file for environment variables for Terraform. e.g. `tf-vars-dev.tfvars`               |
| TF_VERSION             | Terraform version e.g. run `terraform --version` to check what you're using locally e.g. 1.8.5      |
| TF_WORKING_DIR         | Directory where all terraform files are kept in relation to the root for your repo e.g. `terraform` |

- \* only required for Terraform Plan and Approve workflow

Ensure secrets are set for the following in your workflow. Check the example usage on this page.

| Variable            | Explanation                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| aws-region          | Target AWS region e.g. `eu-west-2`                                                                                                                                                                                                                                                                                                                                                            |
| aws-iam-role        | This is the arn for a AWS IAM role with a trust policy, which enables GitHub as a OIDC provider to assume the role with certain permissions. A policy should also be attached to the role, applying the pinciple of 'least privilige'. Please consult this [AWS blog](https://aws.amazon.com/blogs/security/use-iam-roles-to-connect-github-actions-to-actions-in-aws/) for further guidance. |
| tf-plan-approvers\* | Specify GitHub user e.g. `user` or users e.g. `user1,user2`, who can approve Terraform Plan                                                                                                                                                                                                                                                                                                   |

- \* only required for Terraform Plan and Approve workflow

</details>

<details>

<summary>Variables for Serverless Deploy workflow</summary>

In relation to your GitHub environments e.g. for `feature`, `dev` and `prod`, manually set the following variables:

| Variable           | Explanation                                                       |
| ------------------ | ----------------------------------------------------------------- |
| APP                | App name. AWS ECR and Docker Hub repos should have the same name. |
| ENV                | `feature` / `dev` / `prod`                                        |
| NODE_VERSION       | Node version. e.g. 20                                             |
| SERVERLESS_VERSION | Serverless framework version e.g. > 3.38.0                        |

Ensure secrets are set for the following in your workflow. Check the example usage on this page.

| Variable              | Explanation                                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| aws-region            | Target AWS region e.g. `eu-west-2`                                                                                                                                                                                                                                                                                                                                                            |
| aws-iam-role          | This is the arn for a AWS IAM role with a trust policy, which enables GitHub as a OIDC provider to assume the role with certain permissions. A policy should also be attached to the role, applying the pinciple of 'least privilige'. Please consult this [AWS blog](https://aws.amazon.com/blogs/security/use-iam-roles-to-connect-github-actions-to-actions-in-aws/) for further guidance. |
| dockerhub-username    | Username for Docker Hub                                                                                                                                                                                                                                                                                                                                                                       |
| dockerhub-token       | Docker Hub token                                                                                                                                                                                                                                                                                                                                                                              |
| serverless-access-key | Serverless Framework access key                                                                                                                                                                                                                                                                                                                                                               |

</details>

<details>

<summary>Variables for Pytest Workflow</summary>

| Variable        | Explanation                                                                                                                                                                        |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PYTHON_VERSION  | e.g. 3.10                                                                                                                                                                          |
| PYTEST_TEST_DIR | e.g. `src/tests`                                                                                                                                                                   |
| test_file       | This variable has to be directly specified in the workflow which calls the reusable workflow e.g. ${{ fromJson('["api_mapping_manager", "record_manager", "scraper", "utils"]') }} |

</details>

## Acknowledgements

The workflows comprise of different GitHub Actions below. Please see these GitHub Action projects for further information:

- [actions/checkout](https://github.com/actions/checkout)
- [aws-actions/configure-aws-credentials](https://github.com/aws-actions/configure-aws-credentials)
- [hashicorp/setup-terraform@v3](https://github.com/hashicorp/setup-terraform)
- [borchero/terraform-plan-comment@v1](https://github.com/borchero/terraform-plan-comment)
- [trstringer/manual-approval@v1](https://github.com/trstringer/manual-approval)
