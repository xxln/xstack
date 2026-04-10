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

### Phase 3.5: Pull Personal Context
Before formatting, gather what's currently on Xiulin's mind:
1. Use `conversation_search` with 2–3 queries covering different threads (visa/supervisor, business/partnership, relationships/wellbeing, NZ/EU outreach)
2. Combine with what's in `userMemories` (already in context — don't re-fetch)
3. Identify the 3–5 threads that have appeared most recently and matter most
4. For each thread, note: current state, what's pending, what pattern (if any) it reveals

This phase is what makes the briefing feel like it's *for Xiulin* and not a generic news roundup. It must run before Phase 4.

### Phase 4: Format & Save
1. Format using the structure in Output Format below
2. Get today's date in `YYYY-MM-DD` format
3. Use `Read and Write Apple Notes:add_note` with `name: "{date} Daily Briefing"`
4. Reply in chat with a 4–6 line digest summary + confirmation the note was saved

## Output Format

**Critical: Apple Notes renders HTML.** Plain-text dividers (═══, ---) and blank lines collapse into running text and look terrible on phone. ALWAYS use HTML tags. Never use ASCII art for structure.

Use this exact HTML structure:

```html
<h1>Daily Briefing — {D Month YYYY}</h1>
<p><i>{One-line framing of the day, optional}</i></p>

<h2>1. {Topic Name}</h2>
<ul>
  <li><b>{Headline}</b><br>
  {1–2 sentence summary}<br>
  <i>Why it matters:</i> {personal relevance, 1 sentence}<br>
  <a href="{URL}">{domain} →</a></li>

  <li><b>{Headline}</b><br>
  {summary}<br>
  <i>Why it matters:</i> {relevance}<br>
  <a href="{URL}">{domain} →</a></li>

  <li><b>{Headline}</b><br>
  {summary}<br>
  <i>Why it matters:</i> {relevance}<br>
  <a href="{URL}">{domain} →</a></li>
</ul>

<h2>2. {Topic Name}</h2>
<ul>...</ul>

<h2>3. {Topic Name}</h2>
<ul>...</ul>

<h2>4. {Topic Name}</h2>
<ul>...</ul>

<h2>Where You Are Right Now</h2>
<p>{2–4 sentence honest synthesis of the threads currently on Xiulin's mind — visa status, supervisor relationship, business decisions, relationships, wellbeing patterns. Pull from conversation_search and userMemories. Name what's pending, what's stuck, and what pattern (if any) is showing up across threads. Do not flatter. Do not therapize.}</p>

<h2>Suggestions for Today</h2>
<ul>
  <li><b>{Action}</b> — {1 sentence on why, tied to a specific briefing item or current concern}</li>
  <li><b>{Action}</b> — {why}</li>
  <li><b>{Action}</b> — {why}</li>
</ul>
<p><i>Suggestions must be concrete and same-day actionable (≤30 minutes each). No "consider exploring" or "you might want to think about." Either say what to do or don't include it.</i></p>

<h2>Today's One Thought</h2>
<p>{Reflection — honest, not flattering. May use <b>bold</b> for emphasis on the key sentence.}</p>
```

### Formatting rules
- Use `<h1>` for the title, `<h2>` for each topic, "Where You Are Right Now," "Suggestions for Today," and the closing thought — never lower levels
- Headlines are `<b>bold</b>`, "Why it matters" labels are `<i>italic</i>`
- Links use a clean format: `<a href="full-url">domain.com →</a>` — never paste raw URLs into running text
- One `<ul>` per topic; one `<li>` per item with `<br>` between summary, relevance, and link
- If a section has no new items, use a single `<p>` paragraph instead of an empty list — say so honestly ("No new launches today worth flagging")
- Never use ASCII dividers, never use Markdown (`#`, `**`, `-`), never use blank lines as structure
- Keep total length scannable on phone — aim for under one screen of scrolling per topic

### Personal context discipline
- The "Where You Are Right Now" section must reflect *current* threads from recent conversations, not static memory facts. If conversation_search returns nothing new, say "no significant updates since yesterday" rather than recycling old summaries.
- Suggestions must be actionable today, not abstract. Bad: "reflect on your priorities." Good: "open the CCNY application page and read the requirements (15 min)."
- Never pretend to be a therapist. If a wellbeing pattern surfaces (avoidance, withdrawal, rumination), name it briefly and once — don't dwell, don't diagnose, don't repeat what's already been said in past conversations.
- If something serious comes up in conversation_search (escalating distress, isolation, crisis language), the suggestion section should include reaching out to a professional or trusted person — directly, not euphemistically.

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
**Version**: 1.2 — added Phase 3.5 (pull personal context from conversation_search + memory) and two new output sections: "Where You Are Right Now" and "Suggestions for Today"

---
See framework.md for detailed methodology
See examples.md for sample briefings and topic configurations
