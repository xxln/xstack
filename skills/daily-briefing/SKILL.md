---
name: daily-briefing
description: Generate Xiulin's morning research briefing across configurable topics, save to either Notion or Apple Notes. Use when user says "daily briefing", "morning brief", "morning update", "what's new today", "today's briefing", "brief me", "run my briefing", "daily update", or any similar request for a recurring research summary. Default save destination is Notion (works on all platforms). Use Apple Notes only when user explicitly requests it or is on macOS.
---

# Daily Briefing

## When To Use
- User asks for "daily briefing", "morning brief", or "what's new today"
- User says "brief me" or "run my briefing"
- Start of a new day when user wants a research roundup
- User wants to update or change which topics get researched
- User wants to switch save destination between Notion and Apple Notes

---

## Save Destination

The briefing can be saved to **Notion** or **Apple Notes**. They are not equivalent:

| | Notion | Apple Notes |
|---|---|---|
| Works on iPad / web | ✅ Yes | ❌ No |
| Works on macOS Claude app | ✅ Yes | ✅ Yes |
| Auto-save via Claude | ✅ Yes | ✅ macOS only |
| Rich formatting | ✅ Yes | ✅ HTML rendered |

### How to choose
- **Default**: Use **Notion** unless the user explicitly requests Apple Notes or is clearly on macOS.
- **User says "save to Notes" / "use Apple Notes"**: Switch to Apple Notes for that session.
- **User says "save to Notion" / "use Notion"**: Use Notion.
- **Ambiguous**: Default to Notion and note at the end that Apple Notes is available on macOS.

### Detecting platform
If the user mentions being on Mac or macOS, or if Apple Notes tools respond without error, Apple Notes is available. On iPad or web, only Notion works. If Apple Notes fails mid-run, fall back to Notion automatically and tell the user.

---

## Core Principle
**Topics live in a config location, not in the skill.** The user can change focus areas anytime without touching code or asking Claude to "remember" preferences.

- **Notion**: A single top-level page called `Daily Briefing` holds the topic list inside a section headed `Topics`. Each day's briefing is saved as a subpage under this same `Daily Briefing` page, so everything lives in one place.
- **Apple Notes**: A single editable note called `Daily Briefing Topics` holds the topic list. Each day's briefing is saved as a separate note titled `{date} Daily Briefing`.

## The Process

### Phase 0: Determine Save Destination
1. Check the user's message for explicit preference ("save to Notes", "use Notion", etc.)
2. If no preference stated, default to **Notion**
3. All read/write operations in this run use the same destination

### Phase 1: Read Topics

**If using Notion:**
1. Search for the top-level page titled `Daily Briefing` using `Notion:notion-search` (query: `Daily Briefing`, filter to page type). Save the page ID — you'll need it again in Phase 4 as the parent for today's briefing subpage.
2. Fetch the page with `Notion:notion-fetch`.
3. Locate the `Topics` section — a heading (usually `## Topics` or `# Topics`) followed by a numbered or bulleted list. Parse the list items as the active topics. Ignore everything outside that section (other headings, past briefing links, instructions blocks).
4. If the `Daily Briefing` page does not exist, follow **Setup (First Run)** below to create it, then continue.
5. If the page exists but has no `Topics` section or the section is empty, tell the user and ask what topics to add before proceeding.

**If using Apple Notes:**
1. Use `Read and Write Apple Notes:get_note_content` with `note_name: "Daily Briefing Topics"`
2. If the note doesn't exist, create it with `Read and Write Apple Notes:add_note` using the default template and tell the user
3. Parse the numbered list — anything below the `---` separator is meta-instructions, not topics

### Phase 1.5: Pull Extracted Briefing Items as Research Feedback (Notion only)
This phase only runs when the active destination is Notion. Skip entirely for Apple Notes. **This phase must run before Phase 2 so the items can steer the research.**

The `Daily Briefing` parent page contains a child database called `Extracted Briefing Items`. This database holds items Xiulin has previously flagged, along with **her feedback** — reactions, corrections, follow-up questions, "more like this," "less like this," etc. Treat these as research guidance: they tell you what angles to pursue, what sources she trusts, what she has already seen, and what she found unhelpful.

1. When fetching the `Daily Briefing` page in Phase 1, look for a child database reference in the page body — it appears as `<database url="...">` or a `<data-source url="collection://...">` tag. Save the database/data source ID.
2. If the database is not found, skip this phase silently, note "no extracted items database found" for Phase 4, and proceed to Phase 2 with no feedback context.
3. Fetch the database with `Notion:notion-fetch` to inspect the schema. Identify:
   - The **date** property (prefer an explicit date column like `Date`, `Extracted On`, `Captured`; fall back to `Created time` if none exists)
   - The **feedback** property (any rich-text field named `Feedback`, `Notes`, `Reaction`, `Comment`, etc.) — this is the most important one
   - Any tags, topic-link, or source URL property
