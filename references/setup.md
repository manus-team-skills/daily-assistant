# Daily Assistant Setup (First Run Only)

These steps only need to run once. They are gated by flags in `/home/ubuntu/daily_assistant_config.json`. At the start of the very first run, check this file (initialize as `{}` if it does not exist) and skip any step whose flag is already set.

On subsequent runs within the same task, setup is already complete — skip directly to the relevant workflow.

---

## Step 1: Enable Required Connectors
**Skip if** `config["connectors_confirmed"] == true`.

This skill requires three connectors. Prompt the user to enable any that are missing:

| Connector | Purpose | How to enable |
| :--- | :--- | :--- |
| **Gmail** / **Outlook Mail** / **Superhuman Mail** | Fetch unread emails, send Morning/Evening digests | Settings → Connectors → Gmail / Outlook Mail / Superhuman Mail |
| **Google Calendar** / **Outlook Calendar** | Fetch today's events and identify deep-work windows | Settings → Connectors → Google Calendar / Outlook Calendar |
| **Slack** | Send Morning/Evening digests as Slack DMs | Settings → Connectors → Slack |

Once all three are confirmed, write `"connectors_confirmed": true` to the config file before proceeding.

---

## Step 2: Identify the User
**Skip if** `config["user_name"]`, `config["user_email"]`, and `config["user_slack_id"]` are all already set.

Determine:
1. **User's name** — used in greetings (e.g., "Hey Sarah,")
2. **User's email address** — used as the send-to address for emails, and to determine whether an email is addressed directly to the user (To: field) vs. CC'd only
3. **User's Slack user ID** — used as the `channel_id` for Slack DMs. Find it via `slack_read_user_profile` (defaults to current user) or `slack_search_users`.

Ask the user for any values not already in the config. Once confirmed, write all three to the config file. They are now available in context for the rest of this task.

---

## Step 3: Create Scheduled Tasks
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
