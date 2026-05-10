# Stage 02 — Content Summarize

**Input:** `last_run/new_urls.json`
**Output:** `last_run/summaries.json`

Fetch and summarize each new URL from stage 01.

## Instructions

For each item in `new_urls.json`:

1. **Fetch the page HTML.** Set a 10-second timeout. On failure (4xx, 5xx, timeout), log `[02] Warning: could not fetch {url}` and skip — do not include it in the output.

2. **Strip boilerplate.** Remove navigation, headers, footers, sidebars, cookie banners, and ad slots. Keep the main content area only. Use `<article>`, `<main>`, or the largest coherent text block as the target.

3. **Extract the title.** Prefer `<h1>` content. Fall back to `<title>` tag (strip the site name suffix if present, e.g., " | Rival Analytics").

4. **Generate the summary.** Write 2–3 sentences that cover:
   - What the piece is actually about (not just the title restated)
   - The core argument or key takeaway
   - Any specific data points, tools, or named examples mentioned

5. **Classify the content type.** Choose the single best fit:
   `how-to` · `case-study` · `listicle` · `product-update` · `thought-leadership` · `comparison` · `data-report` · `other`

6. **Extract key themes.** 3–5 noun phrases that a content strategist would use to tag this piece (e.g., `["Meta Ad Library", "competitor spend tracking", "budget optimization"]`).

7. **Estimate word count** from the stripped text.

## Output format

```json
[
  {
    "domain": "rival-analytics.com",
    "url": "https://rival-analytics.com/blog/how-to-track-competitor-ads",
    "title": "How to Track Competitor Ad Spend on Meta",
    "summary": "Covers three methods for estimating competitor Meta ad spend using the Ad Library, third-party tools, and manual spreadsheet tracking. Emphasizes the Ad Library's free-tier limitations and positions paid tools as necessary for accuracy at scale.",
    "content_type": "how-to",
    "key_themes": ["Meta Ad Library", "competitor spend tracking", "ad intelligence", "budget estimation"],
    "word_count": 1840
  }
]
```

Write to `last_run/summaries.json`. Maintain the same order as `new_urls.json`. Log `[02] Summarized {n} of {total} URLs ({skipped} skipped due to fetch errors).`
