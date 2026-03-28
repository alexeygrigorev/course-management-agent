# Course Management Agent

## Overview

This project enables interaction with the Course Management Platform to manage courses, homeworks, projects, and deployments.

## Platform Instances

- **Production**: `https://courses.datatalks.club`
- **Development**: `https://dev.courses.datatalks.club`
- **Repository**: `DataTalksClub/course-management-platform`

## Authentication

- `AUTH_TOKEN` environment variable (stored in `.env`)
- Works for both dev and prod instances

## Course Repositories

All course content is located in `~/git/`:

| Repository | Path | Purpose |
|------------|------|---------|
| data-engineering-zoomcamp | `~/git/data-engineering-zoomcamp/` | Data Engineering course |
| machine-learning-zoomcamp | `~/git/machine-learning-zoomcamp/` | ML course |
| mlops-zoomcamp | `~/git/mlops-zoomcamp/` | MLOps course |
| llm-zoomcamp | `~/git/llm-zoomcamp/` | LLM course |
| ai-dev-tools-zoomcamp | `~/git/ai-dev-tools-zoomcamp/` | AI Dev Tools course |


## Available Skills

- `course-content` - Full CRUD for courses, homeworks, and projects via `/api/` REST API
- `homework-questions` - Full CRUD for homework questions via `/api/` REST API
- `deploy-prod` - Trigger production deployment workflow

## Courses on Production

Active courses (accessed via `https://courses.datatalks.club/<course_slug>/`):

| Course Slug | Full Name | Year |
|-------------|-----------|------|
| de-zoomcamp-2026 | Data Engineering Zoomcamp | 2026 |
| de-zoomcamp-2025 | Data Engineering Zoomcamp | 2025 |
| de-zoomcamp-2024 | Data Engineering Zoomcamp | 2024 |
| ml-zoomcamp-2025 | Machine Learning Zoomcamp | 2025 |
| ml-zoomcamp-2024 | Machine Learning Zoomcamp | 2024 |
| llm-zoomcamp-2025 | LLM Zoomcamp | 2025 |
| llm-zoomcamp-2024 | LLM Zoomcamp | 2024 |
| mlops-zoomcamp-2025 | MLOps Zoomcamp | 2025 |
| mlops-zoomcamp-2024 | MLOps Zoomcamp | 2024 |
| sma-zoomcamp-2025 | SMA Zoomcamp | 2025 |
| sma-zoomcamp-2024 | SMA Zoomcamp | 2024 |
| ai-dev-tools-2025 | AI Dev Tools | 2025 |

## Slug Patterns

### Course Slugs

Format: `<course_short>-zoomcamp-<year>`

| Course | Short Name |
|--------|------------|
| Data Engineering | `de-zoomcamp` |
| Machine Learning | `ml-zoomcamp` |
| MLOps | `mlops-zoomcamp` |
| LLM | `llm-zoomcamp` |
| AI Dev Tools | `ai-dev-tools` |
| SMA | `sma-zoomcamp` |

### Homework Slugs

Format: Usually `hw1`, `hw2`, `hw01`, `hw02`, etc.

Examples:
- `hw1`, `hw2`, `hw3` ... `hw10`
- Special: `dlt` (workshop), `agents`

### Project Slugs

Format: `project1`, `project2`, etc. or descriptive names

Examples:
- `project1`, `project2`, `project3`
- `midterm`
- `capstone1`, `capstone2`

### States

| Code | Name |
|------|------|
| `CL` | Closed |
| `OP` | Open |
| `SC` | Scored (closed) |
| `CO` | Completed |
| `PR` | Peer Review |
| `CS` | Capstone Submission |
