---
name: daily-assistant
description: Daily assistant workflows for any user. Handles morning calendar overviews, context-aware email triage (using sender relationship, company size, and prior history), evening meeting follow-up summaries, and topic-based to-do list maintenance. Use when setting up scheduled updates, generating daily summaries, or updating a user's action items.
---

# Daily Assistant

This skill enables a comprehensive daily assistant workflow. It integrates morning planning, context-aware email triage, evening follow-up extraction, and active to-do list maintenance using automated schedules.

---

## Setup (First Run Only)

These steps only need to run once. They are gated by flags in `/home/ubuntu/daily_assistant_config.json`. At the start of the very first run, check this file (initialize as `{}` if it does not exist) and skip any step whose flag is already set.

On subsequent runs within the same task, setup is already complete — skip directly to the relevant workflow.

### Step 1: Enable Required Connectors
**Skip if** `config["connectors_confirmed"] == true`.

This skill requires three connectors. Prompt the user to enable any that are missing:

| Connector | Purpose | How to enable |
| :--- | :--- | :--- |
| **Gmail** / **Outlook Mail** / **Superhuman Mail** | Fetch unread emails, send Morning/Evening digests | Settings → Connectors → Gmail / Outlook Mail / Superhuman Mail |
| **Google Calendar** / **Outlook Calendar** | Fetch today's events and identify deep-work windows | Settings → Connectors → Google Calendar / Outlook Calendar |
| **Slack** | Send Morning/Evening digests as Slack DMs | Settings → Connectors → Slack |

Once all three are confirmed, write `"connectors_confirmed": true` to the config file before proceeding.

### Step 2: Identify the User
**Skip if** `config["user_name"]`, `config["user_email"]`, and `config["user_slack_id"]` are all already set.

Determine:
1. **User's name** — used in greetings (e.g., "Hey Sarah,")
2. **User's email address** — used as the send-to address for emails, and to determine whether an email is addressed directly to the user (To: field) vs. CC'd only
3. **User's Slack user ID** — used as the `channel_id` for Slack DMs. Find it via `slack_read_user_profile` (defaults to current user) or `slack_search_users`.

Ask the user for any values not already in the config. Once confirmed, write all three to the config file. They are now available in context for the rest of this task.

### Step 3: Create Scheduled Tasks
**Skip if** `config["schedules_created"] == true`.

Prompt the user:

> "To automate your daily digests, I'll set up two recurring tasks — one for your Morning Digest at 9 AM and one for your Evening Follow-up at 6 PM (Mon–Fri). Shall I create them now?"

If the user confirms, run the following two commands **sequentially**:

**Morning Digest (Mon–Fri at 09:00 AM):**
```bash
manus-config schedule create --title "Morning Digest" --cron "0 0 9 * * 1-5" --repeated --detail "Run the Morning Digest routine: (1) Reconcile /home/ubuntu/todo.md. (2) Fetch today's calendar. (3) Triage unread emails per calendar-email-rules.md. (4) Update todo.md. (5) Send Morning Digest via Slack DM, email, and in-chat."
```

**Evening Follow-up (Mon–Fri at 06:00 PM):**
```bash
manus-config schedule create --title "Evening Follow-up" --cron "0 0 18 * * 1-5" --repeated --detail "Run the Evening Follow-up routine: (1) Reconcile /home/ubuntu/todo.md. (2) Scan today's Granola meeting notes. (3) Extract action items. (4) Update todo.md. (5) Send Evening Follow-up via Slack DM, email, and in-chat."
```

After both succeed, write `"schedules_created": true` to the config file and confirm:

> "Done! Your Morning Digest will run every weekday at 9 AM and your Evening Follow-up at 6 PM. You can pause or edit them anytime in your Manus task settings."

If the user declines, write `"schedules_created": true` anyway (to suppress re-prompting on future runs) and proceed to the relevant workflow.

---

## Delivery

All outputs MUST be delivered in this order:
1. **Slack DM** — send to the user's Slack user ID via `slack_send_message`
2. **Email** — send to the user's email address via the active mail connector (e.g., `gmail_send_messages`, `outlook_send_messages`, or `superhuman_send_messages`)
3. **In-chat** — render in standard Markdown

For content structure and channel rendering syntax, load **`references/output-templates.md`** at the composition step only — not at the start of a run.

- **Tone**: Conversational, warm, easy to skim. Keep sections brief.
- **Omission**: Omit NOISE emails entirely — do not mention them, not even a count.

---

## Core Capabilities & Schedule Mapping

The Daily Assistant maintains a unified, persistent to-do list at `/home/ubuntu/todo.md` as its single source of truth.

