---
name: daily-assistant
description: Daily assistant workflows for any user. Handles morning calendar overviews, context-aware email triage (using sender relationship, company size, and prior history), evening meeting follow-up summaries, and topic-based to-do list maintenance. Use when setting up scheduled updates, generating daily summaries, or updating a user's action items.
---

# Daily Assistant

This skill enables a comprehensive daily assistant workflow. It integrates morning planning, context-aware email triage, evening follow-up extraction, and active to-do list maintenance using automated schedules.

---

## First Run

On the very first run, load **`references/setup.md`** and complete the setup steps (connector verification, user identity, and schedule creation). Setup state is persisted in `/home/ubuntu/daily_assistant_config.json` — once all flags are set, skip setup on all subsequent runs.

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
Use the active mail connector (e.g., `gmail` MCP, `outlook-mail` MCP, or `superhuman` MCP) to fetch unread emails. Load and apply rules from **`references/calendar-email-rules.md`**:
- **Explicit Mail Search**: For each sender, run an explicit search to determine Prior History score. This is mandatory — do not rely on recall alone.
- **Pre-screen**: skip calendar notifications, access requests, automated alerts.
- **Rule Precedence**: Apply the Rule Priority Chain strictly (Pre-screening > Cold Email Demotion > Not Addressed to User > Standard Scoring).
- **Never expose triage reasoning** — no context scores, no "addressed to X", no classification labels.

### Step 3: Update To-Do List
Load **`references/todo-rules.md`**. Read `/home/ubuntu/todo.md`, add any urgent email action items, then re-sort using the topic-based schema. Write the updated file back to disk.

### Step 4: Compose and Send Morning Digest
Load **`references/output-templates.md`**. Render the Morning Digest content structure once, then apply the channel syntax rules for each target:
1. **Slack** — render with Slack syntax. Send via `slack_send_message`.
2. **Email** — render with email syntax. Send via the active mail connector.
3. **In-Chat** — render with Markdown syntax and display in-chat.

---

## Workflow B: Evening Follow-up (06:00 PM)

### Step 0: Reconcile todo.md
Read `/home/ubuntu/todo.md` and archive any `- [x]` items in active sections before adding new ones.

### Step 1: Scan Meeting Notes
Use `granola` MCP (`list_meetings` then `get_meetings`) to fetch notes for meetings held today.
- If no notes found, skip gracefully — do not prompt the user

### Step 2: Extract Action Items
Load and apply extraction logic from **`references/meeting-followup.md`**:
- Direct assignments to the user, implicit commitments, follow-up items.
- Formulate each as a clear action verb + who + when.

### Step 3: Update To-Do List
Load **`references/todo-rules.md`** if not already in context. Append new items to `/home/ubuntu/todo.md` under the correct topic category. Re-sort within each category by urgency. Write the updated file back to disk.

### Step 4: Compose and Send Evening Follow-up
Load **`references/output-templates.md`**. Render the Evening Follow-up content structure once, then apply the channel syntax rules for each target:
1. **Slack** — render with Slack syntax. Send via `slack_send_message`.
2. **Email** — render with email syntax. Send via the active mail connector.
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

| Resource | Load When |
| :--- | :--- |
| **[references/setup.md](references/setup.md)** | First run only (check `daily_assistant_config.json` flags) |
| **[references/calendar-email-rules.md](references/calendar-email-rules.md)** | Workflow A, Step 2 (email triage) |
| **[references/meeting-followup.md](references/meeting-followup.md)** | Workflow B, Step 2 (action item extraction) |
| **[references/todo-rules.md](references/todo-rules.md)** | Any time `todo.md` is being updated |
| **[references/output-templates.md](references/output-templates.md)** | Step 4 of either workflow (composition) |
