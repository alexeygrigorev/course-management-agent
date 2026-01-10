---
name: deploy-prod
description: Trigger production deployment workflow for course management platform
---

# Deploy to Production

## Overview

Triggers the manual production deployment workflow for the course-management-platform repository.

## Repository

- **Owner**: DataTalksClub
- **Repo**: course-management-platform
- **Workflow**: `.github/workflows/deploy-prod.yaml`

## Usage

### Basic Deployment (Auto-detect Tag)

```bash
gh workflow run -R DataTalksClub/course-management-platform deploy-prod.yaml -f confirmProdDeploy=true
```

### Deployment With Specific Tag

```bash
gh workflow run -R DataTalksClub/course-management-platform deploy-prod.yaml \
  -f confirmProdDeploy=true \
  -f deployTag=v1.2.3
```

## Inputs

| Input | Type | Required | Description |
|-------|------|----------|-------------|
| `confirmProdDeploy` | boolean | Yes | Must be `true` to confirm production deployment |
| `deployTag` | string | No | Specific tag version to deploy (auto-detected if omitted) |

## Check Deployment Status

```bash
# List recent workflow runs
gh run list -R DataTalksClub/course-management-platform --workflow=deploy-prod.yaml --limit 5

# Watch latest run
gh run watch -R DataTalksClub/course-management-platform

# View specific run details
gh run view -R DataTalksClub/course-management-platform <run-id>
```

## Get Last Deployed Version

```bash
gh api repos/DataTalksClub/course-management-platform/contents/.prod-versions --jq '.content | @base64d' | tail -1
```

Format: `YYYYMMDD-HHMMSS-commit_sha`

Example: `20260110-090705-3c79668`
- Date: 2026-01-10
- Time: 09:07:05 UTC
- Commit: 3c79668

## Verify Deployment (Health Endpoint)

```bash
curl -s https://courses.datatalks.club/data/health/
```

Response:
```json
{"status": "ok", "version": "20260110-100048-5b33100"}
```

The `version` should match the latest entry in `.prod-versions`.

### Poll Until Version Matches

It may take some time for the deployment to propagate. Poll every 10 seconds:

```bash
# Get expected version from .prod-versions
EXPECTED=$(gh api repos/DataTalksClub/course-management-platform/contents/.prod-versions --jq '.content | @base64d' | tail -1)

# Poll health endpoint until version matches (max 2 minutes)
for i in {1..12}; do
  VERSION=$(curl -s https://courses.datatalks.club/data/health/ | jq -r '.version')
  echo "[$i] Current: $VERSION, Expected: $EXPECTED"
  [[ "$VERSION" == "$EXPECTED" ]] && { echo "✓ Deployed!"; break; }
  [[ $i -lt 12 ]] && sleep 10
done
```

## What the Workflow Does

1. Runs `deploy/deploy_prod.sh` with optional tag
2. Uses AWS credentials from GitHub Secrets
3. Commits and pushes `.prod-versions` file with deployment record
4. Production updates health endpoint with deployed version

---

# Execution

1. Trigger the deployment workflow and capture the run ID
2. Watch the workflow run until completion
3. Get the expected version from `.prod-versions` file
4. Poll the health endpoint until the deployed version matches
5. Report final status to the user

Use `gh run watch` with a timeout of up to 3 minutes. For verification, poll the health endpoint up to 12 times (10 second intervals = 2 minutes) before reporting the final status.
