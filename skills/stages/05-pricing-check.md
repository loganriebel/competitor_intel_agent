# Stage 05 — Pricing Check

**Input:** `config/targets.yaml` (domains + pricing paths)
**Output:** `last_run/pricing_diff.json`

Runs in parallel with stages 01–04. Detects pricing page changes by diffing against the previous run's snapshot.

## Instructions

For each competitor that has a `pricing_path` in `config/targets.yaml`:

1. **Fetch** `https://{domain}{pricing_path}`. Set a 15-second timeout.

   If the page returns minimal content (< 200 characters of body text after stripping), log `[05] Warning: {domain} pricing page may be JavaScript-rendered — skipping diff to avoid false positive` and move on. Do not report a change.

2. **Extract pricing content** from the HTML:
   - Plan names
   - Prices (monthly and annual if shown)
   - Key feature bullets per plan (top 5 per plan is sufficient)
   - Any visible promotional pricing or banners

   Convert to a normalized plain-text representation — consistent whitespace, lowercase plan names, prices formatted as `$XX/mo`.

3. **Load the previous snapshot** at `last_run/{domain}-pricing.txt`. If absent (first run), write the current snapshot and mark `change_detected: false` — no diff possible on first run.

4. **Diff** the current text against the snapshot. Flag as a change if any of the following differ:
   - A price value changed
   - A plan name was added or removed
   - A feature bullet was added or removed from a plan

   Cosmetic changes (whitespace, punctuation, reordered bullets with identical text) should not trigger a change flag.

5. **Write the updated snapshot** to `last_run/{domain}-pricing.txt` (overwrite).

6. **Classify the change type** if a change was detected:
   `price_increase` · `price_decrease` · `plan_added` · `plan_removed` · `feature_added` · `feature_removed` · `multiple`

## Output format

```json
[
  {
    "domain": "adtracker-pro.com",
    "change_detected": true,
    "change_summary": "Pro tier price increased from $49/mo to $59/mo. No plan or feature changes detected.",
    "change_type": "price_increase",
    "previous_snapshot_date": "2026-05-08",
    "current_snapshot_date": "2026-05-09"
  },
  {
    "domain": "rival-analytics.com",
    "change_detected": false,
    "change_summary": null,
    "change_type": null,
    "previous_snapshot_date": "2026-05-08",
    "current_snapshot_date": "2026-05-09"
  }
]
```

Write to `last_run/pricing_diff.json`. Include all checked domains regardless of whether a change was detected — the orchestrator uses the full list when compiling the report.

Log: `[05] Pricing check complete — {n} domains checked, {c} changes detected.`
