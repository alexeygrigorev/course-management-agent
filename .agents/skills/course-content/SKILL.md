---
name: course-content
description: Manage courses, homeworks, and projects via REST API
---

# Course Content API

## Overview

This skill provides commands to manage courses, homeworks, and projects via the REST API. Supports list, create, update state/dates/description, and guarded delete.

## Configuration

- **Production instance**: `https://courses.datatalks.club`
- **Dev instance**: `https://dev.courses.datatalks.club`
- **Auth token**: Available as `AUTH_TOKEN` environment variable

## API Endpoints

### Courses

```
GET /api/courses/                              - List all courses
POST /api/courses/                             - Create course
GET /api/courses/<course_slug>/                - Course details with homeworks & projects
PATCH /api/courses/<course_slug>/              - Update course
```

### Homeworks

```
GET    /api/courses/<course_slug>/homeworks/          - List homeworks
POST   /api/courses/<course_slug>/homeworks/          - Create homework(s)
GET    /api/courses/<course_slug>/homeworks/<id>/     - Homework detail
PATCH  /api/courses/<course_slug>/homeworks/<id>/     - Update homework
DELETE /api/courses/<course_slug>/homeworks/<id>/     - Guarded delete
GET    /api/courses/<course_slug>/homeworks/by-slug/<slug>/    - Detail by slug
PUT    /api/courses/<course_slug>/homeworks/by-slug/<slug>/    - Create/update by slug
PATCH  /api/courses/<course_slug>/homeworks/by-slug/<slug>/    - Update by slug
DELETE /api/courses/<course_slug>/homeworks/by-slug/<slug>/    - Guarded delete by slug
```

### Projects

```
GET    /api/courses/<course_slug>/projects/           - List projects
POST   /api/courses/<course_slug>/projects/           - Create project(s)
GET    /api/courses/<course_slug>/projects/<id>/      - Project detail
PATCH  /api/courses/<course_slug>/projects/<id>/      - Update project
DELETE /api/courses/<course_slug>/projects/<id>/      - Guarded delete
GET    /api/courses/<course_slug>/projects/by-slug/<slug>/     - Detail by slug
PUT    /api/courses/<course_slug>/projects/by-slug/<slug>/     - Create/update by slug
PATCH  /api/courses/<course_slug>/projects/by-slug/<slug>/     - Update by slug
DELETE /api/courses/<course_slug>/projects/by-slug/<slug>/     - Guarded delete by slug
```

Full URL example:
- Production: `https://courses.datatalks.club/api/courses/ml-zoomcamp-2026/homeworks/`
- Dev: `https://dev.courses.datatalks.club/api/courses/ml-zoomcamp-2026/homeworks/`

## Authentication

The `AUTH_TOKEN` environment variable is already set — just use it directly:

```bash
-H "Authorization: Token ${AUTH_TOKEN}"
```

## Generated API Specification

Before changing course content, fetch the generated OpenAPI spec from the target environment and treat it as the source of truth:

```bash
curl -s "https://courses.datatalks.club/api/openapi.json" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

The OpenAPI endpoint is token-protected. Use it for current routes, request bodies, responses, and delete safety rules. This skill file is workflow guidance.

## API Request Best Practices

### ALWAYS follow this pattern for every request:

```bash
# 1. Make request (NEVER use jq - output raw JSON)
curl -s -X POST "URL" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### ALWAYS verify after creating:

```bash
# 2. Check the response with a separate GET request
curl -s -X GET "URL" -H "Authorization: Token ${AUTH_TOKEN}"
# 3. Verify items were created (check IDs, question count)
```

### NEVER do these:

```bash
# WRONG - Complex command substitution can fail
-H "Authorization: Token $(cat .env | grep AUTH_TOKEN | cut -d= -f2)"

# WRONG - Run multiple requests without verifying between them

# WRONG - Hardcode the token
```

## Listing Courses

```bash
curl -s -X GET "https://courses.datatalks.club/api/courses/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

**Response:**
```json
{
  "courses": [
    {"slug": "ml-zoomcamp-2026", "title": "ML Zoomcamp 2026", "description": "...", "finished": false}
  ]
}
```

## Getting Course Details

```bash
curl -s -X GET "https://courses.datatalks.club/api/courses/<course_slug>/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