4. Find the most recent date present in the database:
   a. If the parent page already has a view sorted by date descending, prefer `Notion:notion-query-database-view` on that view URL.
   b. Otherwise create a temporary view with `Notion:notion-create-view`: type `table`, `configure: "SORT BY \"<date property>\" DESC"`, then query it. The view can persist — no cleanup required.
   c. Read the date of the first row returned; that is the "most recent date."
5. Collect every row whose date equals that most recent date. For each row, capture: title, summary, feedback text, tags, and any source URL.
6. **Synthesize the feedback into research guidance** before Phase 2 starts. Produce a short internal list of:
   - Angles Xiulin wants more of (from positive feedback or "follow up on X")
   - Angles to avoid (from "already seen this," "not relevant," "too shallow")
   - Specific follow-up questions she left open
   - Sources/outlets she has explicitly praised or dismissed
   - Items whose titles/URLs should not be re-surfaced in today's research
7. Pass both the raw items and the synthesized guidance forward:
   - The **guidance** shapes Phase 2 search queries and source selection.
   - The **raw items** are rendered as a dedicated section in Phase 4 output, so Xiulin can see which feedback was acted on and which items are still on her radar.
8. If the database exists but has zero rows, skip guidance synthesis, note "extracted items database is empty" once for Phase 4, and proceed to Phase 2 normally.

### Phase 2: Research Each Topic
For each topic:
1. **First, apply any relevant feedback from Phase 1.5.** If the synthesized guidance contains angles-to-pursue or angles-to-avoid that match this topic, let them shape your queries. Example: if feedback on an "AI safety" topic says "less policy, more technical interpretability," bias queries toward interpretability research and away from policy news.
2. Use `web_search` with focused queries (3–6 words, include current year)
3. Aim for 3 developments per topic from the past 7 days
4. Prefer primary sources (research labs, university press releases, official announcements) over aggregators — and honor source-level preferences from feedback (if Xiulin has praised or dismissed specific outlets, follow that)
5. **Do not re-surface items Xiulin has already seen.** If a search result title or URL matches a Phase 1.5 extracted item, skip it and find a different development.
6. If a topic is vague, narrow it using Phase 1.5 feedback first, then fall back to what's known about Xiulin from memory (control systems, CFD, NZ/EU pathways, etc.)

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

Format using the HTML structure in Output Format below, then save:

**If saving to Notion:**
1. Use `Notion:notion-create-pages` with `parent: {page_id: "<Daily Briefing page id from Phase 1>"}` so the new briefing is created as a subpage of the `Daily Briefing` parent page. Title: `{YYYY-MM-DD} Daily Briefing`. Body: the briefing content (see Output Format below).
2. **Reorder so the newest briefing appears at the top of the subpage list.** By default, Notion appends new child pages to the bottom. To correct this, immediately after creating the subpage:
   a. Fetch the `Daily Briefing` parent page with `Notion:notion-fetch` to see the current block layout, including the `<page url="...">` reference for the briefing you just created and any prior briefing references.
   b. Use `Notion:notion-update-page` with `command: update_content` to move the new page reference to the top of the `## Briefings` section (or, if no such section exists on the page, to the position immediately below the `## Topics` section). The `old_str` should capture the current placement of the new reference plus enough surrounding context to be unique; the `new_str` should move it to the intended top position. Leave the `Topics` section and any "Notes for Claude" section untouched.
   c. If the parent page has no `## Briefings` section yet, add one while performing the reorder: insert `## Briefings` as a heading directly below the `## Topics` section and place the new page reference beneath it.
3. Confirm: `Saved to Notion under "Daily Briefing" as "{date} Daily Briefing" (pinned to top of briefings list).`

**If saving to Apple Notes:**
1. Use `Read and Write Apple Notes:add_note` with `name: "{date} Daily Briefing"`
2. Confirm: `Saved to Apple Notes as "{date} Daily Briefing".`

**If save fails** (e.g. Apple Notes unavailable on iPad):
1. Present the full briefing HTML in chat
2. Explain which destination failed and why
3. Offer to save to the alternative destination instead

## Output Format

**Critical: Both Notion and Apple Notes render HTML.** Plain-text dividers (═══, ---) and blank lines collapse into running text. ALWAYS use HTML tags. Never use ASCII art for structure.

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

<h2>From Your Extracted Items — {Most Recent Date}</h2>
<p><i>Items from the <b>Extracted Briefing Items</b> database that shaped today's research. Your feedback on these items was used to bias search queries and skip already-seen results.</i></p>
<ul>
  <li><b>{Item title}</b><br>
  {summary}<br>
  <i>Your feedback:</i> {feedback text, if present}<br>
  {tags, if present}<br>
  <a href="{URL from database row, if any}">{domain} →</a></li>
  <li>...</li>
