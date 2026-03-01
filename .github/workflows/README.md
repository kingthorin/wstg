# Workflows Documentation

This directory contains GitHub Actions workflows for the WSTG repository. Helper scripts used by these workflows are located in the `scripts/` subdirectory (with its own README).

## Version Information

These workflows use:
- Node.js and Python for various automation tasks
- GitHub Actions for checkout, setup, artifact management, and API interactions

## `build-checklists.yml`

For building checklists and Create a PR with changes made in the master.

- Trigger: Push, Only when files inside document directory is changed. Manual (`workflow_dispatch`), GitHub web UI.
- See: `/.github/xlsx/` in the root of the repository for XLSX build.

## `build-ebooks.yml`

For building PDF and EPUB e-Books at release.

- Trigger: Tag applied to repository. Manual (`workflow_dispatch`), GitHub web UI.
- See: `/.github/pdf/` in the root of the repository for PDF build specific configurations.
- See: `/.github/epub/` in the root of the repository for EPUB build specific configurations.

## `comment.yml`

Triggered by the completion of other workflows in order to comment lint or other results on PRs.
The workflows which leverage it should create a `pr_number` text file and `artifact.txt` with the content to be commented, which are attached to their workflow runs as `artifact`.

This workflow:
- Minimizes (collapses) previous comments from the same workflow run with appropriate classifiers:
  - `RESOLVED` when the workflow succeeds
  - `OUTDATED` when the workflow fails
- Only posts NEW comments on failure (not on success)
- Uses GitHub Actions for artifact retrieval and PR comment management

- Trigger: Other workflows `workflow_run`.

## Shared Pattern for Markdown Checks

The Markdown Link Check, Lint Check, and Terminology Lint Check workflows share one layout and reuse the **composite action** `actions/get-changed-files` so the “get changed files” logic is not duplicated. Each workflow has **one job** so the PR checks UI shows three checks (not six):

- **PR at workspace root** – The PR branch is checked out at the workspace root with `fetch-depth: 0`. That way the local composite action `./.github/actions/get-changed-files` is found (it is resolved from the root ref, i.e. the PR).
- **Base in `base/`** – The base branch is checked out into `base/`. All **config** (e.g. `base/.github/configs/`) and the **content** that gets checked come from base. Changed files from the PR are overlaid onto `base/`, then the tool runs on `base/` with base’s config.
- **Get changed files** – The composite action `actions/get-changed-files` runs at root (no `working_directory`). It outputs `files_updated` (.md only) and `all_changed`. Paths under `.github/` are excluded. The job then copies those files from root into `base/` and runs the checker on `base/` using config from `base/`.

**Trade-off:** The workflow YAML and the composite action are taken from the **PR** ref (so a PR can change them). Only the **config and base content** used for the actual checks are from the base branch. To have workflow and config both from base only, you’d need to inline the “get changed files” script in each workflow (no shared action).

### `actions/get-changed-files`

Composite action that computes the list of files changed between the base ref and the PR head. Used by all three Markdown workflows. Call it from the workspace root (PR checkout); optional input `working_directory` can be set if the action is ever used from a different layout.

- **Inputs:** `base_ref` (required), `working_directory` (optional).
- **Outputs:** `files_updated`, `all_changed`.

## `dummy.yml`

Utility action named "Markdown Lint Check" (same name as `md-lint-check.yml`) that serves as a fallback to satisfy branch protection requirements. This workflow only runs when NO Markdown files are changed in a PR (e.g., only an image or YAML that isn't linted). It's a complementary workflow to `md-lint-check.yml` that ensures the required "Markdown Lint Check" status check passes even when there are no Markdown files to lint.

- Trigger: Pull Requests (when no `.md` files are changed).

## `md-link-check.yml`

Checks Pull Requests for broken links.

This workflow:
- Has a single job (**link-check**). Checkout: PR at root, base in `base/`. Changed files from the composite action `actions/get-changed-files`; overlay root → `base/`; run link check on `base/` with `base/.github/configs/`.
- On `workflow_dispatch`, checks out the repo at root and runs the link checker over all Markdown.

- Trigger: Pull Requests (when `.md` files are changed, excluding `.github/**`). Manual (`workflow_dispatch`).
- Config File: `markdown-link-check-config.json`

## `md-lint-check.yml`

Checks Markdown files and flags style or syntax issues.

This workflow:
- Has a single job (**lint**). Checkout: PR at root, base in `base/`. Uses composite action; overlay root → `base/`; run markdownlint on `base/` with config and `format_lint_output.py` from `base/`.

- Trigger: Pull Requests (when `.md` files are changed, excluding `.github/**`).
- Config File: `.markdownlint.json`

## `md-textlint-check.yml`

Checks Markdown files for spelling style and typo issues.

This workflow:
- Has a single job (**textlint**). Checkout: PR at root, base in `base/`. Uses composite action; overlay root → `base/`; run textlint on `base/` with config from `base/`.

- Trigger: Pull Requests (when `.md` files are changed, excluding `.github/**`).
- Config File: `.textlintrc.json`

## `www_latest_update.yml`

Publishes the latest web content using the @wstgbot account to `OWASP/www-project-web-security-testing-guide`.

- Trigger: Push.
- See: `/.github/www/latest/` in the root of the repository.

## `www_stable_update.yml`

Publishes stable and versioned web content using the @wstgbot account to `OWASP/www-project-web-security-testing-guide`.

- Trigger: Tag applied to repository (format `v*`).
- See: `/.github/www/` in the root of the repository.
