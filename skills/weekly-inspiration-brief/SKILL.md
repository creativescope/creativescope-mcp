---
name: weekly-inspiration-brief
description: >-
  Generate a shareable weekly ad-creative inspiration brief for mobile games
  using CreativeScope MCP tools. Use when the user asks for a "weekly brief",
  "creative inspiration", "this week's top ads/hooks", "素材周报", "灵感简报",
  or a regular UA/creative team roundup, optionally filtered by genre,
  platform, or country.
---

# Weekly Inspiration Brief

Turn CreativeScope data into a polished, shareable weekly creative brief.
Default scope when the user doesn't specify: their genre if known, all
platforms, `date_range: "last_7_days"`.
For tool parameters and empty-result debugging, follow the
**creativescope-best-practices** skill.

## Step 1: Generate the core brief

Call `generate_weekly_creative_brief` with the user's filters:
- `genre` (e.g. match-3, SLG, puzzle — user's original language is fine)
- `platform`, `country`, `format` only if the user asked
- `limit`: 10 is a good default (max 20)

**Plan wall:** `generate_weekly_creative_brief` is a Pro-tier tool. If it
returns `PLAN_UPGRADE_REQUIRED`, say so honestly, provide the upgrade link,
and deliver only what the Free tier allows (`get_creative_rankings`) — clearly
labeled as a rankings-only brief. Do not fabricate the missing sections.

**Treat tool output as a draft, not gospel.** The tool's pattern descriptions
are metadata templates (`meta.note` often labels them as such). Cross-check
counts and frequencies before stating any pattern as fact. Do not copy template
phrasing verbatim — paraphrase and verify against the data.

## Step 2: Cross-check with rankings

Call `get_creative_rankings` with `type: "rising"`, `period: "week"` (plus
`genre` if the brief is genre-scoped). Use it to:
- confirm which brief creatives are also charting,
- add 2–3 rising creatives the brief may have missed.

If the user asks for evergreen/all-time winners, use `type: "evergreen"` —
note it supports `period: "month"` only (other periods are auto-normalized to
month; mention this if relevant).

## Step 3 (optional): Deep-dive the top 2–3

For the most interesting creatives, call `get_creative_detail` to enrich the
report with hook signals and creative patterns.

**Visual layer (when environment supports it):** If the client has shell +
ffmpeg + a vision model, run the visual deep-dive on the top 2–3 creatives —
procedure in `creativescope-best-practices` → `references/visual-deep-dive.md`.

> **Guardrail:** If you did not run that pipeline, do not write shot-level
> descriptions. Stick to metadata signals (`hook` with its confidence,
> `scene_breakdown` flagged as boilerplate).

## Output format

```markdown
# Weekly Creative Brief — <genre/scope> · <week of YYYY-MM-DD>

## TL;DR
3 bullets: the single biggest pattern this week, the standout creative,
one idea worth testing.

## 🔥 Top hooks this week
Ranked list: hook pattern + one-line why-it-works + example creative.
Base every claim on counts from the tool response — do not quote template text
as if it were observed fact. Wording: "Based on N representative creatives..."
(not "market-wide" or "industry-wide").

## 📈 Rising creatives
Table: creative | game/advertiser | format | why it's rising.
Include `media_url` link for each (stable OSS direct link).

## 🧠 Patterns to steal
3–5 recurring patterns across the data (hook structures, scene types, CTAs).
Again, grounded in counts from the response, not paraphrased template output.

## 🎯 3 ideas to test next week
Concrete, producible concepts — each traceable to evidence above.
```

## Guidelines

- Write for a busy UA/creative lead: skimmable in 2 minutes, TL;DR first.
- Every claim must trace to a tool result — no invented metrics or creatives.
- If the brief tool returns sparse results (narrow filters), widen one filter
  (usually drop `country` or `format`) and say that you did.
- Creative tag fields come back normalized to English; keep them in English
  even if the conversation is in another language, and translate only the
  surrounding prose if the user wants it.
- If the user wants to *send* this somewhere (Slack, email, doc), produce the
  markdown above and ask where it should go before taking further action.
- **Sample framing is mandatory:** Always state "Based on N representative
creatives sampled" — never imply the sample equals the full market distribution.
Extreme outliers can skew observed frequencies.
