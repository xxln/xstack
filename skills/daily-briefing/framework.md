# Daily Briefing Framework — Methodology

## The Core Insight
A daily briefing fails when it becomes either (a) generic news Xiulin could get from any RSS feed, or (b) a flattery loop that reinforces what he already wants to hear. The skill's job is to deliver **personally consequential signal** — items that connect to a specific decision, opportunity, or pattern in his life right now.

## The Four-Layer Filter

Every candidate item passes through four filters before making the briefing:

### Layer 1: Recency
Was this published or significantly updated in the last 7 days? If older but newly relevant (e.g., a paper resurfaced because of a new application), note the original date.

### Layer 2: Substance
Is this an actual development, or just commentary on someone else's development? Skip the commentary, link the original.

### Layer 3: Relevance
Does it connect to Xiulin's situation? Use the personal context lens:
- **Academic**: Italy PhD, NZ pathway, control systems, CFD/turbulence
- **Migration**: EU and NZ visa/skilled-migrant pathways, English-taught programs
- **Income**: Remote work, one-person company, algorithm commercialization
- **Decision-making**: Items that bear on choices he's actively weighing

### Layer 4: Actionability
Could he do something with this in the next 7 days — read it, email someone, apply, save it for later? If no plausible action exists, deprioritize.

## Search Strategy

### Query Construction
- 3–6 words, specific
- Include the current year for recency-sensitive topics
- Use the entity name when known (lab, university, person, product) — these rank higher than generic terms
- Avoid quotation marks unless searching for an exact phrase

### Source Hierarchy (highest to lowest)
1. Primary: research lab pages, university press releases, official product announcements, government sites
2. Secondary: reputable journalism (Nature News, Quanta, MIT Tech Review, Reuters)
3. Tertiary: aggregators (only if primary unavailable)
4. Avoid: SEO listicles, content farms, undated blog posts

### When Search Returns Junk
- Reformulate with different terms — don't search the same query twice
- If still nothing: explicitly say "no significant developments this week" rather than padding with weak items
- A 2-item topic is better than a 3-item topic with one fabricated entry

## The "Why It Matters" Discipline

This is the highest-leverage line in the whole briefing. It's also the easiest place to slip into generic filler.

### Bad (generic)
"Why it matters: This is an important development in the field."

### Bad (flattery)
"Why it matters: Given your impressive background, you'll find this interesting."

### Good (specific to context)
"Why it matters: KU Leuven is English-taught and Schengen — a direct sideways move from L'Aquila if the Shanghai consulate doesn't reply this week."

### Good (challenges a pattern)
"Why it matters: This is the third remote-control-systems opportunity we've surfaced this month. The pattern is real — the question is what's stopping you from applying."

The test: could this exact "why it matters" sentence be copy-pasted into someone else's briefing? If yes, rewrite it.

## The "Today's One Thought" Discipline

The closing reflection is the only place in the briefing where Claude speaks in its own voice rather than reporting findings. It should:

- Connect 2+ items in the briefing to surface a pattern
- OR connect a briefing item to something from Xiulin's known situation
- OR challenge a tendency (avoidance, overcommitment, familiar-safety bias)
- Be honest, not encouraging
- Be one paragraph max — usually 2–3 sentences

### Anti-patterns to avoid
- "You're making great progress on..."
- "Keep up the good work with..."
- "It's exciting to see..."
- Anything that would feel at home on a motivational poster

### Good examples
- "Three of today's four topics surfaced opportunities in the EU. NZ is still your stated priority. Worth noticing that the search results don't reflect that priority — either the priority needs updating or the search topics do."
- "The remote-job track is currently underweighted in your planning. It's the only option that addresses income, Western work history, and visa optionality simultaneously, and yet it's the one you talk about least."

## Failure Modes

### Failure 1: Topic drift
The user changes the Apple Note topics but Claude keeps using the old ones from memory. **Fix**: Always re-read the note at the start of each briefing. Never cache topics.

### Failure 2: Source recycling
Same 3 items appearing in multiple briefings because the search engine ranks them highly. **Fix**: Use `conversation_search` to check the past 3 briefings and skip already-mentioned URLs.

### Failure 3: Over-formatting
Briefing becomes so visually structured it takes longer to scan than to read. **Fix**: The format in SKILL.md is the ceiling, not the floor — strip dividers if a briefing is short.

### Failure 4: Hedging language
"This may be relevant", "could potentially be useful", "might be worth considering". **Fix**: State the connection or don't include the item.
