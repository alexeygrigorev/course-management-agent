---
name: course-content
description: Get or add homeworks and projects to courses via API using curl
---

# Course Content API

## Overview

This skill provides commands to get and create homeworks and projects for courses via the API endpoint. All items are created with `state=CLOSED` (not visible to students).

## Configuration

- **Production instance**: `https://courses.datatalks.club`
- **Dev instance**: `https://dev.courses.datatalks.club`
- **Auth token**: Available as `AUTH_TOKEN` environment variable

## API Endpoint

```
GET /data/<course_slug>/content - Get all homeworks and projects
POST /data/<course_slug>/content - Create new homeworks and projects
```

Full URLs:
- Production: `https://courses.datatalks.club/data/<course_slug>/content`
- Dev: `https://dev.courses.datatalks.club/data/<course_slug>/content`

## Authentication

Use `${AUTH_TOKEN}` environment variable:

```bash
-H "Authorization: Token ${AUTH_TOKEN}"
```

### Troubleshooting AUTH_TOKEN

If the API returns authentication errors, verify the token is set:

```bash
# Check if token is set (should show length, not "NOT SET")
echo "AUTH_TOKEN is: $(if [ -n "$AUTH_TOKEN" ]; then echo "SET (${#AUTH_TOKEN} chars)"; else echo "NOT SET"; fi)"

# Test with a simple GET request
curl -s -X GET "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

If the token is not set, ensure it's loaded from `.env` or set explicitly.

## API Request Best Practices

### ALWAYS follow this pattern for every request:

```bash
# 1. Make POST request (NEVER use jq - output raw JSON)
curl -s -X POST "URL" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### ALWAYS verify after creating:

```bash
# 2. Check the response with a separate GET request
curl -s -X GET "URL" -H "Authorization: Token ${AUTH_TOKEN}"
# 3. Verify items were created (check IDs, success status, question count)
```

### NEVER do these:

```bash
# WRONG - Complex command substitution can fail (leaves newlines, spaces)
-H "Authorization: Token $(cat .env | grep AUTH_TOKEN | cut -d= -f2)"

# WRONG - Run multiple requests without verifying between them

# WRONG - Assume "no output" means "failed" (silent success is possible)

# WRONG - Hardcode the token
```

### Critical Rule:

**The API creates NEW items every time - it never replaces or updates existing items.** Check for duplicates before creating.

## Getting Course Content (GET)

```bash
curl -X GET "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

**Response:**
```json
{
  "success": true,
  "course": "course-slug",
  "homeworks": [
    {
      "id": 123,
      "slug": "hw-1",
      "title": "Homework 1",
      "due_date": "2025-03-15T23:59:59Z",
      "state": "CL",
      "questions_count": 5
    }
  ],
  "projects": [
    {
      "id": 456,
      "slug": "project-1",
      "title": "Project 1",
      "submission_due_date": "2025-03-20T23:59:59Z",
      "peer_review_due_date": "2025-03-27T23:59:59Z",
      "state": "CL"
    }
  ]
}
```

## Creating Homeworks

### Question Naming

**Use minimal, concise question text**. Students can see full details in homework.md.

- Prefix with "Q1:", "Q2:", etc.
- Keep text short - focus on the core question
- Omit lengthy context, setup, or explanations

**Example:**

```json
// GOOD - concise
"text": "Q1: dbt run --select int_trips_unioned builds which models?"

// BAD - too verbose
"text": "Given a dbt project with staging models (stg_green_tripdata, stg_yellow_tripdata) and an intermediate model int_trips_unioned that depends on both staging models. If you run dbt run --select int_trips_unioned, what models will be built?"
```

### Basic Homework (No Questions)

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks": [
      {
        "name": "Homework 1",
        "slug": "hw-1",
        "due_date": "2025-03-15T23:59:59Z",
        "description": "Optional description"
      }
    ]
  }'
```

### Homework With Questions

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks": [
      {
        "name": "Homework: SQL Basics",
        "slug": "hw-sql-basics",
        "due_date": "2025-03-15T23:59:59Z",
        "description": "Practice SQL queries",
        "questions": [
          {
            "text": "What does SQL stand for?",
            "question_type": "MC",
            "answer_type": "EXS",
            "possible_answers": ["Structured Query Language", "Simple Query Language", "Standard Query Language"],
            "correct_answer": "1",
            "scores_for_correct_answer": 1
          },
          {
            "text": "Write a SELECT statement to get all users from the 'users' table",
            "question_type": "FF",
            "answer_type": "CTS",
            "correct_answer": "SELECT",
            "scores_for_correct_answer": 2
          }
        ]
      }
    ]
  }'
```

### Multiple Homeworks

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks": [
      {
        "name": "Week 1: Introduction",
        "due_date": "2025-03-01T23:59:59Z"
      },
      {
        "name": "Week 2: Data Types",
        "due_date": "2025-03-08T23:59:59Z"
      },
      {
        "name": "Week 3: Functions",
        "due_date": "2025-03-15T23:59:59Z"
      }
    ]
  }'
```

## Creating Projects

### Basic Project

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "projects": [
      {
        "name": "Project 1: Build a Dashboard",
        "slug": "project-1-dashboard",
        "submission_due_date": "2025-03-20T23:59:59Z",
        "peer_review_due_date": "2025-03-27T23:59:59Z",
        "description": "Create an interactive dashboard"
      }
    ]
  }'
