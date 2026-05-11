# Architecture

## Why sitemap diff over crawling

Sitemaps are machine-readable and updated at publish time. Crawling a full site daily is slow (100s of pages per domain), brittle (structure changes break selectors), and noisy (detecting real content changes in rendered HTML is unreliable).

Sitemap diffing fetches one XML file per domain, compares URLs against the last-run snapshot, and only fetches content for genuinely new URLs. On a 5-domain run this typically processes 3–8 new pages vs. crawling thousands. When a domain doesn't expose a standard sitemap, the agent tries `/sitemap_index.xml` and common CMS paths before falling back to a logged warning — never a silent miss.

---

## Why ICP filter before idea generation

Stage 04 (idea generation) is the most token-intensive step. Running it on every piece of competitor content wastes compute and produces off-target ideas.

Stage 03 scores each summary against the ICP definition in `config/targets.yaml` and drops anything that scores low. In practice this filters ~60% of competitor posts — product updates, company news, off-topic listicles. The ICP definition lives in config, not hardcoded in the skill file, so it evolves without touching the pipeline.

---

## Snapshot-as-memory pattern

The agent has no database. Each run writes its intermediate artifacts to `last_run/`:

```
last_run/
├── rival-analytics.com-sitemap.json      # Full URL list from previous run
├── adtracker-pro.com-sitemap.json
├── adtracker-pro.com-pricing.txt         # Pricing page text from previous run
├── new_urls.json                          # Stage 01 output (cleared each run)
├── summaries.json                         # Stage 02 output (cleared each run)
├── scored.json                            # Stage 03 output (cleared each run)
├── ideas.json                             # Stage 04 output (cleared each run)
└── pricing_diff.json                      # Stage 05 output (cleared each run)
```

Two distinct file types:
- **Domain snapshots** (`*-sitemap.json`, `*-pricing.txt`) — persist across runs; these are the memory used for diffing
- **Stage artifacts** (`new_urls.json`, `summaries.json`, etc.) — cleared by Hermes at the start of each run, then rebuilt stage by stage

Diffing flat files is fast and requires no infrastructure. Git history of the `last_run/` snapshots gives a free audit log — `git diff HEAD~1 last_run/adtracker-pro.com-pricing.txt` shows exactly what changed since yesterday.

---

## Stage isolation and resume logic

Each stage reads the previous stage's artifact, does one job, and writes its own artifact. The orchestrator skill (`skills/competitor-intel.md`) checks which artifacts exist at startup:

- `new_urls.json` missing → start from stage 01
- `summaries.json` missing → start from stage 02
- `scored.json` missing → start from stage 03
- `ideas.json` missing → start from stage 04
- `pricing_diff.json` missing → run stage 05

This means a network timeout during stage 02 resumes from stage 02 on the next run — sitemaps aren't re-fetched, ICP scores aren't re-computed. Stage 05 (pricing check) runs independently of the content pipeline, so a failure there doesn't block the ideas from surfacing.

---

## Hermes scheduling model

[Hermes](https://hermes-agent.nousresearch.com/) is an autonomous AI agent by Nous Research with a built-in cron scheduler. It ticks every 60 seconds, executes due jobs in isolated agent sessions, and uses OpenRouter for LLM routing. No system cron or daemon required.

The `hermes/daily-run.md` file defines what the Hermes job does each morning:
1. **Guard** — confirm `config/targets.yaml` is present and non-empty
2. **Clear** — delete stage artifacts from the previous run (preserving domain snapshots)
3. **Delegate** — load the pipeline skills and run the orchestrator

All pipeline logic — stage sequencing, resume detection, artifact chaining — lives in `skills/competitor-intel.md` and its stage files. Changing the schedule means updating it in Hermes. Adding a notification means one line in `hermes/daily-run.md`. Neither change touches the pipeline.

---

## Single agent over multi-agent

The pipeline runs as a single Claude Code agent session working through stage files sequentially. A multi-agent design (one agent per stage, passing artifacts via message) introduces coordination overhead and context hand-off errors without meaningful parallelism benefit at this scale.

The one exception: stage 05 (pricing check) runs independently because it touches different data sources and doesn't depend on the content pipeline. The orchestrator starts it in parallel with stages 01–04 and merges results in the final report.
