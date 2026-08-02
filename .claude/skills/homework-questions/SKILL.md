---
name: homework-questions
description: Manage homework questions via REST API
---

# Homework Questions API

## Overview

This skill provides commands to manage questions for homeworks via the REST API. Supports list, create, update, and guarded delete.

## Configuration

- **Production instance**: `https://courses.datatalks.club`
- **Dev instance**: `https://dev.courses.datatalks.club`
- **Auth token**: Available as `AUTH_TOKEN` environment variable, works for both dev and prod

## Workflow

0. **Find homework IDs**: Use `course-content` skill to list homeworks and get their **IDs** (the questions API uses homework IDs, not slugs)
1. **Read the homework file**: typically it's named homework.md
2. **Read solutions file**: Analyze the content of the homework folder. There could be the solutions.md file (or similar) with the answers
3. **Check existing questions**: ALWAYS verify current question count BEFORE creating
4. **Create questions**: POST questions with correct answers to the homework
5. **Verify creation**: ALWAYS check question count AFTER creating
6. **Open homework**: Use PATCH on the homework to set `"state": "OP"` (via course-content skill)
7. **Provide summary**: List all questions with their answers and include the homework link

## API Endpoints

```
GET    /api/courses/<course_slug>/homeworks/<homework_id>/questions/                - List questions
POST   /api/courses/<course_slug>/homeworks/<homework_id>/questions/                - Create question(s)
GET    /api/courses/<course_slug>/homeworks/<homework_id>/questions/<question_id>/  - Question detail
PATCH  /api/courses/<course_slug>/homeworks/<homework_id>/questions/<question_id>/  - Update question
DELETE /api/courses/<course_slug>/homeworks/<homework_id>/questions/<question_id>/  - Guarded delete
```

Full URL example:
- Production: `https://courses.datatalks.club/api/courses/ml-zoomcamp-2026/homeworks/123/questions/`
- Dev: `https://dev.courses.datatalks.club/api/courses/ml-zoomcamp-2026/homeworks/123/questions/`

**Important:** The endpoint uses homework **ID** (numeric), not slug. Get the ID from the course-content list first.

## Authentication

The `AUTH_TOKEN` environment variable is already set — just use it directly:

```bash
-H "Authorization: Token ${AUTH_TOKEN}"
```

## Generated API Specification

Before changing questions, fetch the generated OpenAPI spec from the target environment and treat it as the source of truth:

```bash
curl -s "https://courses.datatalks.club/api/openapi.json" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

The OpenAPI endpoint is token-protected. Use it for current routes, request bodies, responses, and delete safety rules. This skill file is workflow guidance.

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

### Critical Rule:

**ALWAYS verify the question count BEFORE and AFTER creating.** The API adds new questions every time - it never replaces or deduplicates existing questions.

## Step 1: Find Homework IDs

Use the course detail endpoint to list all homeworks with their IDs:

```bash
curl -s -X GET "https://courses.datatalks.club/api/courses/<course_slug>/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

This returns all homeworks with their **IDs**, slugs, titles, and states. Use the `id` field for question endpoints.

Question detail responses include `answers_count`, `can_delete`, and `delete_blockers`.

## Step 2: Read Solutions File

Read the solutions from the cohort directory to get correct answers:

```bash
# Example path
~/git/data-engineering-zoomcamp/cohorts/2026/01-docker-terraform/solution.md
```

**If no solution file exists** (e.g., homework says "Will be added after the due date"), create questions WITHOUT the `correct_answer` field. Students can still submit, but there will be no auto-grading.

**General rule**: Do NOT specify `answer_type` unless explicitly asked. It is optional and defaults to appropriate value.

## Step 3: List Existing Questions

```bash
curl -s -X GET "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/questions/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

**Response:**
```json
{
  "homework_id": 123,
  "homework_title": "Homework 1",
  "questions": [
    {
      "id": 1,
      "text": "What is 2+2?",
      "question_type": "MC",
      "answer_type": null,
      "possible_answers": ["3", "4", "5"],
      "correct_answer": "2",
      "scores_for_correct_answer": 1
    }
  ]
}
```

## Step 4: Create Questions

### IMPORTANT: Question Text Formatting

Do NOT add prefixes like "Question 1.", "Q1.", "Question 2." etc. to the question `text` field. Strip any numbering prefix and use only the title/question itself.

Example from homework file: `### Question 1. Bruin Pipeline Structure`
- **WRONG**: `"text": "Question 1. Bruin Pipeline Structure"`
- **CORRECT**: `"text": "Bruin Pipeline Structure"`

