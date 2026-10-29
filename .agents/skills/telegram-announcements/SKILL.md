---
name: telegram-announcements
description: Draft or publish DataTalksClub course announcements in Telegram using the local course-channel bot. Use for course updates, homework releases, deadline changes, and reminders; a request for a draft alone does not authorize posting.
---

# Telegram announcements

Use this skill for course Telegram channels, not for personal bot messages. The course-channel bot is in `~/bots/au-tomator-telegram-bot/`. Its `.env` contains `TELEGRAM_TOKEN`, and `main.py` maps Telegram chat IDs to course channels and public links. Read the token inside the sending process; never print it, put it in a command argument, or copy it into this repository. The `zapier-telegram-bot/send.sh` workflow also sends to Zapier and is not the course announcement path.

## Draft

Check announcement facts against the relevant course repository or live course platform, especially links and deadlines. Match the user's requested tone and formatting. If the user asks only for a draft, return the message without sending it.

## Publish

1. Confirm that the user has asked to post and identify the intended course channel. Use the mapping in `~/bots/au-tomator-telegram-bot/main.py`; if the destination is ambiguous, clarify it before sending.
2. Check the bot with Telegram `getMe`, `getChat`, and `getChatMember`. Verify the channel title and that the bot can post there.
3. Send the final text once with Telegram Bot API `sendMessage` to the mapped chat ID. Use a plain-text message unless the user requests formatting. A Python process using `urllib.request` can load `TELEGRAM_TOKEN` from `.env` and make the POST without exposing it in shell arguments.
4. If the API confirms success, capture `message_id` and return `https://t.me/<channel_username>/<message_id>`.
5. If the request times out or its result is uncertain, check the public channel page at `https://t.me/s/<channel_username>` for the exact message before retrying. A timeout can happen after Telegram has already posted. If the channel cannot be checked and success is uncertain, report the uncertainty instead of risking a duplicate post.

Do not change course deadlines, repository files, or other external systems merely because an announcement mentions them; perform those actions only when the user has requested them separately.

## Templates

- `templates/peer-review-ready.md`: project is ready for peer review (post after assign-reviews succeeds).
