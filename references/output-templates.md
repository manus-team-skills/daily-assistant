# Daily Assistant Output Templates

This file defines the content structure and channel-rendering rules for all Daily Assistant outputs. There is one canonical content structure per digest type; rendering syntax is adapted per channel at generation time.

---

## Rendering Syntax by Channel

Before generating any output, identify the target channel and apply the corresponding syntax rules throughout:

| Element | Slack | Email (plain text) | In-Chat (Markdown) |
| :--- | :--- | :--- | :--- |
| **Title / Subject** | First line, bold: `*Title*` | Separate `Subject:` field | `# Title` |
| **Section headers** | Bold line: `*HEADER*` | Divider: `━━━ HEADER ━━━` | `## Header` |
| **Bold** | `*text*` | Plain text (no markup) | `**text**` |
| **Italic** | `_text_` | Plain text (no markup) | `*text*` |
| **Bullets** | `•` | `•` | `-` |
| **Numbered list** | `1.` | `1.` | `1.` |
| **Hyperlinks** | `<url\|label>` | Plain URL on its own line | `[label](url)` |
| **Section spacing** | One blank line between sections | One blank line between dividers | One blank line between sections |
| **Within-section spacing** | No blank lines | No blank lines | No blank lines |
| **Draft replies** | Include as `> Draft: …` | Omit entirely | Include as `> Draft: …` |
| **Character limit** | 5000 per message; split if needed | No limit | No limit |

---

## Morning Digest

### Content Structure

Render the following sections in order. Skip any section that has no content (e.g., no URGENT emails → omit that block entirely).

**1. Header**
Greeting + date. One line.
> Example: "Here's your morning rundown for Wednesday, May 27."

**2. Most Important Tasks**
Exactly 3 tasks, each with a one-line reason. Drawn from the top of `todo.md`.

**3. Calendar**
Chronological list of today's events. One line per event. Note any deep-work windows (2+ uninterrupted hours). Omit all-day events unless OOO or Travel.

**4. Emails**
Grouped into priority tiers (apply triage rules from `calendar-email-rules.md`):
- **URGENT** — items requiring action within the hour. Include sender, subject, 1-sentence summary, and a source link.
- **ACTION** — items requiring action today. Same format as URGENT.
- **FYI** — informational items. One compact line each: sender + summary.
- Omit NOISE entirely. Never expose triage reasoning, scores, or classification labels.

**5. Footer**
Warm one-line sign-off.

---

## Evening Follow-up

### Content Structure

**1. Header**
Date + brief intro. One line.
> Example: "Here's what came out of today's meetings."

**2. New Action Items**
Grouped by source meeting. For each meeting: meeting name + source link, then one bullet per action item (action verb + who + deadline if known). If no meeting notes were found, say so in one line — do not skip the section.

**3. Updated To-Do List**
Full contents of `todo.md`, organized by topic category. Omit the "FYI — No Action Needed" section to keep it clean.

**4. Footer**
Warm one-line sign-off.

---

## Delivery Order

Always send in this order to ensure the user receives the digest even if they are not in the chat:
1. Slack DM → `slack_send_message` with `channel_id` from `daily_assistant_config.json`
2. Email → active mail connector (e.g., `gmail_send_messages` or equivalent) to `user_email` from `daily_assistant_config.json`
3. In-chat → render in standard Markdown