### Single Question

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/questions/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "What does SQL stand for?",
    "question_type": "MC",
    "possible_answers": ["Structured Query Language", "Simple Query Language", "Standard Query Language"],
    "correct_answer": "1",
    "scores_for_correct_answer": 1
  }'
```

### Bulk Create (Multiple Questions)

Send a JSON array:

```bash
curl -s -X POST "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/questions/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '[
    {
      "text": "What does SQL stand for?",
      "question_type": "MC",
      "possible_answers": ["Structured Query Language", "Simple Query Language"],
      "correct_answer": "1"
    },
    {
      "text": "Describe Paris",
      "question_type": "FL"
    }
  ]'
```

**Response (201):**
```json
{
  "created": [
    {"id": 1, "text": "What does SQL stand for?", "question_type": "MC", ...},
    {"id": 2, "text": "Describe Paris", "question_type": "FL", ...}
  ]
}
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

### MC with multiple accepted answers (radio buttons)

When a question has a range where two adjacent options are both acceptable
(e.g. timing measurements that straddle a boundary), keep `MC` (radio
buttons) and set `correct_answer` to a comma-separated list. The student
picks ONE option; any listed index is scored correct.

```bash
# Two acceptable answers, student selects any one
"question_type": "MC",
"correct_answer": "3,4"
```

This is different from `CB` (checkboxes): `CB` requires the student to
select ALL listed answers. Use `MC` with comma-separated indices when each
answer is independently sufficient.

## Updating Questions (PATCH)

```bash
curl -s -X PATCH "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<hw_id>/questions/<question_id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"correct_answer": "3", "scores_for_correct_answer": 2}'
```

### Patchable Fields

| Field | Description |
|-------|-------------|
| `text` | Question text |
| `question_type` | `MC`, `FF`, `FL`, or `CB` |
| `answer_type` | `ANY`, `FLT`, `INT`, `EXS`, or `CTS` |
| `possible_answers` | Array of options (converted to newline-delimited) |
| `correct_answer` | Correct answer value |
| `scores_for_correct_answer` | Points for correct answer |

## Deleting Questions

Questions can be deleted only when they have no answers. Do not delete questions from a homework that has collected submissions; deleting answered questions would delete submitted answer data, so the API rejects it.

```bash
curl -s -X DELETE "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<hw_id>/questions/<question_id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

## Step 5: Open Homework

After creating questions, open the homework using PATCH (via course-content skill):

```bash
curl -s -X PATCH "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/" \
  -H "Authorization: Token ${AUTH_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"state": "OP"}'
```

## Step 6: Verify and Provide Summary

After creating questions, verify and provide a summary:

```bash
# Verify questions were created
curl -s -X GET "https://courses.datatalks.club/api/courses/<course_slug>/homeworks/<id>/questions/" \
  -H "Authorization: Token ${AUTH_TOKEN}"
```

Then provide:

```
Created N questions for "<Homework Title>"

Homework link: https://courses.datatalks.club/<course_slug>/homework/<homework_slug>

| # | Question | Correct Answer | Options |
|---|----------|----------------|---------|
| 1 | What is... | Option A | A, B, C, D |
| 2 | Select all... | Option A, C | A, B, C, D |
```

## Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `text` | string | Yes (for create) | Question text |
| `question_type` | string | No | `MC` (default), `FF`, `FL`, or `CB` (only if explicitly requested) |
| `answer_type` | string | No | `ANY`, `FLT`, `INT`, `EXS`, or `CTS` |
| `possible_answers` | array | No | Array of answer options (for MC questions) |
| `correct_answer` | string | No | For MC: 1-based index (`"1"`), for CB: comma-separated (`"1,3"`) |
| `scores_for_correct_answer` | int | No | Points for correct answer (default: 1) |

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

## Important Notes

- **NEVER use jq** in curl commands - output raw JSON only
- **Keep question text minimal** - students get full context from homework.md file
- **ALWAYS verify question count** before and after creating (API adds, never replaces)
- **Use homework ID, not slug** for the questions endpoint
- **Open homework via PATCH** on the homework endpoint, not in the questions request
- **DELETE is guarded** - questions with existing answers cannot be deleted

## Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `Authentication token required` | Missing/invalid token | Check `AUTH_TOKEN` env var |
| `Course or homework not found` | Invalid slugs or ID | Use course detail to find IDs first |
| `Invalid JSON` | Malformed JSON | Check JSON syntax |
| `Cannot update field: X` | Invalid field in PATCH | Check patchable fields list |
| `text is required` | Missing text in create | Provide `text` field |
| `Cannot delete question with existing answers` | Question has submitted answers | Do not delete; update only if appropriate |
