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
- Deadlines: weekday + date + time + CET, taken from the project's `peer_review_due_date`
  converted to Europe/Berlin. If it falls at 00:00, also say "end of <previous weekday>".
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
