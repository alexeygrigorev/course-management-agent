# Course Management Agent

Automation helper for interacting with the [Course Management Platform](https://github.com/DataTalksClub/course-management-platform).

## What This Does

Helps manage courses, homeworks, projects, and deployments for DataTalksClub zoomcamps via CLI skills.

- **course-content** - Get/create homeworks and projects via API
- **homework-questions** - Get/create questions for homeworks via API
- **deploy-prod** - Trigger production deployment workflow

## Setup

```bash
# Auth token stored in .env
AUTH_TOKEN=your_token_here
```

## Details

See [CLAUDE.md](CLAUDE.md) for full documentation on:
- Platform instances (prod/dev)
- Course repositories
- Slug patterns
- API usage
