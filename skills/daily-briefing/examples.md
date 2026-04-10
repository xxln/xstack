# Daily Briefing Examples

## Example Topic Configurations

### Configuration A: Current default (research + career)
```
1. AI tools that enhance my productivity
2. Advances in turbulence research and CFD
3. Master or PhD programs fitting my background
4. One-person company ideas or remote engineering jobs
```

### Configuration B: Heavy on migration
```
1. New Zealand skilled migrant policy and Green List engineering changes
2. EU Blue Card and post-study work visa updates (Italy, Netherlands, Germany)
3. PhD programs in control systems / fluid dynamics with funded positions
4. Remote engineering roles open to non-EU candidates
```

### Configuration C: Heavy on commercialization
```
1. Indie hacker / one-person SaaS launches in technical/scientific niches
2. Algorithm licensing models and IP protection for solo founders
3. CFD and simulation software market trends
4. AI tools that reduce development time for solo technical founders
```

### Configuration D: Reflection-focused
```
1. Research on decision-making under uncertainty
2. Stories of people who navigated multi-country academic careers
3. Frameworks for evaluating partnership and equity offers
4. Articles on intrinsic motivation vs achievement-driven anxiety
```

## Example Briefing — Short Topic Returns Nothing

When a topic genuinely has no significant developments in the past week, do NOT pad with weak items. The honest version:

```
═══════════════════════════════════
2. ADVANCES IN TURBULENCE RESEARCH
═══════════════════════════════════

No significant new developments in the past 7 days. The Georgia Tech 35-trillion-point DNS from February remains the most-cited recent work — worth revisiting if you haven't read it: https://phys.org/news/2026-02-supercomputer-simulations-turbulence-theories-trillion.html

Suggestion: this topic may be too narrow for daily cadence. Consider broadening to "fluid mechanics + scientific computing" or moving to a weekly check.
```

This is more useful than three padded items because it (a) tells the truth, (b) gives one fallback resource, (c) suggests a configuration change.

## Example "Today's One Thought" Variations

### Variation 1: Pattern noticing
> Two of the four topics today surfaced KU Leuven and TU Eindhoven. The Netherlands route has been showing up consistently for three weeks. Worth noting that you've never explicitly considered it — only Italy and NZ have made it into your active planning.

### Variation 2: Counter-balancing optimism
> Today's AI section is full of "10x productivity" claims. The honest read: most of these tools save 30 minutes a day at best, and you already use the highest-leverage ones (Claude, NotebookLM). The marginal next tool probably isn't the bottleneck — your supervisor meeting prep is.

### Variation 3: Surfacing avoidance
> Remote job listings appeared in your briefing for the fifth straight day. You haven't applied to any. That's information: either the topic should come off the list, or there's something underneath the not-applying that's worth looking at directly.

### Variation 4: Tradeoff naming
> The KU Leuven PhD and the remote €60k contract role are mutually exclusive in practice — you can't do a funded PhD and a full-time remote job. Picking one means closing the other. Worth being explicit about which you're actually choosing.

## Example Chat Reply (after saving)

```
Saved to Apple Notes as "2026-04-10 Daily Briefing".

AI tools: GPT-5.4 hits 1M context + autonomous desktop workflows
Turbulence: MIT Lozano-Durán publishes causal-inference forecasting paper
Programs: KU Leuven funded PhD in neuromorphic control just opened
Remote/business: 3,228 PhD-tagged remote roles on FlexJobs as of today

Today's thought: The remote-job track keeps surfacing but you keep
not applying. Worth 30 minutes today to ask why.
```

## Example: User Wants to Change Topics Mid-Conversation

**User**: "Can you swap out the AI tools topic for something on Italian visa policy?"

**Claude action**:
1. Read current `Daily Briefing Topics` note
2. Show user the current list
3. Replace line 1 with the new topic
4. Save with `update_note_content`
5. Confirm: "Done. Tomorrow's briefing will track Italian visa policy instead of AI tools. The change is in your `Daily Briefing Topics` note — you can edit it directly anytime."

## Example: Briefing Run Twice Same Day

**User** (at 2pm, after morning briefing already saved): "Run my briefing"

**Claude response**: "You already have today's briefing saved. Want me to (a) refresh it with anything new since this morning, (b) re-run a specific topic, or (c) skip it and read tomorrow's tomorrow?"

Don't silently overwrite the morning note — that destroys data. Always ask.
