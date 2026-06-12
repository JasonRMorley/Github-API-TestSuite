# GitHub API Support Suite

This repository contains a Postman and Newman test suite for validating key GitHub repository API workflows. The suite exercises repository creation, retrieval, update, and deletion, including positive and negative scenarios.

## Repository Contents

- `Postman/github-api.postman_collection.json` - Postman collection containing the GitHub API tests.
- `Postman/github-api-environment.template.json` - Environment template for local and CI execution.
- `Jenkinsfile` - Jenkins pipeline for running the collection with Newman and publishing an HTML report.

## Test Coverage

The collection currently validates these GitHub repository API flows:

- Create a repository.
- Reject repository creation when the name is missing.
- Reject repository creation with an invalid token.
- Reject duplicate repository creation.
- Retrieve an existing repository.
- Return `404` for missing or invalid repository lookups.
- Update repository metadata.
- Reject repository updates with invalid credentials or missing repositories.
- Delete a repository.
- Reject repository deletion with invalid credentials or missing repositories.

## Prerequisites

- Node.js and npm.
- Newman.
- Newman HTML Extra reporter.
- A GitHub fine-grained personal access token with permissions to create, read, update, and delete repositories in the target account.

Install the Newman tooling with:

```bash
npm install -g newman newman-reporter-htmlextra
```

## Environment Variables

The Postman environment uses these variables:

| Variable | Purpose |
| --- | --- |
| `github-fine-token` | GitHub fine-grained token used for authenticated API requests. |
| `base_url` | GitHub API base URL. Defaults to `https://api.github.com`. |
| `repoOwner` | GitHub account or organization that owns the test repository. |
| `repoName` | Repository name used by the tests. The create test generates a unique value at runtime. |
| `repoId` | Repository ID captured after creation. |
| `repoFullname` | Full repository name captured after creation. |
| `updatedDescription` | Description value used by the update repository test. |

Do not commit real tokens. Keep `github-fine-token` empty in committed environment files and inject it locally or through CI credentials.

## Running Locally

Export your GitHub token and run the collection with Newman:

```bash
newman run Postman/github-api.postman_collection.json \
  -e Postman/github-api-environment.template.json \
  --env-var "github-fine-token=$GITHUB_FINE_TOKEN" \
  --env-var "base_url=https://api.github.com" \
  --env-var "repoOwner=<your-github-username>" \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman-report.html
```

On Windows PowerShell, use:

```powershell
newman run Postman/github-api.postman_collection.json `
  -e Postman/github-api-environment.template.json `
  --env-var "github-fine-token=$env:GITHUB_FINE_TOKEN" `
  --env-var "base_url=https://api.github.com" `
  --env-var "repoOwner=<your-github-username>" `
  -r cli,htmlextra `
  --reporter-htmlextra-export newman-report.html
```

The run creates `newman-report.html` in the repository root.

## Running In Jenkins

The Jenkins pipeline:

1. Checks out the `main` branch.
2. Verifies Node.js, npm, Newman, and `newman-reporter-htmlextra`.
3. Injects the GitHub token from the `github-fine-token` Jenkins credential.
4. Runs the Postman collection with Newman.
5. Archives and publishes `newman-report.html`.

Before running the pipeline, configure a Jenkins secret text credential with the ID `github-fine-token`.

## Operational Notes

- The suite creates and deletes real GitHub repositories. Use a dedicated test account or organization.
- The GitHub token should have the minimum permissions needed for repository administration in the test scope.
- Failed runs can leave a test repository behind if execution stops before the delete step. Clean up any leftover repositories before rerunning duplicate-name scenarios.
- GitHub rate limits and permission changes can affect test results.

