# /hermes-competitor-intel

Scheduler wrapper for the competitor intelligence pipeline. Invoked by system cron at 6am CST daily. Handles scheduling concerns only — all pipeline logic lives in `/competitor-intel`.

## Cron setup

```bash
# 6am CST = 12:00 UTC
0 12 * * * cd /path/to/competitor_intel_agent && claude -p "/hermes-competitor-intel"
```

To run twice daily, change to: `0 6,12 * * *`. No other changes needed.

## On startup

1. **Guard.** Check that `config/targets.yaml` exists and contains at least one competitor domain. If missing or empty, log `[hermes] Error: config/targets.yaml not found or empty — aborting` and exit without running the pipeline.

2. **Clear stage artifacts** from the previous run. Delete the following if they exist:
   - `last_run/new_urls.json`
   - `last_run/summaries.json`
   - `last_run/scored.json`
   - `last_run/ideas.json`
   - `last_run/pricing_diff.json`

   Do **not** delete domain snapshot files (`last_run/*-sitemap.json`, `last_run/*-pricing.txt`). These are the memory used for diffing.

3. **Log run start:** `[hermes] Run started at {ISO timestamp} — checking {n} domains`

## Run the pipeline

Invoke `/competitor-intel` and wait for completion.

If `/competitor-intel` exits with an error, log `[hermes] Pipeline failed at {timestamp} — see above for stage error` and exit. Do not send a report for failed runs.

## After the pipeline

1. Confirm `outputs/{date}-report.md` was written by the orchestrator.
2. Log: `[hermes] Run complete at {ISO timestamp} — report at outputs/{date}-report.md`
3. **(Optional) Send a notification.** Add your delivery method here:

   ```bash
   # Slack webhook example
   curl -s -X POST $SLACK_WEBHOOK_URL \
     -H 'Content-type: application/json' \
     --data "{\"text\": \"Competitor intel report ready: outputs/{date}-report.md\"}"
   ```

   Uncomment and configure as needed. Hermes doesn't require a notification — the report file is the deliverable.

## What Hermes does not do

- Parse or interpret pipeline results
- Retry failed stages (the resume logic in `/competitor-intel` handles that on the next scheduled run)
- Manage config or targets
