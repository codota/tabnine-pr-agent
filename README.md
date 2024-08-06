# Tabnine PR Agent V1

This action analyzes a pull request using the TabNine CLI. It pulls the necessary Docker image, runs an analysis on the pull request, and checks the report in watch mode. This action helps in automating the code review process by leveraging TabNine's AI-powered code analysis capabilities.

# Usage
1. This action works only on the "pull_request" event within the workflow.
```yaml
on:
  pull_request:
    branches:
      - main
```

2. Ensure that repository is [checked out](https://github.com/actions/checkout/tree/v4#readme) within your workflow
3. Add the following to your workflow
```yaml
uses: codota/tabnine-pr-agent@v1
continue-on-error: true
with:
  # Personal Access Token (PAT) issued by the admin to access TabNine's analyzing capabilities
  PAT: <INSERT_YOUR_PAT_HERE>

  # The source branch of the pull request
  ref: origin/${{ github.head_ref }}

  # Path to the repository in the workflow run, typically equal to "github.workspace"
  path_to_repo: ${{ github.workspace }}

  # GitHub username or organization name that owns the repository
  repository_owner: ${{ github.repository_owner }}

  # The name of the GitHub repository
  repository_name: ${{ github.event.repository.name }}
  
  # The ID of the pull request
  pull_request_id: ${{ github.event.pull_request.number }}

  # GitHub token for authentication, provided by GitHub when the workflow is run
  github_token: ${{ secrets.GITHUB_TOKEN }}
```

# Examples
Full workflow that is triggered on "pull_request":

```yaml
name: "Review PR"

on:
  pull_request:
    branches:
      - main

jobs:
  review_pr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Analyse PR
        uses: codota/tabnine-pr-agent@main
        continue-on-error: true
        with:
          PAT: ${{ secrets.COACHING_PAT }}
          ref: origin/${{ github.head_ref }}
          path_to_repo: ${{ github.workspace }}
          repository_owner: ${{ github.repository_owner }}
          repository_name: ${{ github.event.repository.name }}
          pull_request_id: ${{ github.event.pull_request.number }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Special case: inviting someone outside Tabnine organisation
To invite an external user, you'll need to provide them with a couple of essential items:
1. A specially created "Deploy key", which is a read-only SSH key. This can be individually created for each invitee or for a group of people, depending on your preference.
2. An advanced workflow configuration, which you can directly copy and paste from below:

```yaml
name: "Review PR"

on:
  pull_request:
    branches:
      - main

jobs:
  review_pr:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout the repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Checkout the action repository
        uses: actions/checkout@v4
        with:
          repository: codota/tabnine-pr-agent
          ref: v1
          path: ./.github/actions/tabnine-pr-agent
          ssh-key: ${{ secrets.ACTION_ACCESS_PRIVATE_SSH_KEY }}

      - name: Analyse PR
        uses: ./.github/actions/tabnine-pr-agent
        continue-on-error: true
        with:
          PAT: ${{ secrets.COACHING_PAT }}
          ref: origin/${{ github.head_ref }}
          path_to_repo: ${{ github.workspace }}
          repository_owner: ${{ github.repository_owner }}
          repository_name: ${{ github.event.repository.name }}
          pull_request_id: ${{ github.event.pull_request.number }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

Note: You'll need to configure a few `secrets` in your repository settings:
* COACHING_PAT (which should be obtained from the tabnine CLI)
```shell
tabnine auth status
```
* ACTION_ACCESS_PRIVATE_SSH_KEY (which should be obtained from Tabnine)