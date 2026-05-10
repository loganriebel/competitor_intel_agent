# Stage 03 — ICP Score

**Input:** `last_run/summaries.json`, `config/targets.yaml` (ICP definition)
**Output:** `last_run/scored.json`

Score each summary against the ICP definition and filter out low-resonance content before idea generation runs.

## Instructions

Load the full ICP definition from `config/targets.yaml` — role, pain points, resonant content types, and non-resonant signals.

For each item in `summaries.json`:

1. **Score ICP resonance on a 1–10 scale.** Consider:
   - Does the content address the ICP's role or day-to-day responsibilities?
   - Does it touch one or more of the ICP's stated pain points?
   - Is the content type listed under `content_that_resonates`?
   - Is the content type listed under `content_that_does_not_resonate`? (apply a floor of 3 if so)
   - Would the ICP likely save, share, or act on this?

2. **Assign a resonance label:**
   - `high` — score 7–10: directly targets ICP pain points, strong likelihood of engagement
   - `medium` — score 4–6: tangentially relevant, partial overlap with ICP concerns
   - `low` — score 1–3: off-brand, company news, or no meaningful overlap

3. **Write a one-sentence `resonance_reason`** explaining the score. Be specific — reference the pain point or content type that drove the score.

4. **Filter:** exclude `low` items from the output array entirely. Log how many were filtered.

## Output format

```json
[
  {
    "domain": "rival-analytics.com",
    "url": "https://rival-analytics.com/blog/how-to-track-competitor-ads",
    "title": "How to Track Competitor Ad Spend on Meta",
    "content_type": "how-to",
    "key_themes": ["Meta Ad Library", "competitor spend tracking", "ad intelligence"],
    "icp_score": 9,
    "icp_resonance": "high",
    "resonance_reason": "Directly addresses the ICP's core pain point of tracking competitor activity; how-to format with specific tool recommendations strongly matches stated content preferences."
  }
]
```

Write to `last_run/scored.json`. Log: `[03] {n} items scored — {high} high, {medium} medium, {low} filtered out.`

Sort the output by `icp_score` descending so the orchestrator compiles the report in relevance order.