**Response includes homeworks and projects with their IDs:**
```json
{
  "slug": "ml-zoomcamp-2026",
  "title": "ML Zoomcamp 2026",
  "homeworks": [
    {"id": 123, "slug": "hw1", "title": "Homework 1", "due_date": "...", "state": "CL"}
  ],
  "projects": [
    {"id": 456, "slug": "project1", "title": "Project 1", "submission_due_date": "...", "peer_review_due_date": "...", "state": "CL"}
  ]
}
```

Homework and project detail responses include `submissions_count`, `can_delete`, and `delete_blockers`. Prefer the `by-slug` endpoints for corrections when working from course repository files. Use `PUT` on `by-slug` routes for idempotent publish/rerun workflows.

## Creating Homeworks

### Single Homework

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Homework 1",
    "slug": "hw1",
    "due_date": "2026-03-15T23:59:59Z",
    "description": "Optional description"
  }'
```

### Bulk Create (Multiple Homeworks)

Send a JSON array:

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '[
    {"name": "Week 1: Introduction", "due_date": "2026-03-01T23:59:59Z"},
    {"name": "Week 2: Data Types", "due_date": "2026-03-08T23:59:59Z"}
  ]'
```

### Homework With Questions

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Homework: SQL Basics",
    "slug": "hw-sql-basics",
    "due_date": "2026-03-15T23:59:59Z",
    "questions": [
      {
        "text": "What does SQL stand for?",
        "question_type": "MC",
        "possible_answers": ["Structured Query Language", "Simple Query Language"],
        "correct_answer": "1",
        "scores_for_correct_answer": 1
      }
    ]
  }'
```

**Response (201):**
```json
{
  "created": [
    {"id": 123, "slug": "hw-sql-basics", "title": "Homework: SQL Basics", "state": "CL", "questions_count": 1, ...}
  ]
}
```

All homeworks are created with `state=CL` (closed). Use PATCH to open them.

## Updating Homeworks (PATCH)

Use the homework **ID** (from GET response) to update fields:

```bash
# Open a homework
curl -s -X PATCH "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "OP"}'

# Update multiple fields
curl -s -X PATCH "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "OP", "due_date": "2026-04-01T23:59:59Z", "description": "Updated description"}'
```

### Patchable Homework Fields

| Field | Description |
|-------|-------------|
| `state` | `CL` (Closed), `OP` (Open), `SC` (Scored) |
| `title` | Homework title |
| `description` | Homework description |
| `due_date` | Due date (ISO 8601) |
| `learning_in_public_cap` | Learning in public cap |
| `homework_url_field` | Show homework URL field (boolean) |
| `time_spent_lectures_field` | Show time spent on lectures field (boolean) |
| `time_spent_homework_field` | Show time spent on homework field (boolean) |
| `faq_contribution_field` | Show FAQ contribution field (boolean) |

## Deleting Homeworks

Homeworks can be deleted only when both conditions are true:

- The homework is `CL` closed.
- The homework has no submissions.

Do not use delete for a homework that has ever collected submissions. The API rejects deletion instead of deleting submission data.

For idempotent publishing, `PUT /api/courses/<course_slug>/homeworks/by-slug/<slug>/` creates the homework if missing or updates it if present. If the payload includes `questions` for an existing homework, the API replaces the current questions only when the homework is closed and has no submissions.

```bash
curl -s -X DELETE "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

## Creating Projects

### Single Project

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/projects/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Project 1",
    "slug": "project1",
    "submission_due_date": "2026-03-20T23:59:59Z",
    "peer_review_due_date": "2026-03-27T23:59:59Z",
    "description": "Optional description"
  }'
```

### Bulk Create (Multiple Projects)

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/projects/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '[
    {"name": "Midterm", "submission_due_date": "2026-03-20T23:59:59Z", "peer_review_due_date": "2026-03-27T23:59:59Z"},
    {"name": "Capstone", "submission_due_date": "2026-05-01T23:59:59Z", "peer_review_due_date": "2026-05-08T23:59:59Z"}
  ]'
```

All projects are created with `state=CL` (closed). Use PATCH to update state.

## Updating Projects (PATCH)

