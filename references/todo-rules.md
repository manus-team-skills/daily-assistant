# To-Do List Schema and Maintenance Rules (Topic-Based Edition)

The to-do list is maintained in a single file at `/home/ubuntu/todo.md`. This document defines the file schema, category detection rules, urgency sorting, 4D intake filter, and archiving rules.

---

## Part 1: The 4D Intake Filter

Before adding any task, evaluate it against the **4D Framework**:

1. **Do**: If the task takes less than 2 minutes (e.g., a 1-sentence Slack reply), do it immediately — don't add it to the list.
2. **Defer**: If it requires focused work or has a future deadline, add it to the appropriate topic section.
3. **Delegate**: If it belongs to another team member, assign it and add only a lightweight follow-up reminder (e.g., "Check with Alex on X by Friday").
4. **Delete**: If it's low-impact, redundant, or has been in the backlog for 3+ months with no action, archive it permanently.

---

## Part 2: To-Do List File Schema (Topic-Based)

The `/home/ubuntu/todo.md` file MUST follow this structure:

```markdown
# [User Name]'s To-Do List

Last Updated: [YYYY-MM-DD HH:MM]

---

## 🎯 Most Important Tasks
- [ ] **[Task]** — [one-line reason]
- [ ] **[Task]** — [one-line reason]
- [ ] **[Task]** — [one-line reason]

---

## [Emoji] [Category Name]
- [ ] **[Task]** — [context / deadline] *(🔥 urgent)*
- [ ] **[Task]** — [context / deadline]

## [Emoji] [Category Name]
...

---

## 👀 FYI — No Action Needed
- **[Topic]** — [1-line status. Who is handling it.]
- **[Topic]** — [1-line status.]
```

---

## Part 3: Category Detection Rules

Categories are **auto-detected from context** each time the list is updated. Use different categories on different days based on what's actually in the list. Do not force a fixed set of categories.

**How to detect categories:**
- Read all pending tasks and group them by the project, relationship, or domain they belong to
- Assign a short, descriptive category name and a relevant emoji
- Aim for 3–6 categories — enough to organize, not so many it fragments the list
- If a task doesn't fit neatly, use a catch-all like **⚙️ Internal / Ops** or **📥 Inbox**

**Category rules:**
- **Enterprise Sales** (or equivalent revenue-facing category) covers all customer-facing, reseller, and business partner activity — including partner pipeline, reseller contracts, partner enablement, and joint go-to-market deals. Do not create a separate "Partnerships" category; fold it into Enterprise Sales.
- All other categories are auto-detected from context
- Common categories: 📦 Product / Internal, 🎯 Marketing, ⚙️ Ops / Admin, 🌱 Personal

**If the user specifies categories**, use exactly those and do not auto-detect.

**Most Important Tasks duplication rule:** Tasks listed under Most Important Tasks MUST NOT be repeated under their topic category. The MIT section is the canonical entry for those tasks while they are active.

---

## Part 4: Urgency Sorting Within Each Category

Within each category, sort tasks by urgency — most urgent first:

1. **🔥 Urgent** — Deadline today or tomorrow; tag with `*(🔥 urgent)*`
2. **📅 This week** — Deadline within 7 days; no tag needed
3. **🗓️ This month** — Flexible deadline; no tag needed
4. **💤 Someday** — No deadline; place at the bottom of the category

Do not use separate sections for urgency — urgency is expressed through sort order and the `*(🔥 urgent)*` tag on the most time-sensitive items.

## Part 4b: FYI — No Action Needed Section

The **👀 FYI — No Action Needed** section appears at the bottom of the active list. It holds items the user should be aware of but does not need to act on:
- Items being monitored (e.g., waiting on a response from a third party)
- Tasks fully owned by another team member where the user only needs visibility
- Threads or updates that are informational only

Each FYI item is one line: **[Topic]** — [1-line status. Who is handling it.]

---

## Part 5: Maintenance Rules

0. **Reconcile Before Every Run (Step 0 — always first)**:
   - Read the full `todo.md` before making any changes
   - Find all lines in active sections (everything above the FYI section) where the checkbox is `- [x]`
   - For each one: append it to `/home/ubuntu/todo_archive/todo_completed_YYYY_MM.md` as `- [x] ~~**[Task]**~~ — Completed on [today's date]`, then remove it from `todo.md`
   - This reconciles any manual checkbox clicks made in the Manus UI between runs

1. **Daily Update (Morning Digest)**:
   - Re-sort all categories by urgency
   - Select the top 3 tasks across all categories to promote to the Most Important Tasks section
   - Remove those tasks from their topic category (no duplication)
   - Move completed tasks to `/home/ubuntu/todo_archive/todo_completed_YYYY_MM.md` immediately

2. **Completing Tasks**:
   - Remove the completed task from the active list
   - Append it to `/home/ubuntu/todo_archive/todo_completed_YYYY_MM.md` as: `- [x] ~~**[Task Name]**~~ — Completed on [YYYY-MM-DD]`
   - Create the archive file if it does not exist

3. **No Completed & Archived section in todo.md**:
   - The active to-do file does NOT have a Completed & Archived section
   - All completed items go directly to the monthly archive file

4. **FYI section maintenance**:
   - When a monitored item resolves or the other person completes it, remove it from FYI
   - If a FYI item becomes actionable (e.g., no response after 3+ days), promote it to the relevant topic category with a `*(🔥 urgent)*` tag
