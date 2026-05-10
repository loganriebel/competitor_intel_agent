# /competitor-intel

Orchestrator for the competitor intelligence pipeline. Chains five stage skills in sequence, resuming from the last completed stage if artifacts already exist from a partial run.

## On startup

1. Read `config/targets.yaml` — load the competitor domain list and ICP definition. If the file is missing, exit with: `Error: config/targets.yaml not found. Copy config/targets.example.yaml and fill in your values.`
2. Check which stage artifacts exist in `last_run/` to determine where to resume.
3. Start stage 05 (pricing check) in a background thread — it runs independently and doesn't block the content pipeline.

## Resume detection

Check for each artifact in order. Start from the earliest missing one:

| Missing artifact | Resume from |
|------------------|-------------|
| `last_run/new_urls.json` | Stage 01 |
| `last_run/summaries.json` | Stage 02 |
| `last_run/scored.json` | Stage 03 |
| `last_run/ideas.json` | Stage 04 |
| All artifacts present | Skip to report compilation |

## Stage sequence

Run each stage by reading its skill file and following its instructions:

1. **`skills/stages/01-sitemap-diff.md`** → writes `last_run/new_urls.json`
2. **`skills/stages/02-content-summarize.md`** → writes `last_run/summaries.json`
3. **`skills/stages/03-icp-score.md`** → writes `last_run/scored.json`
4. **`skills/stages/04-idea-generation.md`** → writes `last_run/ideas.json`
5. **`skills/stages/05-pricing-check.md`** → writes `last_run/pricing_diff.json` *(parallel)*

If `new_urls.json` is empty (no new content found for any domain), skip stages 02–04 and go directly to report compilation.

## Report compilation

After all stages complete, compile a single markdown report at `outputs/YYYY-MM-DD-report.md` using today's date. The report structure:

1. **Header** — run date, domains checked, total new URLs found, total ideas generated
2. **Per-domain content section** — for each domain with new content: list of new URLs, summaries, ICP scores, and generated ideas
3. **Pricing changes** — list only domains where `change_detected: true` from `pricing_diff.json`; if none, a single line: "No pricing changes detected."
4. **Footer** — tokens used this run, next scheduled run time

## On failure

If any stage fails, log the error with the stage name and exit. Do not attempt subsequent stages. Stage artifacts written before the failure are preserved — the next run will resume from the failed stage.

Never delete or overwrite domain snapshot files (`last_run/*-sitemap.json`, `last_run/*-pricing.txt`) — these are the memory used for diffing and are only written by the stage that creates them, never by the orchestrator.
