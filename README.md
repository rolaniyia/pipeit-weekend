# PipeIt Weekend

A small GitHub Actions demo project that validates a repository README and then builds and deploys a custom CI status page to GitHub Pages.

This repository is designed to showcase a simple pipeline that:

- checks that `README.md` includes a Markdown heading
- creates a custom composite GitHub Action
- builds a status card page with build metadata
- publishes the result to GitHub Pages

## Overview

The project uses GitHub Actions to automate a lightweight release workflow:

1. A `test` job verifies the repository is in the expected state.
2. A `build` job runs a custom composite action that writes a status page into the `site/` directory.
3. A `deploy` job publishes the generated site to GitHub Pages.

The workflow is defined in `.github/workflows/pipeline.yml`, and the reusable build logic lives in `.github/actions/build-card/action.yml`.

## Repository structure

```text
.
├── .github/
│   ├── actions/
│   │   └── build-card/
│   │       └── action.yml
│   └── workflows/
│       └── pipeline.yml
├── tests/
│   └── check.sh
├── README.md
└── site/   # generated during the build workflow
```

## What the project checks

The validation script in `tests/check.sh` ensures that `README.md` has a title line beginning with `# `.

This is intentionally simple: it proves that the repository can run a check before proceeding to build and deployment.

The script exits successfully only if the README contains a heading; otherwise it prints an error and fails the workflow.

## Custom GitHub Action

The custom action at `.github/actions/build-card/action.yml` defines a composite action with:

- one required input: `name`
- one optional input: `message` with a default value
- one output: `page`

During execution, it:

- creates a `site/` directory
- generates `site/index.html`
- includes metadata such as:
  - run number
  - commit short SHA
  - UTC build timestamp
  - GitHub event name
- writes a summary entry to `GITHUB_STEP_SUMMARY`

The page content is a simple HTML card styled in a clean, modern layout and intended to communicate that all checks passed.

## Workflow behavior

The CI pipeline is triggered by:

- pushes to `main`
- a scheduled cron job
- manual `workflow_dispatch` runs

The workflow runs three jobs:

### 1. test

Runs on `ubuntu-latest` and executes:

```bash
bash tests/check.sh
```

If the README heading check fails, the job stops and prevents the build and deploy steps from running.

### 2. build

This depends on `test` and runs the custom composite action:

```yaml
- uses: ./.github/actions/build-card
  with:
    name: Raphael Olaniyi
    message: Built automatically by my own GitHub Action
```

After the action writes the page, the workflow packages the generated `site` directory using `actions/upload-pages-artifact@v5`.

### 3. deploy

This depends on `build` and uses `actions/deploy-pages@v5` to publish the generated website to GitHub Pages.

## Local validation

You can run the same validation locally:

```bash
bash tests/check.sh
```

If your `README.md` starts with a heading like:

```md
# PipeIt Weekend
```

then the check passes.

## GitHub Pages deployment

This repository is configured to deploy a static site to GitHub Pages using the Pages deployment actions. The deployment job is connected to the `github-pages` environment and uses the generated artifact from the build step.

## Why this repository is useful

This project is a practical example of:

- GitHub Actions fundamentals
- custom reusable actions
- composite action authoring
- validation checks before deployment
- static site publishing via GitHub Pages

It is a small but complete demonstration of automating a simple CI/CD pipeline in a repository.

## Notes

The repository currently focuses on simple automation rather than a broad application codebase. The main goal is to demonstrate how workflow steps, custom actions, tests, and deployment can work together in a single GitHub-hosted project.
