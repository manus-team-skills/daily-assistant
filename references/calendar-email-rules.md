# Calendar and Email Triage Rules (Context-Aware Edition)

This document defines the pre-screening filters, context-aware relationship scoring matrix, cold email demotion rules, and the standard output template for the **Morning Digest (09:00 AM)**.

---

## Part 1: Email Pre-screening Filters

Before evaluating or categorizing any emails, apply these filters to exclude noise. Silently ignore and omit the following from the triage report:
- **Calendar Notifications**: Invites, updates, acceptances, declines, or reminders.
- **Access Requests**: Google Drive, Notion, Slack, Figma, or other file-sharing/access request emails.
- **Pure Automated Alerts**: System alerts, password resets, or standard subscription receipts.

---

## Part 2: Context-Aware Relationship Scoring Matrix

Do not classify emails solely based on subject lines or content urgency. Instead, evaluate the **sender's context** using the matrix below.

### Prior History Memory & Active Mail Search

> **CRITICAL MEMORY RULE (Solving "Session Amnesia")**: 
> Every execution run is a completely fresh agent session with no cross-session memory of prior interactions. You **MUST NOT** assume or guess the "Prior History" score. 
> 
> To accurately determine if there is a prior relationship, you **MUST** run an explicit search using the active mail connector (Gmail, Outlook, or Superhuman) for the sender's email address:
> 1. Extract the sender's email address (e.g., `sender@domain.com`).
> 2. Execute the appropriate search tool (e.g., `gmail_search_messages` or equivalent) with the query: `to:sender@domain.com OR from:sender@domain.com` (exclude the current email thread if possible).
> 3. Inspect the search results:
>    - **High Score (+2)**: If you find sent messages *from* the user or their team to this sender, or a multi-turn active thread in the past.
>    - **Medium Score (+1)**: If you find prior inbound emails from this sender but no outgoing replies from the user or their team.
>    - **Low/Zero Score (0)**: If there are absolutely no prior messages involving this email address (brand new contact).

### Relationship Scoring Criteria

| Factor | High Score (+2) | Medium Score (+1) | Low/Zero Score (0) |
| :--- | :--- | :--- | :--- |
| **Prior History** | Active thread; the user or their team has replied to this sender in the past (verified via explicit mail search). | 1-way inbound but sender has reached out before; no prior replies (verified via explicit mail search). | No prior email history; brand new contact. |
| **Company Size & Tier** | Large/well-known enterprise, tier-1 VC/investor, major press, or key strategic partner. | Mid-market company, growing startup, or secondary partner. | Small business, individual sender, or unknown domain. |
| **Relationship with User/Org** | Existing active customer, signed partner, or warm introduction. | Qualified lead, pipeline prospect, or prospective vendor. | Cold outreach, cold sales pitch, or unsolicited inquiry. |

### Classification Routing Rules

Evaluate the total context score and content indicators to route emails into one of four standard buckets:

```
Total Context Score = Prior History Score + Company Size Score + Relationship Score
```

#### 1. URGENT (Action Required within 1 Hour)
- **Condition**: Total Context Score is **4 or higher** **AND** the email content indicates immediate commercial or operational urgency (e.g., system downtime, active contract blocker, immediate demo request from tier-1 lead).
- **Action**: Place at the top of the Morning Digest. Draft a customized, warm, and concise reply. Suggest action `[Respond]`.

#### 2. ACTION (Action Required Today)
- **Condition**: Total Context Score is **2 or 3** **AND** the email requests a specific follow-up, meeting, or response today.
- **Action**: Draft a reply and suggest adding to the to-do list. Suggest action `[Respond]`.

#### 3. FYI (Read Later, No Reply Needed)
- **Condition**: Total Context Score is **1 or 2** **AND** the email is informational (e.g., project updates, CC'd threads where teammates are active, newsletters from strategic partners).
- **Action**: Summarize in 1 sentence. Suggest action `[Archive]` or `[Do Nothing]`.

#### 4. NOISE (Archive Immediately)
- **Condition**: Total Context Score is **0** (Cold Email) **OR** the email is a generic automated notification.
- **Action**: Summarize in 1 sentence. Suggest action `[Archive]`.

---

## Part 3: Explicit Rule Priority Chain (Resolving Rule Conflicts)

When an email triggers multiple rules that suggest different classifications (e.g., an email is both CC'd only and a cold sales email), you **MUST** resolve the conflict by applying the rules in the strict order of precedence defined below. **Lower-numbered rules always override higher-numbered rules.**

| Priority | Rule Name | Description / Conflict Resolution |
| :--- | :--- | :--- |
| **1 (Highest)** | **Pre-screening Filter** | If an email matches any filter in Part 1 (e.g., calendar invite, access request), it is **silently ignored** immediately. No other rules apply. |
| **2** | **Cold Email Demotion** | If an email is identified as cold outreach (Prior History Score = 0 AND Relationship = Cold), its classification **MUST** be capped at **NOISE** (or **FYI** if it is from a highly strategic company, Score = 1). It can **NEVER** be URGENT or ACTION, even if the user is in the `To:` field. |
| **3** | **Not Addressed to User** | If the user is only CC'd or BCC'd (not in the `To:` field), the classification **MUST** be capped at **FYI**. This applies even if the sender is a high-value contact (Context Score >= 4) or the content sounds highly urgent. |
| **4 (Lowest)** | **Standard Context Score Routing** | If none of the above override rules are triggered, classify the email strictly according to its **Total Context Score** and content urgency. |

### Conflict Examples & Resolutions:
- **Example A**: An email is a cold outreach (Priority 2) AND the user is CC'd (Priority 3). 
  - *Resolution*: Priority 2 wins. Cap at **NOISE** (or FYI if from a strategic company).
- **Example B**: An email is from a VIP partner (Context Score = 6, standard routing would be URGENT) BUT the user is only CC'd (Priority 3).
  - *Resolution*: Priority 3 wins. Cap at **FYI**.

---

## Part 4: Output Formatting

This document covers triage logic only. For content structure and channel-specific rendering syntax (Slack, email, in-chat), load **`references/output-templates.md`** when entering the composition step of any workflow.

- For to-do list schema and topic categorization, load **`references/todo-rules.md`** only when updating `todo.md`.
- **Lazy Loading Rule**: Do not load either reference file during pre-screening or scoring. Only load them at the generation step that requires them.
