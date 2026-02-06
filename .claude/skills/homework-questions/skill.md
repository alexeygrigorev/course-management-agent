---
name: homework-questions
description: Get or add questions to homeworks via API using curl
---

# Homework Questions API

## Overview

This skill provides commands to get and create questions for homeworks via the API endpoint.

## Configuration

- **Production instance**: `https://courses.datatalks.club`
- **Dev instance**: `https://dev.courses.datatalks.club`
- **Auth token**: Available as `AUTH_TOKEN` environment variable, works for both dev and prod

## Workflow

0. **Find homework slugs**: If you don't know the homework slug, first use the `course-content` skill to list all homeworks and get their slugs
1. **Read the homework file**: typically it's named homework.md
2. **Read solutions file**: Analyze the content of the homework folder. There could be the solutions.md file (or similar) with the answers
3. **Check existing questions**: ALWAYS verify current question count BEFORE creating
4. **Create questions**: POST questions with correct answers to the homework
5. **Verify creation**: ALWAYS check question count AFTER creating and confirm `"success": true` in response
6. **Open homework**: Always include `"state": "OP"` when creating questions to open the homework
7. **Provide summary**: List all questions with their answers and include the homework link

## API Endpoint

```
GET /data/<course_slug>/homework/<homework_slug>/content - Get homework details and questions
POST /data/<course_slug>/homework/<homework_slug>/content - Create questions for homework
```

Full URLs:
- Production: `https://courses.datatalks.club/data/<course_slug>/homework/<homework_slug>/content`
- Dev: `https://dev.courses.datatalks.club/data/<course_slug>/homework/<homework_slug>/content`

## Authentication

`AUTH_TOKEN` environment variable

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
# 1. Make POST request (do NOT pipe to jq - may cause silent output)
curl -s -X POST "URL" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

### ALWAYS verify after creating:

```bash
# 2. Check the response with a separate GET request
curl -s -X GET "URL" -H "Authorization: Token ${AUTH_TOKEN}"
```

### NEVER do these:

```bash
# WRONG - Complex command substitution can fail (leaves newlines, spaces)
-H "Authorization: Token $(cat .env | grep AUTH_TOKEN | cut -d= -f2)"

# WRONG - Run multiple requests without verifying between them

# WRONG - Assume "no output" means "failed" (silent success is possible)
```

### Critical Rule:

**ALWAYS verify the question count BEFORE and AFTER creating.** The API adds new questions every time - it never replaces or deduplicates existing questions.

## Step 1: Find Homework Slugs

Use the course-content endpoint to list all homeworks:

```bash
curl -X GET "https://dev.courses.datatalks.club/data/<course_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

This returns all homeworks with their slugs, titles, and due dates. Find the target homework's `slug`.

## Step 2: Read Solutions File

Read the solutions from the cohort directory to get correct answers:

```bash
# Example path
~/git/data-engineering-zoomcamp/cohorts/2026/01-docker-terraform/solution.md
```

**If no solution file exists** (e.g., homework says "Will be added after the due date"), create questions WITHOUT the `correct_answer` field. Submit like usual - students can still submit, but there will be no auto-grading.

**General rule**: Do NOT specify `answer_type` unless explicitly asked. It is optional and defaults to appropriate value.

## Step 3: Create Questions

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

### Basic Question (Single Answer - MC)

```bash
curl -X POST "https://dev.courses.datatalks.club/data/<course_slug>/homework/<homework_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "questions": [
      {
        "text": "What does SQL stand for?",
        "question_type": "MC",
        "possible_answers": ["Structured Query Language", "Simple Query Language", "Standard Query Language"],
        "correct_answer": "1",
        "scores_for_correct_answer": 1
      }
    ],
    "state": "OP"
  }'
```

### Multiple Correct Answers

If a homework question has multiple valid answers (e.g., "select any of them"), use `MC` with ONE correct answer. The student can select any valid option.

Only use `CB` (checkboxes) if explicitly requested - it requires students to select ALL correct answers.

```bash
# Typical case: multiple valid answers, use MC with one
"question_type": "MC",
"correct_answer": "4"

# Only if explicitly requested: checkboxes for ALL correct answers
"question_type": "CB",
"correct_answer": "4,5"
```

### State Update

Always include `"state": "OP"` to open the homework when creating questions:

```bash
curl -X POST "https://dev.courses.datatalks.club/data/<course_slug>/homework/<homework_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "questions": [
      ...
    ],
    "state": "OP"
  }'
```

## Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | No | Question text |
| `question_type` | string | No | `MC` (default), `FF`, `FL`, or `CB` (only if explicitly requested) |
| `answer_type` | string | No | `ANY`, `FLT`, `INT`, `EXS`, or `CTS` |
| `possible_answers` | array | No | Array of answer options (for MC questions) |
| `correct_answer` | string | No | For MC: 1-based index (`"1"`), for CB: comma-separated (`"1,3"`) |
| `scores_for_correct_answer` | int | No | Points for correct answer (default: 1) |
| `state` | string | No | Include `"OP"` to open homework when creating |

## Question Types

| Code | Name | Correct Answer Format |
|------|------|----------------------|
| `MC` | Multiple Choice | Single index: `"1"` (use for most questions) |
| `FF` | Free Form | Text value: `"answer"` |
| `FL` | Free Form Long | Text value: `"answer"` |
| `CB` | Checkboxes | Comma-separated indices: `"1,3,4"` (ONLY if explicitly requested) |

## Answer Types

| Code | Name | Description |
|------|------|-------------|
| `ANY` | Any | No validation |
| `FLT` | Float | Decimal number |
| `INT` | Integer | Whole number |
| `EXS` | Exact String | Exact match |
| `CTS` | Contains String | Contains text |

## Step 4: Verify and Provide Summary

After creating questions, verify and provide a summary:

```bash
# Verify questions were created
curl -X GET "https://dev.courses.datatalks.club/data/<course_slug>/homework/<homework_slug>/content" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

## Step 5. Give the user the URL to check the homework


```
Created N questions for "<Homework Title>"

Homework link: https://courses.datatalks.club/<course_slug>/homework/<homework_slug>


| # | Question | Correct Answer | Options |
|---|----------|----------------|---------|
| 1 | What is... | Option A | A, B, C, D |
| 2 | Select all... | Option A, C | A, B, C, D |
```

## Important Notes

- **NEVER use jq** in curl commands - even if installed, it can cause issues on some systems
- **Keep question text minimal** - students get full context from homework.md file
- **ALWAYS verify question count** before and after creating (API adds, never replaces)

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication token required` | Missing/invalid token | Check `AUTH_TOKEN` env var |
| `Course or homework not found` | Invalid slugs | Use course-content to find slugs first |
| `Invalid JSON` | Malformed JSON | Check JSON syntax |