| Time (Local) | Routine |
| :--- | :--- |
| **09:00 AM Mon–Fri** | **Morning Digest** |
| **06:00 PM Mon–Fri** | **Evening Follow-up** |
| **Ad-hoc** | **To-Do Maintenance** |

---

## Workflow A: Morning Digest (09:00 AM)

### Step 0: Reconcile todo.md
Read `/home/ubuntu/todo.md` and archive any `- [x]` items found in active sections to `/home/ubuntu/todo_archive/todo_completed_YYYY_MM.md`, then remove them from `todo.md`. This honours manual checkbox clicks before the digest runs.

### Step 1: Fetch Calendar
Use the appropriate calendar connector (e.g., `google-calendar` MCP or `outlook-calendar` MCP) to list today's events.
- Filter out all-day events unless "OOO" or "Travel"
- Identify deep-work windows (2+ uninterrupted hours)
- Note back-to-back conflicts

### Step 2: Triage Emails
Use the active mail connector (e.g., `gmail` MCP, `outlook-mail` MCP, or `superhuman` MCP) to fetch unread emails. Apply rules from **`references/calendar-email-rules.md`**:
- **Explicit Mail Search**: For each sender, run an explicit search via the active mail connector (e.g., `gmail_search_messages` or equivalent search tools) with `to:sender@domain.com OR from:sender@domain.com` to determine Prior History score. This is mandatory — do not rely on recall alone.
- **Pre-screen**: skip calendar notifications, access requests, automated alerts.
- **Rule Precedence**: Apply the Rule Priority Chain strictly (Pre-screening > Cold Email Demotion > Not Addressed to User > Standard Scoring).
- **Never expose triage reasoning** — no context scores, no "addressed to X", no classification labels.

### Step 3: Update To-Do List
Read `/home/ubuntu/todo.md`, add any urgent email action items, then re-sort using the topic-based schema in `references/todo-rules.md`. Write the updated file back to disk.

### Step 4: Compose and Send Morning Digest
Load **`references/output-templates.md`**. Render the Morning Digest content structure once, then apply the channel syntax rules for each target:
1. **Slack** — render with Slack syntax. Send via `slack_send_message`.
2. **Email** — render with email syntax. Send via the active mail connector (e.g., `gmail_send_messages`, `outlook_send_messages`, or `superhuman_send_messages`).
3. **In-Chat** — render with Markdown syntax and display in-chat.

---

## Workflow B: Evening Follow-up (06:00 PM)

### Step 0: Reconcile todo.md
Read `/home/ubuntu/todo.md` and archive any `- [x]` items in active sections before adding new ones.

### Step 1: Scan Meeting Notes
Use `granola` MCP (`list_meetings` then `get_meetings`) to fetch notes for meetings held today.
- If no notes found, skip gracefully — do not prompt the user

### Step 2: Extract Action Items
Apply extraction logic from **`references/meeting-followup.md`**:
- Direct assignments to the user, implicit commitments, follow-up items.
- Formulate each as a clear action verb + who + when.

### Step 3: Update To-Do List
Append new items to `/home/ubuntu/todo.md` under the correct topic category. Re-sort within each category by urgency. Write the updated file back to disk.

### Step 4: Compose and Send Evening Follow-up
Load **`references/output-templates.md`**. Render the Evening Follow-up content structure once, then apply the channel syntax rules for each target:
1. **Slack** — render with Slack syntax. Send via `slack_send_message`.
2. **Email** — render with email syntax. Send via the active mail connector (e.g., `gmail_send_messages`, `outlook_send_messages`, or `superhuman_send_messages`).
3. **In-Chat** — render with Markdown syntax and display in-chat.

---

## Workflow C: To-Do List Maintenance (Ad-hoc)

The to-do list is maintained at `/home/ubuntu/todo.md` using the schema in **`references/todo-rules.md`**.

Before any update, reconcile `todo.md` first: archive any `- [x]` items in active sections, then make the requested changes.

Update the list when:
- An urgent email action item surfaces during Morning Digest
- New action items are extracted during Evening Follow-up
- The user explicitly requests an update

---

## Related Resources (Lazy Loaded)

- **[references/calendar-email-rules.md](references/calendar-email-rules.md)** — Context-aware scoring matrix, explicit Gmail search rules, and rule priority chain. Load during email triage (Workflow A, Step 2).
- **[references/meeting-followup.md](references/meeting-followup.md)** — Meeting note scanning and action item extraction. Load during Workflow B, Step 2.
- **[references/todo-rules.md](references/todo-rules.md)** — Topic-based to-do schema, urgency sorting, 4D intake filter, and archiving rules. Load when updating `todo.md`.
- **[references/output-templates.md](references/output-templates.md)** — Unified content structures for Morning Digest and Evening Follow-up, plus channel rendering syntax (Slack, email, in-chat). Load at Step 4 of either workflow.
