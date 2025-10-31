# PR Approval Check

A GitHub Action that requires approval from a specific GitHub organization team member before a pull request can be merged. The action checks PR reviews and verifies that at least one reviewer is an active member of the specified team and has approved the PR.

## Features

- ✅ Validates PR approvals from specific team members
- ✅ Sets commit status checks based on approval state
- ✅ Detects and reports change requests from team members
- ✅ Works with GitHub Apps and personal access tokens

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `org` | GitHub organization that owns the approver team | No | `trufflesecurity` |
| `approver_team` | Team slug that must approve (e.g. `product-eng`) | No | `product-eng` |
| `github_token` | Token with permission to read PRs and write commit statuses | Yes | - |

## Usage

### Basic Example

```yaml
name: PR Approval Check

on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  approval:
    runs-on: ubuntu-latest
    steps:
      - name: Require Product Eng approval
        uses: trufflesecurity/pr-approval-check@v1
        with:
          org: trufflesecurity
          approver_team: product-eng
          github_token: ${{ secrets.GITHUB_TOKEN }}
```

### Using GitHub App Authentication

For enhanced security and better rate limits, use a GitHub App:

```yaml
on:
  pull_request:
    types: [opened, reopened, synchronize]

jobs:
  approval:
    runs-on: ubuntu-latest
    steps:
      - name: Mint installation token
        id: app-token
        uses: actions/create-github-app-token@v2
        with:
          app-id: ${{ secrets.PR_APPROVAL_CHECK_APP_ID }}
          private-key: ${{ secrets.PR_APPROVAL_CHECK }}

      - name: Require Product Eng approval
        uses: trufflesecurity/pr-approval-check@v1
        with:
          org: trufflesecurity
          approver_team: product-eng
          github_token: ${{ steps.app-token.outputs.token }}
```

## How It Works

1. The action sets an initial `pending` commit status
2. It fetches all reviews for the pull request
3. For each reviewer, it checks if they are an active member of the specified team
4. If a team member has approved, the action succeeds
5. If a team member has requested changes, the action fails with their username
6. If no team member has approved, the action fails

## Commit Status

The action creates a commit status with the context `pr-approval-check` that can be used as a required status check in branch protection rules:

- **Success**: ✅ Approved by `@org/team-name`
- **Failure**: ⚠️ Requires approval from `@org/team-name` or changes requested by team member
- **Pending**: Waiting for approval from `@org/team-name` team member

## Requirements

The GitHub token must have the following permissions:
- `pull-requests: read` - to read PR reviews
- `statuses: write` - to set commit status checks
- `members: read` - to check team membership (if using a GitHub App)

