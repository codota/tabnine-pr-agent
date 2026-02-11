# Tabnine PR Agent

AI-powered code review using Tabnine CLI Agent. This GitHub Action performs comprehensive code reviews on pull requests, providing inline comments and summary feedback based on engineering best practices.

## Features

- 🔍 **Automated Code Review**: Analyzes PRs for bugs, security issues, and production readiness
- 🎯 **Risk-Based Triage**: Classifies PRs by risk level (Low/Standard/High) and adjusts review depth accordingly
- 🔐 **Security Analysis**: Checks for SQL injection, auth issues, secrets exposure, and more
- 🚀 **Performance Review**: Identifies N+1 queries, algorithmic complexity issues, and scalability concerns
- 🌐 **Cross-Repository Analysis**: Detects breaking changes that affect other repositories
- 📝 **Inline Comments**: Posts targeted feedback directly on specific code lines
- ✅ **Summary Comments**: Provides holistic PR assessment with metadata
- 🎨 **Customizable Prompts**: Use your own custom review prompts while keeping the default as fallback

## Usage

### Prerequisites

1. This action works with the `pull_request` event
2. Ensure the repository is [checked out](https://github.com/actions/checkout) in your workflow
3. Set up required secrets and permissions

### Required Permissions

```yaml
permissions:
  contents: read
  pull-requests: write
```

### Basic Example

```yaml
name: "Tabnine Code Review"

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read
  pull-requests: write

jobs:
  code_review:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Tabnine Code Review
        uses: codota/tabnine-pr-agent@v1
        with:
          TABNINE_KEY: ${{ secrets.TABNINE_KEY }}
          github_token: ${{ secrets.GITHUB_TOKEN }}
          repository: ${{ github.repository }}
          pull_request_number: ${{ github.event.pull_request.number }}
          head_sha: ${{ github.event.pull_request.head.sha }}
          base_sha: ${{ github.event.pull_request.base.sha }}
```

### Advanced Example with Custom Prompt

```yaml
- name: Tabnine Code Review
  uses: codota/tabnine-pr-agent@v1
  with:
    TABNINE_KEY: ${{ secrets.TABNINE_KEY }}
    github_token: ${{ secrets.GITHUB_TOKEN }}
    repository: ${{ github.repository }}
    pull_request_number: ${{ github.event.pull_request.number }}
    head_sha: ${{ github.event.pull_request.head.sha }}
    base_sha: ${{ github.event.pull_request.base.sha }}
    tabnine_host: 'https://your-custom-host.com'
    model_id: 'your-custom-model-id'
    prompt: |
      Review this PR focusing on:
      1. Security vulnerabilities
      2. Performance issues
      3. Code maintainability
      
      Post inline comments for critical issues only.
      Provide a summary with risk assessment.
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `TABNINE_KEY` | Yes | - | Tabnine authentication credentials (JSON format) |
| `github_token` | Yes | - | GitHub token for API authentication (use `secrets.GITHUB_TOKEN`) |
| `repository` | Yes | - | Repository in `owner/repo` format (use `github.repository`) |
| `pull_request_number` | Yes | - | PR number to review (use `github.event.pull_request.number`) |
| `head_sha` | Yes | - | PR head commit SHA (use `github.event.pull_request.head.sha`) |
| `base_sha` | Yes | - | PR base commit SHA (use `github.event.pull_request.base.sha`) |
| `tabnine_host` | No | `https://console.tabnine.com` | Tabnine host URL (for self-hosted/EMT installations) |
| `model_id` | No | `c5ff943b-972a-45e7-9242-a3367c907074` | Model ID for the Tabnine CLI agent |
| `prompt` | No | *(comprehensive default)* | Custom prompt for the Tabnine CLI agent (see [Custom Prompts](#custom-prompts)) |

## Custom Prompts

The action includes a comprehensive default prompt that performs:
- Risk tier classification (Low/Standard/High Risk)
- Engineering audit across multiple dimensions (correctness, security, performance, etc.)
- Cross-repository impact analysis
- Infrastructure and configuration review
- Inline comments with severity levels
- Summary comments with metadata

You can override this with your own custom prompt by providing the `prompt` input. The custom prompt should instruct the Tabnine CLI on:
- What aspects of the code to review
- How to classify issues
- What format to use for comments
- When to post feedback

### Example Custom Prompt

```yaml
prompt: |
  You are a senior engineer reviewing this pull request.
  
  Focus on:
  1. Security: Check for SQL injection, XSS, auth issues
  2. Performance: Look for N+1 queries, missing indexes
  3. Tests: Verify test coverage for new features
  
  For each issue found:
  - Post an inline comment on the specific line
  - Use severity: [Critical], [Warning], or [Suggestion]
  - Explain why it matters and suggest a fix
  
  Post a summary comment with:
  - Overall assessment
  - List of key findings
  - What looks good about this PR
```

## Secrets Setup

Create a repository or organization secret named `TABNINE_KEY` containing your Tabnine authentication credentials in JSON format.

## How It Works

1. **Installation**: Downloads and installs the Tabnine CLI
2. **Authentication**: Configures Tabnine credentials and GitHub CLI authentication
3. **Cleanup**: Removes previous bot comments to avoid duplicates
4. **Review**: Executes the Tabnine CLI with the provided prompt
5. **Feedback**: Posts inline comments and summary directly on the PR

## Output

The action produces:
- **Inline review comments**: Posted on specific lines of code that need attention
- **Summary comment**: Posted on the PR with overall assessment and metadata
- Both types of comments are prefixed with "#### Tabnine PR Bot" for easy identification

## Troubleshooting

- **Authentication Failures**: Verify `TABNINE_KEY` is set correctly
- **No Comments Posted**: Check that the PR has actual code changes
- **Permission Errors**: Ensure `pull-requests: write` permission is granted

## License

See [LICENSE](LICENSE) file for details.
