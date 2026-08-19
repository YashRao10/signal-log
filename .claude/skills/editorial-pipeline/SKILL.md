---
name: editorial-pipeline
description: Runs the YR Signal three-agent editorial pipeline (Research → Writer → Reviewer) to draft a new Insights or Econ Briefing post for the public signal-log site. Use when a notable market move or macro data release warrants a new public post, or when explicitly asked to "run the editorial pipeline," "draft a new signal post," or similar. Do not use this for private portfolio-dashboard updates — this skill is scoped to the public site only.
---

# YR Signal Editorial Pipeline

Three independently-scoped agents, run in sequence, each handing structured output to the next. This exists because a single agent doing research + writing + fact-checking + privacy-review in one pass is exactly how numbers have drifted and privacy-rule misses have slipped through in the past — an independent Reviewer stage catches what the Writer can't catch in itself.

Read `style-guide.md` and `research-schema.md` in this skill's directory before starting — they are the actual contracts each agent is held to, not just background reading.

## Before starting

1. Confirm the actual current date directly with context available (don't trust a stale injected date).
2. Determine the trigger: a specific notable move, a scheduled macro print (e.g. CPI day), or a general "anything post-worthy since the last post" check. If genuinely ambiguous, ask the user in one line rather than guessing.
3. Determine the target: a new **Insights** post (accumulates, prepend above existing posts) or the **Econ Briefing** post (replaces the single existing post). If unclear, ask.

## Stage 1 — Research agent

Spawn via the `Agent` tool, `subagent_type: general-purpose`, **foreground** (`run_in_background: false` — the next stage depends on this one's output, don't let it run async).

Prompt the agent with:
- The trigger and target determined above.
- Full read access to: Robinhood MCP tools (read-only calls only — `get_portfolio`, `get_equity_positions`, `get_equity_quotes`, `get_equity_orders`, `get_index_quotes`, `get_realized_pnl`; never a write/order-placement tool), the FRED pull (`agent/fetch_context.py` in the finance-projects repo, or direct FRED MCP calls if available), and WebSearch for catalyst research on any notable move.
- Explicit instruction: output ONLY a JSON file matching `research-schema.md`'s shape, written to a scratch path (e.g. `<scratchpad>/research-brief-<date>.json`). No narrative prose, no draft copy, no HTML. Every catalyst needs a real source URL; every macro figure needs a real `as_of` date from this session's actual pull.
- Explicit instruction: do not touch `index.html`, `rss.xml`, or `sitemap.xml`.

Read the resulting JSON file back before moving to Stage 2 — confirm it's valid JSON and every field that should have a source actually has one. If a required field is missing or a source is fabricated-looking, send it back to a second Research pass rather than letting Stage 2 draft from incomplete data.

## Stage 2 — Writer agent

Spawn via the `Agent` tool, `subagent_type: general-purpose`, **foreground**.

Prompt the agent with:
- The research brief JSON from Stage 1 (paste its contents into the prompt, don't just reference the file path — make it impossible to drift from).
- `style-guide.md`'s full content (paste it in) — structural conventions, privacy rules, voice.
- 2-3 excerpts of existing posts from `index.html` (`#insights` or `#econ` section matching the target) for voice-matching — pull these yourself and include in the prompt rather than making the agent go find them.
- Explicit instruction: every number in the draft must trace to a specific field in the JSON brief. Do not invent, round loosely, or add any fact not present in the brief. Do not touch the live site files — write the draft to a scratch file only.
- Explicit instruction: draft in the exact HTML structure from `style-guide.md` (post-title/post-meta/paragraphs for Insights; stat-tiles + post for Econ Briefing).

## Stage 3 — Reviewer agent

Spawn via the `Agent` tool, `subagent_type: general-purpose`, **foreground**.

Prompt the agent with:
- The draft from Stage 2 (full content, pasted in).
- The original research brief JSON from Stage 1 (pasted in) — the Reviewer checks the draft against this, not against its own re-research.
- `style-guide.md`'s full content.
- Explicit instruction to check and report, structured (not prose):
  1. **Number trace**: does every numeric claim in the draft match a field in the brief exactly? List any that don't.
  2. **Privacy scan**: any dollar amount, share count, or portfolio percentage anywhere in the draft? List any hits verbatim with surrounding context.
  3. **Fabrication check**: any claim in the draft that has no corresponding field in the brief at all?
  4. **Structure/voice check**: does it match the style guide's HTML structure and tone?
- Ask for a final verdict: `PASS` or `FAIL` with an itemized list of what needs fixing.

## After Stage 3

- **If FAIL**: send the Reviewer's itemized issues back to a second Writer pass (Stage 2 again, same brief, plus the issue list). Re-run Stage 3 on the revision. Cap at 2 revision rounds — if still failing after that, stop and hand the draft + issues to the user rather than looping indefinitely.
- **If PASS**: do NOT auto-publish. This is public-facing content — present the draft to the user for a final human look before touching `index.html`/`rss.xml`/`sitemap.xml`, same human-in-the-loop standard as every other push to this repo. Once approved, make the actual edits, commit, and push exactly as done manually today (see `[[project_public_trade_site_idea]]` conventions if picking this up in a fresh session with no memory of prior sessions).

## What this does NOT do

- Does not touch the private `portfolio_dashboard.html` or its repo — that stays a manual/direct-session process, this skill is public-site-only.
- Does not run on a schedule — this is human-triggered, same constraint documented in the econ-briefing-agent work (the scheduled-routine path hit an unresolved GitHub connector access issue before; don't re-attempt that without new information).
- Does not skip the human approval step, even on a clean PASS — public content always gets a final look before it goes live.
