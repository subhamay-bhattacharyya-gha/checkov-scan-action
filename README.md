![Release](https://github.com/subhamay-bhattacharyya-gha/checkov-scan-action/actions/workflows/release.yaml/badge.svg)&nbsp;![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/99d0bd3b1a9d6991fca653dc4c73dbdc/raw/checkov-scan-action.json?)

# 🚀 Checkov Scan

**Checkov Scan** is a GitHub Composite Action that runs a [Checkov](https://www.checkov.io/) scan for security and compliance issues on CloudFormation / Terraform / Kubernetes IaC code etc.

---

## 🔍 What It Does

1. **Assumes an AWS IAM Role** using GitHub OIDC.
2. **Runs Checkov** scan on supported IaC frameworks (CloudFormation, Terraform, Kubernetes, or all).

---

## 📦 Inputs

| Name            | Description                                                                 | Required | Default                                                             |
|-----------------|-----------------------------------------------------------------------------|----------|---------------------------------------------------------------------|
| `aws-role-arn`  | ARN of the IAM role to assume.                                              | ✅ Yes   | `arn:aws:iam::111122223333:role/github-oidc-role`                  |
| `aws-region`    | AWS region where resources are deployed.                                    | ✅ Yes   | `us-east-1`                                                         |
| `iac-dir`  | Directory path where the IaC templates are located.                           | ✅ Yes   | `cfn`                                                               |
| `iac-framework` | IaC framework for Checkov (`cloudformation`, `terraform`, `kubernetes`, `all`). | ✅ Yes   | `cloudformation`                                                    |
| `soft-fail`     | If `true`, Checkov scan failures will not fail the pipeline.                | ✅ Yes   | `true`                                                              |
| `github-token`  | GitHub token for authenticating the workflow.                              | ✅ Yes   |                                                                     |

---

## 📤 Outputs

| Name                | Description                              |
|---------------------|------------------------------------------|
| `valid-template`    | `true` if the template is valid          |
| `validation-error`  | Validation error message, if any         |

---

## 🛠 Usage Example

```yaml
jobs:
  validate-template:
    runs-on: ubuntu-latest
    steps:
      - name: Run Checkov Scan
        uses: subhamay-bhattacharyya-gha/checkov-scan-action@main
        with:
          aws-role-arn: arn:aws:iam::111122223333:role/github-oidc-role
          aws-region: us-east-1
          iac-dir: cfn
          iac-framework: cloudformation
          soft-fail: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## License

MIT
