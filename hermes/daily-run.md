# Hermes Cron Job — Competitor Intel Daily Run

Configured in [Hermes](https://hermes-agent.nousresearch.com/) (Nous Research's autonomous agent). Runs daily at 6am CST in an isolated Hermes agent session.

## Schedule

Set this job to run daily at 6am CST in your Hermes dashboard or config. Hermes's built-in cron scheduler ticks every 60 seconds and executes due jobs in isolated sessions — no system cron required.

## Job prompt

```
You are running the daily competitor intelligence pipeline.

Before starting:
1. Confirm config/targets.yaml exists and contains at least one competitor domain.
   If missing or empty, stop and log: "Error: config/targets.yaml not found or empty."

2. Clear stage artifacts from the previous run. Delete if they exist:
   - last_run/new_urls.json
   - last_run/summaries.json
   - last_run/scored.json
   - last_run/ideas.json
   - last_run/pricing_diff.json

   Do NOT delete domain snapshot files (last_run/*-sitemap.json, last_run/*-pricing.txt).
   These are the memory used for diffing and must persist between runs.

3. Log: "[hermes] Run started at {ISO timestamp} — checking {n} domains"

Then run the pipeline by following the instructions in skills/competitor-intel.md.

After the pipeline completes:
- Confirm outputs/{date}-report.md was written.
- Log: "[hermes] Run complete at {ISO timestamp} — report at outputs/{date}-report.md"
- (Optional) Send a notification via your configured Hermes integration.
```

## Skills to load

Load `skills/competitor-intel.md` and all files in `skills/stages/` before running.

## On failure

If the pipeline exits with an error, Hermes logs the failure. The next scheduled run will resume from the last completed stage — stage artifacts written before the failure are preserved, and domain snapshots are never deleted.

## Changing the schedule

Update the cron schedule in Hermes. No pipeline code changes required.
