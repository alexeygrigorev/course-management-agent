# Template: project is ready for peer review

Post this to the course Telegram channel right after peer review is triggered
(`POST /api/courses/<course_slug>/projects/by-slug/<project_slug>/assign-reviews/`
returned `SUCCEEDED` and the project state is `PR`).

Based on Alexey's past announcements:

- https://t.me/mlzoomcamp/183 ("Time to evaluate your peers!")
- https://t.me/mlzoomcamp/235 ("We closed the form for submitting capstone 1")
- https://t.me/mlzoomcamp/238 ("Now it's time to learn from your peers!")
- https://t.me/mlzoomcamp/240 and https://t.me/mlzoomcamp/244 (reviewing three peers is mandatory)

## Rules

- Plain text, no semicolons.
- Peer review lasts exactly one week from when reviews are assigned. assign-reviews
  resets `peer_review_due_date` to assignment time + 7 days.
- Deadline: ALWAYS read `peer_review_due_date` via
  `GET /api/courses/<course_slug>/projects/by-slug/<project_slug>/` AFTER assign-reviews
  has run. Never reuse a date from earlier posts, the project config or the submission deadline.
- Format: `Weekday, Month D, HH:MM CET`, converted to Europe/Berlin
  (e.g. `2026-11-05T06:00:00Z` -> `Thursday, November 5, 07:00 CET`).
- `{eval_link}` = `https://courses.datatalks.club/{course_slug}/project/{project_slug}/eval`
- `{peers}` = `number_of_peers_to_evaluate` from the project.

## Template

```text
Now it's time to learn from your peers!

We closed the form for submitting {project}. If you submitted it, you will find your peer review assignments here:

{eval_link}

Review deadline: {review_deadline}

Remember that reviewing {peers} peers is a mandatory step. If you don't complete it, you will not pass the project.

Have fun and happy learning!
```
