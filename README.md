# 🚀 Checkov Scan

![Release](https://github.com/subhamay-bhattacharyya-gha/checkov-scan-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Monthly Commits](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Custom Badge](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/99d0bd3b1a9d6991fca653dc4c73dbdc/raw/checkov-scan-action.json?)

## Overview

**Checkov Scan** is a reusable GitHub Composite Action that runs [Checkov](https://www.checkov.io/) scans for security and compliance issues across IaC frameworks like CloudFormation, Terraform, and Kubernetes.

---

## 🔍 What It Does

1. **Validates the checkout ref** — verifies the provided release tag exists in the repo; falls back to the current branch if not found.
2. **Checks out the repository** at the validated ref.
3. **Runs Terraform Checkov Scan Detector** to identify Terraform-specific scan targets (optional, non-blocking).
4. **Runs Checkov** against the specified directory and framework.
5. **Generates SARIF output** and uploads it to GitHub Code Scanning (Security tab) via `github/codeql-action/upload-sarif@v4`.

---

## 📦 Inputs

| Name               | Required | Default            | Description                                                                                                                         |
|--------------------|----------|--------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `release-tag`      | No       | —                  | Git release tag to check out. If omitted or not found, defaults to current branch ref.                                              |
| `iac-dir`          | Yes      | `iac`              | Directory path where the IaC templates are located. Ignored when `plan-file` is set.                                                |
| `iac-framework`    | Yes      | `terraform`        | IaC framework for Checkov (`cloudformation`, `terraform`, `kubernetes`, etc.). Ignored when `plan-file` is set.                     |
| `plan-file`        | No       | —                  | Path (relative to the workspace) to a Terraform plan JSON. If set, Checkov scans this file using framework `terraform_plan`.        |
| `output-file-path` | No       | `/github/workspace`| Directory where Checkov writes `results.sarif`. Must be accessible from inside the Checkov container.                               |
| `soft-fail`        | No       | `true`             | If `true`, Checkov scan failures will not fail the pipeline.                                                                        |
| `github-token`     | Yes      | —                  | GitHub token used for API calls. Pass `secrets.GITHUB_TOKEN` from the caller workflow.                                              |

---

## 🔐 Required Permissions

The caller workflow must grant the following permissions for SARIF upload to work:

```yaml
permissions:
  contents: read
  security-events: write
```

---

## 🛠 Usage Examples

### 1. Scan an IaC directory (Terraform / CloudFormation / Kubernetes)

```yaml
jobs:
  checkov:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - name: Run Checkov Scan
        uses: subhamay-bhattacharyya-gha/checkov-scan-action@main
        with:
          iac-dir: infra/platform/tf
          iac-framework: terraform
          soft-fail: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 2. Scan a Terraform plan JSON file

Scanning the plan output is more accurate than a directory scan because Checkov
only reports on resources that actually appear in the plan (respecting
`count`, `for_each`, variable values, and unchanged resources).

```yaml
jobs:
  checkov:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      security-events: write
    steps:
      - uses: actions/checkout@v6

      - uses: hashicorp/setup-terraform@v3

      - name: Generate plan JSON
        working-directory: infra/platform/tf
        run: |
          terraform init -input=false
          terraform plan -out=tfplan.binary -input=false
          terraform show -json tfplan.binary > tfplan.json

      - name: Run Checkov Scan against plan
        uses: subhamay-bhattacharyya-gha/checkov-scan-action@main
        with:
          plan-file: infra/platform/tf/tfplan.json
          soft-fail: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

When `plan-file` is provided, `iac-dir` and `iac-framework` are ignored and
Checkov runs with `framework=terraform_plan`.

### 3. Custom SARIF output location

```yaml
- uses: subhamay-bhattacharyya-gha/checkov-scan-action@main
  with:
    iac-dir: infra/platform/tf
    iac-framework: terraform
    output-file-path: /github/workspace/reports
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

---

## 📋 Action Workflow Steps

| Step                            | Description                                                                        |
|---------------------------------|------------------------------------------------------------------------------------|
| Set Checkout Ref                | Validates the `release-tag` ref exists; falls back to current branch if not found. |
| Checkout Repo                   | Checks out the repository at the resolved ref.                                     |
| Clean SARIF Results             | Removes any pre-existing `results.sarif` to ensure a clean scan.                   |
| Terraform Checkov Scan Detector | Runs an optional pre-scan detector (non-blocking).                                 |
| Run Checkov Scan                | Executes Checkov against the specified IaC directory with SARIF output.            |
| Upload SARIF file               | Uploads `results.sarif` to GitHub Code Scanning (only if the file was generated).  |

---

## License

MIT
