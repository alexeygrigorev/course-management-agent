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

## Notes

- The generated OpenAPI spec from the target environment is the source of truth before changing course content.
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

<<<<<<< Updated upstream

## Peer Review Assignment Notification (send exactly once)

After `POST /api/courses/<course>/projects/by-slug/<project>/assign-reviews/` succeeds
(project state becomes PR), students must get the "peer reviews assigned" email.
assign-reviews does NOT send it automatically.

1. Check the sent-marker first:
   `GET /api/datamailer/send-audits?idempotency_key=peer-review-assignment:<course>:<project>`
   - An audit with status "succeeded" (enqueued_count > 0) means it was already sent. Do NOT send again.
   - No audit, or only "failed" ones, means it is not sent yet and may be sent.
2. Send: there is no API endpoint yet (requested from Dev). Until then, a staff member clicks "Notify peer reviews" in cadmin
   (`POST /cadmin/<course>/project/<project>/notify-peer-reviews`). If the agent can't do it, ask Dev.
3. Re-check send-audits and confirm a "succeeded" entry whose enqueued_count matches the number of submitters.
4. Never resend. The send key is fixed per project. A deliberate resend is allowed only if the marker is intentionally
   reset (a platform change by Dev with Alexey's approval). Never work around it.
5. Mention the email in the Telegram announcement only after step 3 confirms it went out.
=======
## Peer-Review-Assigned Email (Send Once)

The peer-review-assigned email can be sent with a staff token instead of the
cadmin "Notify peer reviews" button. Both use the same idempotency key,
`peer-review-assignment:<course>:<project>`.

- Check first, read-only (never sends):
  `GET /api/courses/<course>/projects/by-slug/<project>/notify-peer-reviews/`
  returns `sent` and the send `audit` (`status`, `enqueued_count`,
  timestamps), or `audit: null` when nothing was sent.
- Send:
  `POST /api/courses/<course>/projects/by-slug/<project>/notify-peer-reviews/`.
  It needs a staff token and the project in `PR` state. It returns `200` with
  the succeeded audit.
- It is send-once. A second POST, or a POST after the cadmin button already
  sent, returns `409 peer_review_notification_already_sent` with the existing
  audit and sends nothing. `502` means the send failed; a retry is allowed.
- Resending after a success needs a deliberate server-side reset:
  `manage.py reset_peer_review_notification_marker <course> <project> --confirm`.
- The GET audit includes `skipped_recipients`: `user_id`, `email`, and a
  machine-readable `reason` (for example `category_unsubscribe` when the
  learner turned off `submission-results` emails, or `hard_bounce`).
- A POST emails every project submitter. Only send on production when the
  user explicitly asks for that course and project. Use GET to verify.

```console
$ curl -s "https://courses.datatalks.club/api/courses/<course>/projects/by-slug/<project>/notify-peer-reviews/" \
    -H "Authorization: Token ${AUTH_TOKEN}"
```

## Send Audits and Skipped Recipients

Staff-token, read-only endpoints for "why did this learner not get the
email?":

- `GET /api/datamailer/send-audits?idempotency_key=<key>` (also `email`,
  `template_key`, `status`, `send_type`) lists audits with their `id`.
- `GET /api/datamailer/send-audits/<id>` returns one audit with
  `skipped_recipients` (`user_id`, `email`, `reason`, `source_object_key`,
  `relay_message_id`, `relay_contact_id`).
- `skipped_recipients_source` says where the details came from: `audit`
  (stored from Relay's send response), `relay_lookup` (older audit, read from
  Relay without writing anything), `not_recorded` (list view of an older
  audit; open the detail endpoint), or `unavailable`.
- Never re-send to "fix" a skip. `category_unsubscribe`, `global_unsubscribe`,
  `hard_bounce` and `complaint` are the learner's or provider's decision.
>>>>>>> Stashed changes
