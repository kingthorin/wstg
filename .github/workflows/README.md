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

## Shared Pattern for Markdown Linting

The Markdown Link Check, Lint Check, and Terminology Lint Check workflows all use the same layout and reuse the same “changed files” logic:

- **Base at workspace root** – The base branch is checked out at the workspace root so that workflow files and config (e.g. `.github/configs/`) come from the base branch, not the PR. This keeps workflow and config under maintainer control.
- **PR in `pr/`** – The PR branch is checked out into the `pr/` directory.
- **Reusable workflow** – Changed files are computed by the reusable workflow `reusable_changed_files.yml`, which is invoked at **job** level (reusable workflows cannot be invoked from a step). The lint/link-check job then overlays changed files from `pr/` onto the root and runs the tool only on those files, using config from the base (root).

### `reusable_changed_files.yml`

Reusable workflow (invoked with `workflow_call`) that computes the list of files changed between the base ref and the PR head.

- **Inputs:** `base_ref` – branch or ref to compare against (e.g. `master`).
- **Outputs:** `files_updated` (space-separated changed `.md` files), `all_changed` (space-separated all changed files). Paths under `.github/` are excluded.
- Used by: `md-link-check.yml`, `md-lint-check.yml`, `md-textlint-check.yml`.

## `dummy.yml`

Utility action named "Markdown Lint Check" (same name as `md-lint-check.yml`) that serves as a fallback to satisfy branch protection requirements. This workflow only runs when NO Markdown files are changed in a PR (e.g., only an image or YAML that isn't linted). It's a complementary workflow to `md-lint-check.yml` that ensures the required "Markdown Lint Check" status check passes even when there are no Markdown files to lint.

- Trigger: Pull Requests (when no `.md` files are changed).

## `md-link-check.yml`

Checks Pull Requests for broken links.

This workflow:
- Runs a **get-changed-files** job that calls `reusable_changed_files.yml` to get changed files (job-level reusable workflow).
- Checks out the **base branch** at the workspace root and the **PR head** into `pr/`.
- Overlays all changed files from `pr/` onto the root (base). Link check runs only on changed `.md` files so relative links resolve; config is always from the base (root).
- On `workflow_dispatch`, runs the link checker over all Markdown in the repo (excluding `pr/`).

- Trigger: Pull Requests (when `.md` files are changed, excluding `.github/**`). Manual (`workflow_dispatch`).
- Config File: `markdown-link-check-config.json`

## `md-lint-check.yml`

Checks Markdown files and flags style or syntax issues.

This workflow:
- Runs a **get-changed-files** job that calls `reusable_changed_files.yml`.
- Checks out the **base branch** at the workspace root and the **PR head** into `pr/`.
- Overlays changed files from `pr/` onto the root and runs `markdownlint-cli2` only on those files, using config and `format_lint_output.py` from the base (root).
- Uploads artifacts for both success and failure to work with `comment.yml`.

- Trigger: Pull Requests (when `.md` files are changed, excluding `.github/**`).
- Config File: `.markdownlint.json`

## `md-textlint-check.yml`

Checks Markdown files for spelling style and typo issues.

This workflow:
- Runs a **get-changed-files** job that calls `reusable_changed_files.yml`.
- Checks out the **base branch** at the workspace root and the **PR head** into `pr/`.
- Overlays changed files from `pr/` onto the root and runs textlint only on those files, using config from the base (root).

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
