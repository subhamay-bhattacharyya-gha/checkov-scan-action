# 🚀 Checkov Scan

![Release](https://github.com/subhamay-bhattacharyya-gha/checkov-scan-action/actions/workflows/release.yaml/badge.svg)&nbsp;![Commit Activity](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Last Commit](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Release Date](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Repo Size](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![File Count](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Issues](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Top Language](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Monthly Commits](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![Custom Badge](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/99d0bd3b1a9d6991fca653dc4c73dbdc/raw/checkov-scan-action.json?)

## Overview

**Checkov Scan** is a reusable GitHub Composite Action that runs [Checkov](https://www.checkov.io/) scans for security and compliance issues across IaC frameworks like CloudFormation, Terraform, and Kubernetes.

---

## 🔍 What It Does

1. **Pulls the enabled check-ID catalog** from a central gist and renders it to the job summary.
2. **Runs Checkov** against the specified directory (or plan JSON file) and framework.
3. **Generates SARIF output** and uploads it to GitHub Code Scanning (Security tab) via `github/codeql-action/upload-sarif@v4`.

> **Caller responsibility:** this action does **not** run `actions/checkout`. Your workflow must check out the repo (and download any plan-file artifact) **before** invoking this action. An inner checkout would `git clean -ffdx` the workspace and delete downloaded artifacts.

---

## 📦 Inputs

| Name               | Required | Default             | Description                                                                                                                                                                                                                             |
| ------------------ | -------- | ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `release-tag`      | No       | —                   | **Deprecated, ignored.** Retained so callers don't hit "Unexpected input(s)"; remove from your workflow — caller now owns `actions/checkout`.                                                                                           |
| `iac-dir`          | Yes      | `iac`               | Directory containing the IaC templates. For directory scans, this is what gets scanned. For `plan-file` scans, this is also used as the source dir for plan enrichment (source file paths / line numbers / code snippets in the SARIF). |
| `iac-framework`    | Yes      | `terraform`         | IaC framework for Checkov (`cloudformation`, `terraform`, `kubernetes`, etc.). Ignored when `plan-file` is set.                                                                                                                         |
| `plan-file`        | No       | —                   | Path (relative to the workspace) to a Terraform plan JSON. If set, Checkov scans this file using framework `terraform_plan`.                                                                                                            |
| `output-file-path` | No       | `/github/workspace` | Directory where Checkov writes `results.sarif`. Must be accessible from inside the Checkov container.                                                                                                                                   |
| `soft-fail`        | No       | `true`              | If `true`, Checkov scan failures will not fail the pipeline.                                                                                                                                                                            |
| `github-token`     | Yes      | —                   | GitHub token used for API calls. Pass `secrets.GITHUB_TOKEN` from the caller workflow.                                                                                                                                                  |

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
      - uses: actions/checkout@v6

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
          # iac-dir still matters here: it's used as the plan-enrichment
          # source dir so SARIF findings carry .tf file paths, line numbers,
          # and code snippets instead of pointing at tfplan.json.
          iac-dir: infra/platform/tf
          soft-fail: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

When `plan-file` is provided, Checkov runs with `framework=terraform_plan`.
`iac-framework` is ignored, but **`iac-dir` should point at the original
Terraform source directory** so Checkov can enrich findings with real file
paths, line numbers, and code snippets via `--repo-root-for-plan-enrichment`.
Without this, summary panels that render the "Code Snippet" field will be
empty because findings are attributed to the single-line `tfplan.json`.

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

| Step                            | Description                                                                          |
|---------------------------------|--------------------------------------------------------------------------------------|
| Debug: Print Action Inputs      | Prints resolved inputs; dumps the plan JSON when `plan-file` is set.                 |
| Clean SARIF Results             | Removes any pre-existing `results.sarif` to ensure a clean scan.                     |
| Retrieve Gist Content           | Pulls the enabled-check-ID catalog from a central gist.                              |
| Print Enabled Checkov Scan IDs  | Renders the enabled checks to the job summary and builds the `--check` argument.     |
| Run Checkov Scan                | Executes Checkov against the IaC directory, or the plan JSON, with SARIF output.     |
| Ensure SARIF file exists        | Writes a minimal empty-results SARIF if Checkov did not produce one.                 |
| Locate SARIF file               | Probes common output locations and falls back to a workspace-wide `find`.            |
| Upload SARIF file               | Uploads `results.sarif` to GitHub Code Scanning.                                     |

---

## License

MIT
