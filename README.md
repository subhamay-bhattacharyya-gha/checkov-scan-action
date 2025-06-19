![Release](https://github.com/subhamay-bhattacharyya-gha/checkov-scan-action/actions/workflows/release.yaml/badge.svg)&nbsp;![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/issues/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya-gha/checkov-scan-action)&nbsp;![](https://img.shields.io/endpoint?url=https://gist.githubusercontent.com/bsubhamay/99d0bd3b1a9d6991fca653dc4c73dbdc/raw/checkov-scan-action.json?)

# 🚀 Checkov Scan

**Checkov Scan** is a reusable GitHub Composite Action that runs [Checkov](https://www.checkov.io/) scans for security and compliance issues across IaC frameworks like CloudFormation, Terraform, and Kubernetes.

---

## 🔍 What It Does

1. **Dynamically checks out a release tag** (if provided) or the current branch.
2. **Runs Checkov** against the specified directory and framework.
3. **Generates SARIF output** for GitHub code scanning (Security tab).

---

## 📦 Inputs

| Name             | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `release-tag`    | Git release tag to check out. If omitted, defaults to current ref.         |
| `iac-dir`        | Directory path where the IaC templates are located.                         |
| `iac-framework`  | IaC framework for Checkov (`cloudformation`, `terraform`, `kubernetes`).   |
| `soft-fail`      | If `true`, Checkov scan failures will not fail the pipeline.               |

---

## 🛠 Usage Example

```yaml
jobs:
  checkov:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Run Checkov Scan
        uses: subhamay-bhattacharyya-gha/checkov-scan-action@main
        with:
          release-tag: v1.0.0
          iac-dir: iac
          iac-framework: "cloudformation"
          soft-fail: true

```

## License

MIT
