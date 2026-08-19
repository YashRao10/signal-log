# Research Brief Schema

The Research agent's ONLY job is to produce a JSON file matching this shape. No prose narrative, no draft copy — just facts with sources. The Writer agent drafts from this; the Reviewer agent checks the draft against this. Every field must be independently verifiable (a real tool-call result or a real URL), never estimated or recalled.

```json
{
  "generated_at": "2026-07-13T14:22:00-04:00",
  "trigger": "one line describing why this brief was requested — a specific mover, a scheduled macro print, a general check",
  "market_moves": [
    {
      "symbol": "TICKER",
      "pct_change": -4.4,
      "vs": "prior close | vs cost basis (public-safe framing only, never state the cost basis $ itself)",
      "catalyst": "one-sentence real reason, or 'no specific news found — reads as broad-market chop' if none exists",
      "sources": ["https://..."]
    }
  ],
  "macro": {
    "fed_funds_rate": {"value": 3.62, "as_of": "2026-07-09", "delta_3mo": -0.01},
    "treasury_10y": {"value": 4.54, "as_of": "2026-07-09", "delta_3mo": 0.25},
    "treasury_2y": {"value": 4.16, "as_of": "2026-07-09", "delta_3mo": 0.37},
    "cpi_yoy_pct": {"value": 4.3, "as_of": "2026-05-01"},
    "unemployment_rate": {"value": 4.2, "as_of": "2026-06-01", "delta_3mo": -0.1},
    "no_new_data": false
  },
  "indices": [
    {"symbol": "SPX", "value": 7575.39, "pct_change": 0.42}
  ],
  "notes_for_writer": "anything structural the writer needs to know — e.g. 'this replaces the existing Econ Briefing post' vs 'this is a new Insights post, prepend above existing ones'"
}
```

Rules for the Research agent specifically:
- Every `catalyst` needs a real source URL from an actual WebSearch result, or must say explicitly no catalyst was found.
- Every macro field needs a real `as_of` date from the actual FRED pull this session — never carry forward a stale date from a prior brief.
- Do not touch the site's HTML/RSS/sitemap files. Output only the JSON brief.
- Do not include any portfolio-specific $ figures or position sizes even in this internal file — habit discipline matters, and it removes any chance of a later stage accidentally leaking it into the public draft.
