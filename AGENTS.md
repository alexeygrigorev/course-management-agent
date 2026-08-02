# Agent Instructions

Claude agents must read this `AGENTS.md` file before working in this repository.
Use this file for Claude and Codex workflow guidance.

## Overview

This project enables interaction with the Course Management Platform to manage
courses, homeworks, projects, and deployments.

## Platform Instances

Use these platform locations:

- Production: `https://courses.datatalks.club`
- Development: `https://dev.courses.datatalks.club`
- Repository: `DataTalksClub/course-management-platform`

## Authentication

Use token authentication for API calls:

- `AUTH_TOKEN` environment variable (stored in `.env`)
- Works for both dev and prod instances

## Course Repositories

All course content is located in `~/git/`:

- `~/git/data-engineering-zoomcamp/`: Data Engineering course
- `~/git/machine-learning-zoomcamp/`: ML course
- `~/git/mlops-zoomcamp/`: MLOps course
- `~/git/llm-zoomcamp/`: LLM course
- `~/git/ai-dev-tools-zoomcamp/`: AI Dev Tools course

This repository has local Claude/Codex workflow skills under `.claude/skills`.
Before working on course platform content, look at the relevant local skill docs.

## Local Skills

Use these local skills when the request matches them:

- `course-content`: Manage courses, homeworks, and projects through the course platform REST API. Use this for listing, creating, updating dates, changing states, changing descriptions, and guarded deletion of course content.
- `homework-questions`: Manage homework questions through the REST API. Use this after identifying homework IDs with `course-content`.
- `deploy-prod`: Trigger the production deployment workflow for `DataTalksClub/course-management-platform`.

## Notes

- The generated OpenAPI spec from the target environment is the source of truth before changing course content.
- Local skill docs are workflow guidance and may be more specific than global skills exposed by the runtime.
- When the user gives a short instruction such as `create hw2 for llm zoomcamp 2026`, restate the concrete interpretation before acting.

For `create hw2 for llm zoomcamp 2026`, use this workflow:

- Identify the `llm-zoomcamp-2026` course.
- Read the cohort homework source from `~/git/llm-zoomcamp/cohorts/2026/`.
- Find answers in the course repo root `.tmp/<homework-slug>/` solutions files when available.
- Check common answer filenames such as `solutions.md` and `solution.md`.
- Create or update the platform homework and questions.
- Verify changes with GET requests.
- Open the homework when publishing is intended.

## API Source of Truth

Before changing course content, fetch the generated OpenAPI specification from
the target environment:

```console
$ curl -s "https://courses.datatalks.club/api/openapi.json" \
    -H "Authorization: Token ${AUTH_TOKEN}"
```

Use the generated OpenAPI spec as the source of truth for:

- Current routes
- Request bodies
- Responses
- Authentication
- Delete safety rules

Local skill docs are workflow guidance only.

## Courses on Production

Active courses are accessed via `https://courses.datatalks.club/<course_slug>/`.
Confirm the current list with the API before making changes.

Use these common course slugs as a starting point:

- `de-zoomcamp-2026`: Data Engineering Zoomcamp 2026
- `de-zoomcamp-2025`: Data Engineering Zoomcamp 2025
- `de-zoomcamp-2024`: Data Engineering Zoomcamp 2024
- `ml-zoomcamp-2025`: Machine Learning Zoomcamp 2025
- `ml-zoomcamp-2024`: Machine Learning Zoomcamp 2024
- `llm-zoomcamp-2026`: LLM Zoomcamp 2026
- `llm-zoomcamp-2025`: LLM Zoomcamp 2025
- `llm-zoomcamp-2024`: LLM Zoomcamp 2024
- `mlops-zoomcamp-2025`: MLOps Zoomcamp 2025
- `mlops-zoomcamp-2024`: MLOps Zoomcamp 2024
- `sma-zoomcamp-2025`: SMA Zoomcamp 2025
- `sma-zoomcamp-2024`: SMA Zoomcamp 2024
- `ai-dev-tools-2025`: AI Dev Tools 2025

## Course Slug Patterns

Course slugs use the `<course_short>-zoomcamp-<year>` format:

- Data Engineering: `de-zoomcamp`
- Machine Learning: `ml-zoomcamp`
- MLOps: `mlops-zoomcamp`
- LLM: `llm-zoomcamp`
- AI Dev Tools: `ai-dev-tools`
- SMA: `sma-zoomcamp`

## Homework Slug Patterns

Homework slugs usually use `hw1`, `hw2`, `hw01`, or `hw02` formats.
Examples include:

- `hw1`, `hw2`, `hw3` ... `hw10`
- Special: `dlt` (workshop), `agents`

## Project Slug Patterns

Project slugs use values such as:

- `project1`, `project2`, `project3`
- `midterm`
- `capstone1`, `capstone2`

## States

Use these state codes:

- `CL`: Closed
- `OP`: Open
- `SC`: Scored (closed)
- `CO`: Completed
- `PR`: Peer reviewing
- `CS`: Collecting submissions