```bash
# Open a project for submissions
curl -s -X PATCH "https://courses.datatalks.club/api/courses/<course_slug>/projects/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "CS"}'
```

### Patchable Project Fields

| Field | Description |
|-------|-------------|
| `state` | `CL` (Closed), `CS` (Collecting Submissions), `PR` (Peer Reviewing), `CO` (Completed) |
| `title` | Project title |
| `description` | Project description |
| `submission_due_date` | Submission deadline (ISO 8601) |
| `peer_review_due_date` | Peer review deadline (ISO 8601) |
| `learning_in_public_cap_project` | LiP cap for project |
| `learning_in_public_cap_review` | LiP cap for review |
| `number_of_peers_to_evaluate` | Number of peers to review |
| `points_for_peer_review` | Points for peer review |
| `time_spent_project_field` | Show time spent field (boolean) |
| `problems_comments_field` | Show problems/comments field (boolean) |
| `faq_contribution_field` | Show FAQ contribution field (boolean) |

## Deleting Projects

Projects can be deleted only when both conditions are true:

- The project is `CL` closed.
- The project has no submissions.

Do not use delete for a project that has ever collected submissions. The API rejects deletion instead of deleting submission data.

For idempotent publishing, `PUT /api/courses/<course_slug>/projects/by-slug/<slug>/` creates the project if missing or updates it if present.

```bash
curl -s -X DELETE "https://courses.datatalks.club/api/courses/<course_slug>/projects/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

## Field Reference

### Homework Creation Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Homework title |
| `slug` | No | URL-friendly identifier (auto-generated from name if omitted) |
| `due_date` | Yes | Due date in ISO 8601 format (e.g., `2026-03-15T23:59:59Z`) |
| `description` | No | Homework description (defaults to empty string) |
| `questions` | No | Array of question objects |

### Project Creation Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Project title |
| `slug` | No | URL-friendly identifier (auto-generated from name if omitted) |
| `submission_due_date` | Yes | Submission deadline in ISO 8601 format |
| `peer_review_due_date` | Yes | Peer review deadline in ISO 8601 format |
| `description` | No | Project description (defaults to empty string) |

### Question Fields (inline with homework creation)

| Field | Required | Description |
|-------|----------|-------------|
| `text` | No | Question text |
| `question_type` | No | `MC`, `FF`, `FL`, or `CB` (defaults to `FF`) |
| `answer_type` | No | `ANY`, `FLT`, `INT`, `EXS`, or `CTS` |
| `possible_answers` | No | Array of answer options (for MC/CB) |
| `correct_answer` | No | Correct answer (index for MC/CB, value for others) |
| `scores_for_correct_answer` | No | Points for correct answer (default: 1) |

## Question Naming

**Use minimal, concise question text**. Students can see full details in homework.md.

```json
// GOOD - concise
"text": "dbt run --select int_trips_unioned builds which models?"

// BAD - too verbose
"text": "Given a dbt project with staging models..."
```

## Date Formats

Both ISO formats are supported:
- `2026-03-15T23:59:59Z` (UTC with Z)
- `2026-03-15T23:59:59+00:00` (UTC with offset)

## Important Notes

- **NEVER use jq** in curl commands - output raw JSON only
- **Keep question text minimal** - students get full context from homework.md file
- **POST creates NEW items** - check for duplicates before creating
- **Use PATCH to change state** - homeworks/projects are created as closed
- **DELETE is guarded** - only delete closed homeworks/projects with no submissions

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication token required` | Missing/invalid token | Check `AUTH_TOKEN` env var |
| `Course not found` | Invalid course slug | Use GET `/api/courses/` to list courses |
| `already exists` | Slug conflict | Use a different slug |
| `Invalid date format` | Malformed date | Use ISO 8601 format |
| `Only closed homeworks can be deleted` | Trying to delete non-closed | PATCH state to CL first |
| `Cannot delete homework with existing submissions` | Homework has submissions | Do not delete; leave it closed |
| `Only closed projects can be deleted` | Trying to delete non-closed | PATCH state to CL first |
| `Cannot delete project with existing submissions` | Project has submissions | Do not delete; leave it closed |
| `Cannot update field: X` | Invalid field in PATCH | Check patchable fields list |
