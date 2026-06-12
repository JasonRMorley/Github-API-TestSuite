# Test Strategy

## Purpose

This strategy defines how the GitHub API Support Suite validates GitHub repository API behavior using Postman, Newman, and Jenkins. The goal is to provide repeatable API-level coverage for common repository lifecycle operations while keeping credentials and generated test data controlled.

## Scope

The suite covers repository lifecycle behavior against the GitHub REST API:

- Create repository.
- Retrieve repository details.
- Update repository metadata.
- Delete repository.
- Validate expected failures for invalid input, invalid authentication, duplicate data, and missing resources.

The suite does not currently cover:

- Branch, commit, issue, pull request, release, or workflow APIs.
- Performance, load, or soak testing.
- GitHub Enterprise Server compatibility.
- Cross-browser or UI behavior.
- Contract validation against a generated OpenAPI schema.

## Test Levels

| Level | Tooling | Purpose |
| --- | --- | --- |
| API integration | Postman and Newman | Validate live GitHub API behavior and request sequencing. |
| CI regression | Jenkins and Newman | Run the collection consistently and publish an execution report. |
| Negative testing | Postman test scripts | Confirm expected error handling for invalid or missing inputs. |

## Test Data Strategy

The positive repository lifecycle flow creates a unique repository name at runtime using a timestamp. Values returned by the GitHub API are stored in the Postman environment and reused by later requests.

Runtime state captured by the suite includes:

- `repoName`
- `repoId`
- `repoFullname`
- `repoOwner`

This approach reduces dependency on pre-existing repositories and allows the collection to validate a complete create-read-update-delete workflow.

## Environment Strategy

Tests should run against `https://api.github.com` unless a deliberate GitHub Enterprise Server target is added later. The current committed environment file is a template and must not contain live secrets.

Required environment configuration:

- `github-fine-token` for authenticated requests.
- `repoOwner` for the target GitHub account or organization.
- `base_url` for the GitHub API host.
- `updatedDescription` for the update assertion.

CI should inject secrets from Jenkins credentials rather than storing them in source control.

## Coverage Matrix

| Area | Scenario | Expected Result |
| --- | --- | --- |
| Create repository | Valid repository payload | `201 Created` and response matches generated repository name. |
| Create repository | Missing required name | `422 Unprocessable Entity`. |
| Create repository | Invalid token | `401 Unauthorized`. |
| Create repository | Duplicate repository name | `422 Unprocessable Entity`. |
| Get repository | Existing repository | `200 OK` and response fields match environment state. |
| Get repository | Missing repository | `404 Not Found`. |
| Get repository | Invalid owner | `404 Not Found`. |
| Update repository | Valid metadata update | `200 OK` and description matches expected value. |
| Update repository | Invalid token | `401 Unauthorized`. |
| Update repository | Missing repository | `404 Not Found`. |
| Delete repository | Existing repository | `204 No Content`. |
| Delete repository | Invalid token | `401 Unauthorized`. |
| Delete repository | Missing repository | `404 Not Found`. |

## Entry Criteria

- Newman and `newman-reporter-htmlextra` are installed.
- GitHub token is available through a local environment variable or Jenkins credential.
- Token has permission to administer repositories in the test owner context.
- No leftover repository exists that would invalidate duplicate-name or cleanup assumptions.
- GitHub API is reachable from the runner.

## Exit Criteria

- All Newman assertions pass.
- The generated HTML report is archived or available for review.
- Any repository created during the test run is deleted.
- Failures are triaged as product behavior, test data, environment, permissions, or suite defects.

## CI Execution

Jenkins is the primary CI executor. The pipeline runs the collection with:

- `cli` reporter for build logs.
- `htmlextra` reporter for a browsable HTML artifact.
- Runtime environment variable overrides for token, owner, repository name, and base URL.

The build should fail when Newman reports failed assertions or request errors.

## Risk Areas

- **Real resource creation:** The suite creates actual GitHub repositories and needs reliable cleanup.
- **Credential permissions:** Fine-grained token scopes can cause false failures if repository administration permissions are incomplete.
- **Rate limits:** Repeated runs can be affected by GitHub API rate limiting.
- **State leakage:** Interrupted runs can leave repositories behind.
- **External dependency:** GitHub API availability and behavior can change independently of this repository.

## Defect Triage

When a test fails, classify the failure before changing the suite:

- API behavior changed.
- Token expired or lacks required permissions.
- Test data was not cleaned up.
- Request ordering or environment variable state is incorrect.
- Assertion is too strict or no longer aligned with expected GitHub behavior.
- GitHub service or network availability issue.

## Maintenance Guidelines

- Keep request names descriptive and stable.
- Add negative tests when new positive workflows are added.
- Keep secrets out of committed files.
- Review token permissions periodically and keep them minimal.
- Ensure destructive operations are paired with cleanup or documented manual recovery steps.
- Update this strategy when the suite expands beyond repository lifecycle APIs.

