# Stage 01 — Sitemap Diff

**Input:** `config/targets.yaml` (competitor domains)
**Output:** `last_run/new_urls.json`

Fetch each competitor's sitemap, diff against the previous run's snapshot, and surface only genuinely new URLs.

## Instructions

For each domain in `config/targets.yaml`:

1. **Fetch the sitemap.** Try these paths in order, stopping at the first success:
   - `https://{domain}/sitemap.xml`
   - `https://{domain}/sitemap_index.xml`
   - `https://{domain}/post-sitemap.xml`
   - `https://{domain}/blog-sitemap.xml`

   If all fail: log `[01] Warning: no sitemap found for {domain}` and skip this domain.

2. **Parse the XML.** Extract all `<loc>` URLs. If the sitemap is an index (contains `<sitemap>` tags), fetch each child sitemap and combine their `<loc>` entries. Capture `<lastmod>` where present.

3. **Load the previous snapshot** at `last_run/{domain}-sitemap.json`. If absent (first run), treat it as an empty list.

4. **Diff.** Find URLs present in the new sitemap but absent from the snapshot. For URLs with `<lastmod>`, also include any that have a `lastmod` within the last 7 days and weren't flagged in the previous run — this catches sitemaps that update `lastmod` without adding new `<loc>` entries.

5. **Write the updated snapshot** to `last_run/{domain}-sitemap.json` (overwrite with the full current URL list).

6. **Append new URLs** to the output list with domain label.

## Output format

```json
[
  {
    "domain": "rival-analytics.com",
    "url": "https://rival-analytics.com/blog/how-to-track-competitor-ads",
    "lastmod": "2026-05-08",
    "title": null
  },
  {
    "domain": "marketlens.io",
    "url": "https://marketlens.io/blog/meta-ad-library-limitations",
    "lastmod": "2026-05-07",
    "title": null
  }
]
```

Write to `last_run/new_urls.json`. If no new URLs were found across all domains, write an empty array `[]` and log `[01] No new content detected across all domains.`

Log a summary line per domain: `[01] {domain}: {n} new URLs found` (including `0` counts).
