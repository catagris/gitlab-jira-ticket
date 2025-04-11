# GitLab CI/CD Jira Branch Validation

A GitLab CI/CD job that enforces branch naming conventions based on Jira ticket IDs and optionally validates that the tickets exist in Jira.

## What It Does

This pipeline job:

1. Ensures branch names include a valid Jira ticket ID (e.g., `PROJECT-123-feature-description`)
2. Optionally connects to your Jira instance to verify the ticket actually exists
3. Blocks pushes and merge requests for branches that don't follow the convention
4. Allows configurable exceptions for protected branches

## Example Message

```
Found Jira ticket ID: SNX-111
✅ Ticket SNX-111 exists in Jira
```

## Setup Instructions

### 1. Add the File to Your Repository

Copy the `jira-branch-validation.yml` file to your GitLab repository.

### 2. Include It in Your Main Pipeline

Add this to your `.gitlab-ci.yml` file:

```yaml
include:
  - local: jira-branch-validation.yml
```

Or import from a separate repository:

```yaml
include:
  - project: 'your-group/ci-templates'
    ref: main
    file: 'jira-branch-validation.yml'
```

### 3. Set Up Jira API Access (Optional)

Set these variables in GitLab CI/CD settings:

- `JIRA_URL`: Your Jira instance URL (e.g., `https://your-company.atlassian.net`)
- `JIRA_USER_EMAIL`: Email address for Jira API authentication
- `JIRA_API_TOKEN`: API token for Jira API authentication

> **Note:** If API variables are not set, the job will only validate the branch name format without checking if the ticket actually exists in Jira.

## API Token Generation

To create a Jira API token:

1. Log in to your Atlassian account
2. Go to Account Settings → Security → Create and manage API tokens
3. Click "Create Classic API token"
4. Give it a name (e.g., "GitLab CI")
5. Copy the token and save it in GitLab CI/CD variables

## Customization Options

### Protected Branches

By default, the validation is skipped for branches named `stg`, `main`, and `master`.

To add additional protected branches, modify this line in the file:

```bash
if [[ "$BRANCH_NAME" == "stg" || "$BRANCH_NAME" == "main" || "$BRANCH_NAME" == "master" ]]; then
```

### Jira Project Keys

The job validates branch names against the pattern `[A-Z]+-[0-9]+`. This works for standard Jira project keys (e.g., `PROJECT-123`).

If you have custom project keys, you might need to adjust the regex pattern.

## How It Works

1. When a push or merge request happens, the job extracts the branch name
2. It checks if the branch name matches the Jira ticket ID pattern using regex
3. If API credentials are configured, it queries the Jira API to confirm the ticket exists
4. It allows the pipeline to continue if the validation passes, or fails it if not

## Error Handling

The job gracefully handles various error conditions:

- If Jira API credentials are missing, it only validates format
- If Jira API is unreachable, it allows the pipeline to continue
- If authentication fails, it warns but doesn't block the pipeline
- If the ticket doesn't exist, it fails the pipeline

## License

This project is available under the MIT License - see the LICENSE file for details.
