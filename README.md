# competitor_intel_agent

> An autonomous AI agent that watches up to 5 competitor domains every morning — detecting new content, scoring it against your ICP, generating content ideas, and flagging pricing changes — then drops a structured report before your first coffee.

Built as part of my [AI agent portfolio](https://github.com/loganriebel). Runs daily at 6am CST via [Hermes](https://hermes-agent.nousresearch.com/) (Nous Research's autonomous agent with a built-in cron scheduler). Companion to [seo_agent_pipeline](https://github.com/loganriebel/seo_agent_pipeline).

---

## Results

Sample output from a live run (domains sanitized):

| Domain | New Content | ICP Resonance | Ideas Generated | Pricing Change |
|--------|-------------|---------------|-----------------|----------------|
| rival-analytics.com | 2 posts | High | 3 ideas | None |
| adtracker-pro.com | 0 posts | — | — | +$10/mo on Pro tier |
| creativespy.io | 1 post | Low (filtered out) | 0 ideas | None |
| marketlens.io | 3 posts | High | 5 ideas | None |
| spytools.co | 0 posts | — | — | None |

→ [See the full sanitized report](outputs/example-report-2026-05-09.md)

---

## What it does

```
config/targets.yaml
        │
        ▼
01 Sitemap Diff ──────────────────────► new_urls.json
        │
        ▼
02 Content Summarize ─────────────────► summaries.json
        │
        ▼
03 ICP Score ─────────────────────────► scored.json   (low-resonance filtered out)
        │
        ▼
04 Idea Generation ───────────────────► ideas.json

        ┌─ runs in parallel with 01–04 ─┐
        ▼                               │
05 Pricing Check ─────────────────────► pricing_diff.json

        │
        ▼
   Hermes Report ────────────────────► outputs/YYYY-MM-DD-report.md
```

Each stage writes a JSON artifact. The orchestrator reads the previous artifact before running the next stage — so a failed run resumes from the last completed step without re-doing work.

---

## Design decisions

- **Sitemap diff over crawling** — sitemaps are machine-readable, fast, and reliable; full crawls are brittle and expensive at daily frequency
- **ICP filter before idea generation** — stage 03 discards low-resonance content before stage 04 runs; this cuts tokens by ~60% on noisy competitor blogs and keeps ideas on-target
- **Stage files, not one giant prompt** — each `.md` skill file has one job and bounded context; the orchestrator chains them and detects which stage to resume from on partial failure
- **Snapshot files as memory** — `last_run/` snapshots replace a database; flat-file diffs are fast, auditable via git history, and require zero infrastructure
- **Hermes as thin scheduler** — the daily-run skill is a single-responsibility wrapper; changing cadence means one cron line, not code

---

## Stack

- **[Hermes](https://hermes-agent.nousresearch.com/)** (Nous Research) — autonomous agent runtime with built-in cron scheduler; runs the pipeline daily at 6am CST
- **OpenRouter** — LLM routing; Hermes routes inference requests through OpenRouter
- **Python 3.10+** — sitemap fetching (`requests` + `xml.etree`), HTML → plain text (`markdownify`), snapshot diffing
- **YAML config** — all domain targets, ICP definition, and output paths in one file; no hardcoding in stage files

---

## File map

```
competitor_intel_agent/
├── README.md                          # This file
├── ARCHITECTURE.md                    # Design decisions in depth
├── config/
│   └── targets.example.yaml           # Domains, ICP definition, output paths
├── skills/
│   ├── competitor-intel.md            # Orchestrator skill (/competitor-intel)
│   └── stages/
│       ├── 01-sitemap-diff.md         # Fetch sitemaps → diff → new_urls.json
│       ├── 02-content-summarize.md    # Fetch new URLs → summaries.json
│       ├── 03-icp-score.md            # Score against ICP → scored.json
│       ├── 04-idea-generation.md      # Generate content ideas → ideas.json
│       └── 05-pricing-check.md        # Scrape pricing page → pricing_diff.json
├── hermes/
│   └── daily-run.md                   # Scheduler wrapper (6am CST cron)
├── last_run/                          # Snapshot memory (gitignored except .gitkeep)
└── outputs/
    └── example-report-2026-05-09.md   # Sanitized sample run output
```

---

## Hermes scheduling

[Hermes](https://hermes-agent.nousresearch.com/) is an autonomous AI agent by Nous Research with a built-in cron scheduler that ticks every 60 seconds and executes due jobs in isolated agent sessions.

The `hermes/daily-run.md` file defines the cron job: what to clear, what to run, and how to deliver the report. Scheduling concerns live in Hermes; pipeline logic lives in the stage files. Swapping from daily to twice-daily means changing the schedule in Hermes, not touching any pipeline code.

---

## How to adapt it

1. Fork the repo
2. `cp config/targets.example.yaml config/targets.yaml` — fill in your domains and ICP definition
3. Create a Hermes cron job pointing at `hermes/daily-run.md`, scheduled for 6am your timezone
4. Optionally adjust ICP scoring thresholds in `skills/stages/03-icp-score.md`

The agent becomes significantly more useful with additional context: your existing content inventory, brand voice, and GSC performance data can all feed into the ICP scoring and idea generation stages.

---

## Requirements

- [Hermes](https://hermes-agent.nousresearch.com/) with an OpenRouter API key
- Python 3.10+
- `pip install requests markdownify pyyaml`
