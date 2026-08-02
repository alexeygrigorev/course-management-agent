# Course Management Agent

Automation helper for interacting with the [Course Management Platform](https://github.com/DataTalksClub/course-management-platform).

## Overview

Helps manage courses, homeworks, projects, and deployments for DataTalksClub zoomcamps via CLI skills.

- `course-content`: Get or create homeworks and projects via API
- `homework-questions`: Get or create questions for homeworks via API
- `deploy-prod`: Trigger the production deployment workflow

## Setup

Store the API token in `.env`:

```text
AUTH_TOKEN=your_token_here
```

## Details

See [AGENTS.md](AGENTS.md) for full documentation on:

- Platform instances (prod/dev)
- Course repositories
- Slug patterns
- API usage