```

### Multiple Projects

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "projects": [
      {
        "name": "Project 1: ETL Pipeline",
        "submission_due_date": "2025-03-20T23:59:59Z",
        "peer_review_due_date": "2025-03-27T23:59:59Z"
      },
      {
        "name": "Project 2: ML Model",
        "submission_due_date": "2025-04-10T23:59:59Z",
        "peer_review_due_date": "2025-04-17T23:59:59Z"
      }
    ]
  }'
```

## Creating Both Homeworks and Projects

```bash
curl -X POST "https://courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "homeworks": [
      {
        "name": "Homework 1",
        "due_date": "2025-03-15T23:59:59Z"
      }
    ],
    "projects": [
      {
        "name": "Project 1",
        "submission_due_date": "2025-03-20T23:59:59Z",
        "peer_review_due_date": "2025-03-27T23:59:59Z"
      }
    ]
  }'
```

## Field Reference

### Homework Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Homework title |
| `slug` | No | URL-friendly identifier (auto-generated from name if omitted) |
| `due_date` | Yes | Due date in ISO 8601 format (e.g., `2025-03-15T23:59:59Z`) |
| `description` | No | Homework description (defaults to empty string) |
| `questions` | No | Array of question objects |

### Question Fields

| Field | Required | Description |
|-------|----------|-------------|
| `text` | No | Question text |
| `question_type` | No | `MC`, `FF`, `FL`, or `CB` (defaults to `FF`) |
| `answer_type` | No | `ANY`, `FLT`, `INT`, `EXS`, or `CTS` |
| `possible_answers` | No | Array of answer options (for MC/CB) |
| `correct_answer` | No | Correct answer (index for MC/CB, value for others) |
| `scores_for_correct_answer` | No | Points for correct answer (default: 1) |

### Project Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Project title |
| `slug` | No | URL-friendly identifier (auto-generated from name if omitted) |
| `submission_due_date` | Yes | Submission deadline in ISO 8601 format |
| `peer_review_due_date` | Yes | Peer review deadline in ISO 8601 format |
| `description` | No | Project description (defaults to empty string) |

## Question Types

| Code | Name | Description |
|------|------|-------------|
| `MC` | Multiple Choice | Single correct answer from a list of options |
| `FF` | Free Form | Short text answer (1-2 sentences) |
| `FL` | Free Form Long | Long text answer (essays, explanations) |
| `CB` | Checkboxes | Multiple correct answers from a list of options |

## Answer Types

| Code | Name | Description |
|------|------|-------------|
| `ANY` | Any | Any input is accepted (no validation) |
| `FLT` | Float | Decimal number validation (e.g., 3.14, -0.5) |
| `INT` | Integer | Whole number validation (e.g., 1, 42, -7) |
| `EXS` | Exact String | Answer must match exactly (case-sensitive) |
| `CTS` | Contains String | Answer must contain the specified text |

## Date Formats

Both ISO formats are supported:
- `2025-03-15T23:59:59Z` (UTC with Z)
- `2025-03-15T23:59:59+00:00` (UTC with offset)

## Response Format

### GET Response
```json
{
  "success": true,
  "course": "course-slug",
  "homeworks": [
    {
      "id": 123,
      "slug": "hw-1",
      "title": "Homework 1",
      "due_date": "2025-03-15T23:59:59Z",
      "state": "CL",
      "questions_count": 5
    }
  ],
  "projects": [
    {
      "id": 456,
      "slug": "project-1",
      "title": "Project 1",
      "submission_due_date": "2025-03-20T23:59:59Z",
      "peer_review_due_date": "2025-03-27T23:59:59Z",
      "state": "CL"
    }
  ]
}
```

### POST Response
```json
{
  "success": true,
  "course": "course-slug",
  "created_homeworks": [
    {
      "id": 123,
      "slug": "hw-1",
      "title": "Homework 1",
      "due_date": "2025-03-15T23:59:59Z",
      "state": "CL",
      "questions_count": 2
    }
  ],
  "created_projects": [
    {
      "id": 456,
      "slug": "project-1",
      "title": "Project 1",
      "submission_due_date": "2025-03-20T23:59:59Z",
      "peer_review_due_date": "2025-03-27T23:59:59Z",
      "state": "CL"
    }
  ],
  "errors": []
}
```

## Error Handling

Partial success is supported - if some items fail, others are still created:

```json
{
  "success": true,
  "course": "ml-zoomcamp",
  "created_homeworks": [
    {"id": 123, "slug": "hw-1", ...}
  ],
  "created_projects": [],
  "errors": [
    {"homework": "Duplicate", "error": "Homework with this slug already exists"}
  ]
}
```

## Important Notes

- **NEVER use jq** in curl commands - even if installed, output raw JSON only
- **Keep question text minimal** - students get full context from homework.md file
- **API creates NEW items** - never replaces or updates existing items (check for duplicates)

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication token required` | Missing/invalid token | Check `AUTH_TOKEN` env var |
| `Course not found` | Invalid course slug | Verify course exists |
| `already exists` | Slug conflict | Use a different slug |
| `Invalid date format` | Malformed date | Use ISO 8601 format |
