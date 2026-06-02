# Meeting Follow-up & Action Item Extraction

This document defines where to scan for meeting notes, how to extract action items, and the standard output template for the **Evening Follow-up (06:00 PM)**.

---

## Part 1: Meeting Notes Scan Locations

At 6:00 PM, search for notes updated or created today in these locations:
1. **Granola**: Use `list_meetings` then `get_meetings` MCP calls to fetch notes for meetings held today. This is the primary source.
2. **Notion**: Search Notion databases or pages for entries modified today that match keywords like `Meeting`, `Sync`, `Call`, `Notes`, `Sync-up`, or `Minutes`.
3. **Local Workspace**: Scan `/home/ubuntu/workspace/` and `/home/ubuntu/notes/` for files with today's date in the filename or metadata modified within the last 12 hours.
4. **Google Drive**: Check Google Docs or Sheets modified today with meeting-related titles.

---

## Part 2: Action Item Extraction Criteria

When scanning notes, look for indicators of commitments and tasks.
- **Direct Assignments**: Lines starting with the user's name, `@[User]`, `[User]`, or `Action: [User]`.
- **Implicit Commitments**: Verbs indicating action by the user or their team, such as *"We will send the proposal by tomorrow"*, *"[User] to follow up with..."*, or *"Need to schedule a follow-up with..."*.
- **To-Do List Formulation**: Standardize all extracted tasks to start with an action-oriented verb, clearly stating:
  - **What** needs to be done.
  - **Who** is the counterparty or recipient.
  - **When** is the deadline (if mentioned).
  - **Where** to find context (e.g., "per Sync with [Company]").


