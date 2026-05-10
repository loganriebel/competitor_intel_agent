# Stage 04 — Idea Generation

**Input:** `last_run/scored.json`, `config/targets.yaml` (ICP + brand voice)
**Output:** `last_run/ideas.json`

Generate differentiated content ideas for my site based on what competitors published, filtered through the ICP lens.

## Instructions

Load `brand_voice` from `config/targets.yaml`.

For each item in `scored.json`:

1. **Understand the competitor's angle.** What framing did they choose? What did they leave out? What assumption does their piece rest on?

2. **Generate 1–3 content ideas** for my site that:
   - Address the same ICP pain point from a differentiated angle — not a rewrite or summary of the competitor's piece
   - Find a unique hook: a contrarian take, a more specific audience, a better data source, a missing use case, a process the competitor glossed over
   - Match the brand voice from config
   - Are realistic to produce without data I don't have access to

3. For each idea, produce:
   - `title` — a working headline, specific enough to brief a writer
   - `angle` — one sentence on what makes this different from the competitor's version and why the ICP would prefer it
   - `format` — suggested content type that fits the angle
   - `icp_pain_point` — which specific pain point from the ICP definition this addresses
   - `source_url` — the competitor URL that surfaced this opportunity
   - `priority` — `high` if source had `icp_score` ≥ 8, `medium` otherwise

## Output format

```json
[
  {
    "title": "The Meta Ad Library Is Lying to You: What It Hides and How to Fill the Gaps",
    "angle": "Rival's post treats the Ad Library as a baseline tool; this piece attacks that assumption, names specific data gaps (spend estimates, dark posts, audience targeting), and positions better intelligence methods as the real answer — creating urgency for readers already using the Ad Library.",
    "format": "thought-leadership",
    "icp_pain_point": "competitor spend tracking",
    "source_url": "https://rival-analytics.com/blog/how-to-track-competitor-ads",
    "priority": "high"
  },
  {
    "title": "We Analyzed 500 Meta Ad Creatives from 5 Competitors — Here's What Separated the Top Performers",
    "angle": "Rival's piece is method-focused (how to track); this is data-focused (what we actually found). Specific numbers, named creative patterns, and a downloadable framework give the ICP something concrete to act on.",
    "format": "data-report",
    "icp_pain_point": "finding content ideas that actually convert",
    "source_url": "https://rival-analytics.com/blog/how-to-track-competitor-ads",
    "priority": "high"
  }
]
```

Write to `last_run/ideas.json`. Sort by `priority` (high first), then by `icp_score` of the source item.

Log: `[04] Generated {n} ideas from {m} scored items.`

Do not generate ideas for `medium`-resonance items if the total idea count from `high`-resonance items already exceeds 10 — quality over volume.
