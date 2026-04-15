# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repo publishes a **GitHub Composite Action** (`action.yaml`) that runs a [Checkov](https://www.checkov.io/) IaC scan and uploads the resulting SARIF to GitHub Code Scanning. There is no application source code — the "product" is `action.yaml` plus the semantic-release tooling that versions it.

## Commands

- `npm ci` — install release tooling (only needed when working on the release pipeline).
- `npx semantic-release` — normally runs in CI on push to `main`; don't run locally unless debugging.
- There is no build, lint, or test suite. Validate changes by running the action in a consumer workflow.

## Architecture

### `action.yaml` — the composite action

All behavior lives in this single file. Important pipeline quirks worth knowing before editing:

1. **Ref resolution (`Set Checkout Ref`)** — if `release-tag` is passed but doesn't exist as a tag/branch on the remote, falls back to `GITHUB_REF_NAME`. Callers rely on this non-failing fallback.
2. **Enabled check IDs come from an external gist** (`gist.githubusercontent.com/bsubhamay/33218d3610939502b8fe69ee4858c575/...enabled-checkov-scans-ids.json`). The gist is parsed with `jq` into a `cloud → service → [CKV_IDs]` map, rendered into `$GITHUB_STEP_SUMMARY`, and flattened into the comma-separated `check` input for `bridgecrewio/checkov-action`. Changing enabled checks = edit the gist, not this repo.
3. **SARIF file location is fragile.** `bridgecrewio/checkov-action` runs inside a container and may write `results.sarif` to `/github/workspace`, the workspace root, or the IaC directory. The `Locate SARIF file` step probes all three and falls back to `find`. If Checkov produces nothing, `Ensure SARIF file exists` writes a minimal empty-results SARIF so the upload step doesn't fail the pipeline. Preserve both safety nets when refactoring.
4. **Upload uses `if: (success() || failure())`** so SARIF is uploaded even when the scan reports findings — this is intentional and pairs with `soft-fail: true` default.

### Release pipeline

- `.github/workflows/release.yaml` runs `semantic-release` on pushes to `main`.
- Config is indirected: `package.json` → `./scripts/plugins/release.config.js`. The `scripts/plugins/*.js` files (analyze-commits, generate-notes, prepare, publish, verify-conditions) exist as custom plugin scaffolding but `release.config.js` currently uses only the standard `@semantic-release/*` plugins — the custom files are not wired in.
- Commit messages must follow Conventional Commits (commitizen + `cz-conventional-changelog`) for version bumping to work.

## Consumer contract

Callers must grant `contents: read` and `security-events: write` and pass `secrets.GITHUB_TOKEN` as `github-token`. See README for the canonical usage example.