</ul>
<p><i>(If the database is missing, empty, or failed to load, replace this section with a single sentence explaining which condition applied — do not omit the heading silently.)</i></p>

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
- Use `<h1>` for the title, `<h2>` for each topic, the "From Your Extracted Items" section, "Where You Are Right Now," "Suggestions for Today," and the closing thought — never lower levels
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

### Notion
If a top-level page titled `Daily Briefing` does not exist:
1. **Ask the user** what topics they want to track — do not invent or assume topics.
2. Create the page with `Notion:notion-create-pages` (workspace-level, no parent) with title `Daily Briefing` and content structured like this:

```markdown
This page is the home for Xiulin's daily briefings. Edit the **Topics** section below to change what gets researched each morning — changes take effect on the next briefing run. Each day's briefing is saved as a subpage under the **Briefings** section, with the most recent at the top.

## Topics

1. {topic as provided by user}
2. {topic as provided by user}
3. {topic as provided by user}

## Briefings

_(New briefings are added here automatically. Most recent at the top.)_

## Extracted Briefing Items

_(Child database — items Xiulin flags from past briefings. The daily briefing pulls rows from the most recent date here.)_

## Notes for Claude

For each topic, find top 3 developments from the past 7 days. Two sentences max per item, with URL and one line on why it matters.
```

3. After creating the page, also create the `Extracted Briefing Items` child database using `Notion:notion-create-database` with `parent: {page_id: "<Daily Briefing page id>"}` and schema:

```sql
CREATE TABLE (
  "Title" TITLE,
  "Summary" RICH_TEXT,
  "Feedback" RICH_TEXT,
  "Source" URL,
  "Tags" MULTI_SELECT('ai':blue, 'policy':red, 'research':green, 'business':orange),
  "Date" DATE
)
```

4. Confirm the page and database were created and the first briefing is ready to run.

### Apple Notes
If `Daily Briefing Topics` does not exist in Apple Notes:
1. **Ask the user** what topics they want to track — do not invent or assume topics.
2. Create the config note with this structure:

```
EDIT THIS NOTE TO CHANGE WHAT GETS RESEARCHED EACH MORNING.
One topic per line. Changes take effect the next briefing — no code edits needed.

1. {topic as provided by user}
2. {topic as provided by user}
3. {topic as provided by user}
...

---
INSTRUCTIONS FOR CLAUDE (do not delete):
For each topic, find top 3 developments from the past 7 days. Two sentences max per item, with URL and one line on why it matters. Output ONLY the briefing, no preamble.
```

3. Confirm the note was created and the first briefing is ready to run.

## Changing Topics
If user says "change my topics", "update topics", "I want to track X instead":
1. Read the topic list from the active destination:
   - **Notion**: fetch the `Daily Briefing` page and read the `Topics` section
   - **Apple Notes**: read the `Daily Briefing Topics` note
2. Show current topics in chat
3. Make the requested edits
4. Save using the appropriate tool:
   - **Notion**: `Notion:notion-update-page` with a `update_content` edit that rewrites only the `Topics` section — leave the rest of the page (briefing subpages, notes for Claude) untouched
   - **Apple Notes**: `Read and Write Apple Notes:update_note_content`
5. Confirm the change will take effect next briefing

## Changing Save Destination
If user says "switch to Notion", "save to Apple Notes", "change where briefings are saved":
1. Acknowledge the change
2. Note the platform constraint if relevant: "Apple Notes only works on the macOS Claude app — on iPad or web, briefings will need to be pasted manually or saved to Notion instead."
3. Apply the change for the current session
4. Remind the user that the topic config lives in each destination separately (Notion: `Daily Briefing` page → `Topics` section; Apple Notes: `Daily Briefing Topics` note), and they may want to copy topics over to keep both in sync.

## Quality Guardrails
- **Never invent sources.** If a search returns nothing useful for a topic, say so explicitly in that section rather than fabricating items.
- **Never reuse yesterday's items.** If the user runs the briefing twice in one day, acknowledge it and offer to refresh from later in the day.
- **Personal relevance is mandatory.** Generic "this is important for the field" is forbidden — every "why it matters" must connect to something concrete about Xiulin's situation.
- **Honest "one thought."** The closing reflection is not flattery. It can challenge a pattern, surface a tradeoff, or suggest a reframe — never just "you're doing great."

## Chat Reply Format
After saving the note, reply with a tight summary:
```
Saved to {Notion / Apple Notes} as "{date} Daily Briefing".

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
**Last Updated**: 2026-04-11
**Author**: Xiulin (with Claude)
**Version**: 1.8 — Extracted Briefing Items are now pulled in Phase 1.5 (before research) and their feedback is synthesized into research guidance that shapes Phase 2 queries, source selection, and already-seen deduping. Items are still rendered in the output so Xiulin can see which feedback was acted on. Starter database schema now includes a `Feedback` column.

---
See framework.md for detailed methodology
See examples.md for sample briefings and topic configurations
