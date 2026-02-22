# Tabnine PR Agent

AI-powered code review for pull requests and merge requests using the Tabnine CLI Agent. Supports **GitHub Actions**, **GitLab CI**, and **Bitbucket Pipelines**.

---

# GitHub Actions Setup

## Prerequisites

Set the following repository secret in **Settings > Secrets and variables > Actions**:

| Secret / Variable | Required | Description |
|---|---|---|
| `TABNINE_KEY` | Yes | Tabnine Personal Access Token. Store as a **repository secret**. |

The action also requires a `github_token` input — typically provided via the built-in `secrets.GITHUB_TOKEN`.

## Usage

1. This action works only on the `pull_request` event within the workflow.

```yaml
on:
  pull_request:
    branches:
      - main

permissions:
  contents: read
  pull-requests: write
```

2. Ensure that the repository is [checked out](https://github.com/actions/checkout/tree/v4#readme) with full history (`fetch-depth: 0`) within your workflow.

3. Add the following step to your workflow:

```yaml
- name: Review PR
  uses: codota/tabnine-pr-agent@v2
  continue-on-error: true
  with:
    # Tabnine authentication token — required
    TABNINE_KEY: ${{ secrets.TABNINE_KEY }}

    # GitHub token for authentication — required
    github_token: ${{ secrets.GITHUB_TOKEN }}

    # Repository in owner/repo format — required
    repository: ${{ github.repository }}

    # Pull request number — required
    pull_request_number: ${{ github.event.pull_request.number }}

    # PR head commit SHA — required
    head_sha: ${{ github.event.pull_request.head.sha }}

    # PR base commit SHA — required
    base_sha: ${{ github.event.pull_request.base.sha }}

    # Tabnine host URL (optional, default: https://console.tabnine.com)
    # tabnine_host: "https://console.tabnine.com"

    # Model ID for the Tabnine CLI agent (optional, overrides DEFAULT_MODEL_ID in action.yml)
    # model_id: "your-model-id"
```

### Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `TABNINE_KEY` | Yes | — | Tabnine Personal Access Token |
| `github_token` | Yes | — | GitHub token for authentication (typically `secrets.GITHUB_TOKEN`) |
| `repository` | Yes | — | Repository in `owner/repo` format |
| `pull_request_number` | Yes | — | Pull request number |
| `head_sha` | Yes | — | PR head commit SHA |
| `base_sha` | Yes | — | PR base commit SHA |
| `tabnine_host` | No | `https://console.tabnine.com` | Tabnine host URL (for self-hosted / EMT installations) |
| `model_id` | No | — | Model ID for the Tabnine CLI agent. If omitted, falls back to `DEFAULT_MODEL_ID` in `action.yml` or the system default from the admin console. |

## Full Workflow Example

```yaml
name: "Tabnine PR Review Agent"

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read
  pull-requests: write

jobs:
  review_pr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Review PR
        uses: codota/tabnine-pr-agent@v2
        continue-on-error: true
        with:
          TABNINE_KEY: ${{ secrets.TABNINE_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          pull_request_number: ${{ github.event.pull_request.number }}
          head_sha: ${{ github.event.pull_request.head.sha }}
          base_sha: ${{ github.event.pull_request.base.sha }}
```

---

# GitLab CI Setup

This repository includes a `.gitlab-ci.yml` configuration in the `GitLab/` directory for running the Tabnine PR Agent on GitLab merge requests.

## Prerequisites

Set the following CI/CD variables in **Settings > CI/CD > Variables**:

| Variable | Required | Description |
|---|---|---|
| `TABNINE_KEY` | Yes | Tabnine Personal Access Token. Mark as **Masked** and **Protected**. |
| `GITLAB_API_TOKEN` | Yes | GitLab personal or project access token with `api` scope. Mark as **Masked**. |
| `TABNINE_HOST` | No | Tabnine host URL for self-hosted / EMT installations (default: `https://console.tabnine.com`) |
| `TABNINE_MODEL_ID` | No | Model ID for the Tabnine CLI agent. If empty, uses the system default from the admin console. |

## Usage

Copy the `.gitlab-ci.yml` file from the `GitLab/` directory to the root of your GitLab repository. The pipeline automatically triggers on merge request events.

If you already have a `.gitlab-ci.yml`, merge the `stages` and `tabnine-code-review` job into your existing configuration.

```yaml
# .gitlab-ci.yml (key sections)
stages:
  - review

tabnine-code-review:
  stage: review
  image: node:20
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
  # ... (see GitLab/.gitlab-ci.yml for full configuration)
  allow_failure: true
```

---

# Bitbucket Pipelines Setup

This repository includes a `bitbucket-pipelines.yml` configuration in the `Bitbucket/` directory for running the Tabnine PR Agent on Bitbucket pull requests.

## Prerequisites

Set the following repository variables in **Repository Settings > Pipelines > Repository variables**:

| Variable | Required | Description |
|---|---|---|
| `TABNINE_KEY` | Yes | Tabnine Personal Access Token. Mark as **Secured**. |
| `BB_API_TOKEN` | Yes | Bitbucket token with `pullrequest:write` and `repository:read` scopes. Mark as **Secured**. |
| `TABNINE_HOST` | No | Tabnine host URL for self-hosted / EMT installations (default: `https://console.tabnine.com`) |
| `TABNINE_MODEL_ID` | No | Model ID for the Tabnine CLI agent. If empty, falls back to `DEFAULT_MODEL_ID` in the pipeline yml or the system default from the admin console. |

## Usage

Copy the `bitbucket-pipelines.yml` file from the `Bitbucket/` directory to the root of your Bitbucket repository. The pipeline automatically triggers on all pull requests.

If you already have a `bitbucket-pipelines.yml`, merge the `pull-requests` section into your existing configuration.

```yaml
# bitbucket-pipelines.yml (key sections)
image: node:20

pipelines:
  pull-requests:
    '**':
      - step:
          name: Tabnine Code Review
          script:
            # ... (see Bitbucket/bitbucket-pipelines.yml for full configuration)
```
