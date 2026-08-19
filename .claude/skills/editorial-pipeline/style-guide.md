# YR Signal — Editorial Style & Privacy Contract

This is the ground-truth contract for anything published to `index.html`, `rss.xml`, or `sitemap.xml` in this repo. The Writer and Reviewer agents in the editorial pipeline both read this file — the Reviewer checks the draft against it line by line, not from memory.

## Hard privacy rules (never violate, no exceptions)

1. **No dollar amounts.** Never state what anything is worth, what was spent, or what was received.
2. **No position sizes or share counts.** Share counts are mathematically equivalent to dollar amounts once combined with a public historical price (the exact price on the trade date is public; share count × price = dollar amount). Never include a share quantity, fractional or whole.
3. **No portfolio percentages or weights.** Never state "X% of the portfolio" or imply relative sizing between holdings (e.g. never say "my biggest holding is X").
4. **Public market data is always fine**: index levels, sector ETF returns, individual stock prices/quotes, FRED macro series, analyst price targets, company news. The line is data *about this portfolio's composition or size* vs. data *about the market*.
5. When in doubt, a number is only public-safe if a stranger with zero knowledge of this portfolio could look it up independently (a stock's closing price, a Fed rate, an index level) — not if it requires knowing something about *this account* to compute.

## No-fabrication rule

- Never invent a stat, price move, or FRED reading that wasn't actually pulled this session.
- On a day with no new data (market holiday, no new FRED release), say so explicitly ("no new release this week") rather than restating an old number as if it's fresh, and rather than skipping the section silently.
- Every numeric claim in a published post must trace back to a specific field in that session's research brief — not "recalled" from a prior post or estimated.

## Voice & audience

- Target reader is a non-expert peer — informed but not necessarily fluent in finance jargon. Define terms on first use or point to the glossary.
- Plain-language over jargon. When a technical term is necessary (e.g. "MC/DC," "2s10s curve"), define it in the same sentence or nearby.
- Direct and matter-of-fact, not hype-y. Real losses are shown alongside real wins — credibility depends on that.
- Ticker symbols are fine to name freely (this is a trade-log site, not anonymized) — the constraint is on size/value, not identity.

## Structural conventions

**Insights / Market Notes post:**
```html
<div class="post">
  <div class="post-title">{Headline — specific, not generic}</div>
  <div class="post-meta">{Mon DD, YYYY} · Market Analysis</div>
  <p>{2-5 paragraphs, each grounded in specific public data points}</p>
</div>
```
New posts are **prepended** above older ones (newest first) — the Insights tab accumulates, it never overwrites.

**Econ Briefing post** (`#econ` section):
This section **replaces its single post in place** — it does not accumulate like Insights. Structure: macro stat-tile row (Fed Funds / 10Y / 2Y / CPI YoY / Unemployment, sourced from that session's FRED pull) followed by one `<div class="post">` block with the narrative.

**After any publish:**
- Bump the header `<div class="updated-line" id="updated-line">` to the actual current date.
- Add a matching `<item>` to `rss.xml` for Insights/Trade Log posts (Econ Briefing updates do NOT get an RSS item — established precedent, don't add one).
- Bump `sitemap.xml`'s `<lastmod>`.

## Date discipline

- Confirm today's actual date before writing it anywhere. Don't trust a stale injected date over what the user has stated directly in the conversation.
- Never date a post in the future or claim data is "live" if it's actually a snapshot from earlier in the day — say "as of {time}" if there's any ambiguity.
