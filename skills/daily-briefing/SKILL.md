---
name: daily-briefing
description: Generate Xiulin's morning research briefing across configurable topics, save to Apple Notes with a date-prefixed title. Use when user says "daily briefing", "morning brief", "morning update", "what's new today", "today's briefing", "brief me", "run my briefing", "daily update", or any similar request for a recurring research summary.
---

# Daily Briefing

## When To Use
- User asks for "daily briefing", "morning brief", or "what's new today"
- User says "brief me" or "run my briefing"
- Start of a new day when user wants a research roundup
- User wants to update or change which topics get researched

## Core Principle
**Topics live in Apple Notes, not in the skill.** The skill reads a single editable note called `Daily Briefing Topics` so the user can change focus areas anytime without touching code or asking Claude to "remember" preferences.

## The Process

### Phase 1: Read Topics
1. Use `Read and Write Apple Notes:get_note_content` with `note_name: "Daily Briefing Topics"`
2. If the note doesn't exist, create it with the default template (see Setup section below) and tell the user
3. Parse the numbered list of topics — anything below the `---` separator is meta-instructions, not topics

### Phase 2: Research Each Topic
For each topic:
1. Use `web_search` with focused queries (3–6 words, include current year)
2. Aim for 3 developments per topic from the past 7 days
3. Prefer primary sources (research labs, university press releases, official announcements) over aggregators
4. If a topic is vague, narrow it using what's known about Xiulin from memory (control systems, CFD, NZ/EU pathways, etc.)

### Phase 3: Synthesize
For each item write:
- 1–2 sentence summary (paraphrased, never quoted)
- Source URL
- One sentence on **why it matters to Xiulin specifically** — connect to his current state (PhD, visa, NZ pathway, business, remote work options)

### Phase 4: Format & Save
1. Format using the structure in Output Format below
2. Get today's date in `YYYY-MM-DD` format
3. Use `Read and Write Apple Notes:add_note` with `name: "{date} Daily Briefing"`
4. Reply in chat with a 4–6 line digest summary + confirmation the note was saved

## Output Format

```
{YYYY-MM-DD} — DAILY BRIEFING (Xiulin)

═══════════════════════════════════
1. {TOPIC NAME}
═══════════════════════════════════

• {Headline}
  Why it matters: {personal relevance in 1 sentence}
  {URL}

• {Headline}
  Why it matters: {personal relevance}
  {URL}

• {Headline}
  Why it matters: {personal relevance}
  {URL}

═══════════════════════════════════
2. {TOPIC NAME}
═══════════════════════════════════
[... same structure ...]

═══════════════════════════════════
TODAY'S ONE THOUGHT
═══════════════════════════════════
{One short reflective observation connecting today's findings
to Xiulin's current decisions or patterns. Honest, not flattering.}
```

## Setup (First Run)
If `Daily Briefing Topics` note doesn't exist, create it with this default content:

```
EDIT THIS NOTE TO CHANGE WHAT GETS RESEARCHED EACH MORNING.
One topic per line. Changes take effect the next briefing — no code edits needed.

1. AI tools that enhance my productivity
2. Advances in turbulence research and CFD
3. Master or PhD programs fitting my background (control systems, fluid dynamics, EU/NZ/AU)
4. One-person company ideas or remote engineering jobs for a PhD candidate

---
INSTRUCTIONS FOR CLAUDE (do not delete):
For each topic, find top 3 developments from the past 7 days. Two sentences max per item, with URL and one line on why it matters. Output ONLY the briefing, no preamble.
```

## Changing Topics
If user says "change my topics", "update topics", "I want to track X instead":
1. Read current `Daily Briefing Topics` note
2. Show current topics in chat
3. Make the requested edits
4. Use `Read and Write Apple Notes:update_note_content` to save
5. Confirm the change will take effect next briefing

## Quality Guardrails
- **Never invent sources.** If a search returns nothing useful for a topic, say so explicitly in that section rather than fabricating items.
- **Never reuse yesterday's items.** If the user runs the briefing twice in one day, acknowledge it and offer to refresh from later in the day.
- **Personal relevance is mandatory.** Generic "this is important for the field" is forbidden — every "why it matters" must connect to something concrete about Xiulin's situation.
- **Honest "one thought."** The closing reflection is not flattery. It can challenge a pattern, surface a tradeoff, or suggest a reframe — never just "you're doing great."

## Chat Reply Format
After saving the note, reply with a tight summary:
```
Saved to Apple Notes as "{date} Daily Briefing".

{Topic 1}: {one-line headline of top item}
{Topic 2}: {one-line headline}
{Topic 3}: {one-line headline}
{Topic 4}: {one-line headline}

Today's thought: {the reflection from the note}
```
Keep it under 10 lines so it's readable on phone.

## Integration

This skill compounds with:
- **first-principles-decomposer** — When a briefing item suggests a major decision, decompose it before acting
- **second-order-consequences** — Project downstream effects of any opportunity surfaced in the briefing
- **conversation_search** — Cross-reference past briefings to detect patterns Xiulin keeps avoiding or returning to

## Skill Metadata
**Created**: 2026-04-10
**Last Updated**: 2026-04-10
**Author**: Xiulin (with Claude)
**Version**: 1.0

---
See framework.md for detailed methodology
See examples.md for sample briefings and topic configurations
