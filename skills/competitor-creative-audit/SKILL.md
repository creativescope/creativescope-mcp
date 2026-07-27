---
name: competitor-creative-audit
description: >-
  Audit a competitor mobile game or studio's advertising creative strategy using
  CreativeScope MCP tools. Use when the user names a game, studio, or advertiser
  and asks things like "analyze their creatives", "what ads are they running",
  "competitor creative analysis", "拆解竞品素材", or wants a creative strategy
  report on a specific advertiser.
---

# Competitor Creative Audit

Produce a structured creative-strategy report for one advertiser (game or studio)
using CreativeScope tools. Work through the phases in order; do not skip phase 1.
For tool parameters, filter values, and empty-result debugging, follow the
**creativescope-best-practices** skill.

> **Plan wall:** Every tool in Phases 1–3 is Pro-tier (`get_creative_rankings`
> in Phase 4 is the only Free one). On `PLAN_UPGRADE_REQUIRED`, state it
> honestly with the upgrade link, deliver only what Free allows, and label the
> blocked sections — never fill them with invented data.

---

## Phase 1: Advertiser Landscape

### Step 1 — Resolve the advertiser

Call `search_advertisers` with the user's game/studio name as `query`.
- If several candidates come back, pick the best match and state which one you
  chose (name + `advertiser_id`). Ask the user only if genuinely ambiguous.
- If nothing matches, try a shorter/alias name once, then tell the user the
  advertiser was not found and suggest checking spelling or region.

Save both `advertiser_id` and `game_id` — they are interchangeable for the
ranking tools.

### Step 2 — Pull the profile

Call `get_advertiser_profile` with the `advertiser_id`.
Extract: recent ad activity, creative volume, target countries, platforms.
These become the "Overview" section of the report.

---

## Phase 2: App Store Performance

> CreativeScope now includes App Store / Google Play ranking data. Use the two
> tools below to paint the game's store performance picture.

### Step 3 — Ranking markets overview

Call `get_game_rank_markets` with the `game_id` from Step 1.
- Default: `store: "all"`, `rank_types: ["free","store_paid","grossing"]`.
- Default date range is last 90 days; do not exceed 365 days.

Extract for the report:
- **Which markets** the game is charting in (country + category + store)
- **Best rank achieved** in each dimension
- **Latest rank** and how many days it has held a rank
- Dominant store (Google Play vs iPhone App Store) and rank type (free / paid / grossing)

### Step 4 — Ranking trend deep-dive

Call `get_game_rank_trend` with `mode: "best"` (default) to get the game's
best daily rank across all supported countries and categories over the date
range. This gives a single performance curve.

If the user cares about a **specific country + category**, call it a second
time with `mode: "exact"`, `country`, and `category`.

Extract for the report:
- Overall trend: rising / flat / declining?
- Any notable spikes (e.g., post-update, post-campaign blitz)
- Free vs paid vs grossing split: where is the game actually monetizing?

> **Caveat:** iPad App Store rankings are **excluded**. Only Google Play and
> iPhone App Store data are available.

---

## Phase 3: Creative Portfolio Analysis

### Step 5 — Sample their creatives

`search_creatives` has **no advertiser filter** — do not invent one. Instead call
`search_creatives` with `query` set to the game/studio name, plus any filters the
user asked for (`platform`, `country`, `format`, `date_range`; `limit` up to 20).
If the user gave no time frame, default to `date_range: "last_30_days"`.

> **Window semantics:** `last_30_days` means **active during the window**, not
> "newly launched" — a creative's `start_date` may be far earlier. Use
> "currently active" wording in the report.

**Critical filtering rule:** After `search_creatives` returns results, **filter
by the `advertiser_id` from Step 1**. Discard any creative whose `advertiser_id`
does not match — these are off-target matches from the keyword search. Report
how many were discarded and why.

**Sampling strategy:** Sort the filtered results by exposure (impressions/views)
and sample in tiers:
- **Head (top 1–2):** Examine scale — what is the dominant format and hook?
- **Mid (next 3–5):** Look for experimentation — variants, new angles, formats
  the advertiser is testing.
Pick 3–5 total. For each selected creative, call `get_creative_detail` to get
hook signals and creative patterns.

**Cross-validation:** Compare the `landing_url` domain or app package name from
the detail result against the `package_names` from the advertiser profile
(Step 2). Flag any mismatch — it may indicate a misattributed creative.

### Step 6 — Visual deep-dive (when environment supports it)

If the client has shell access, ffmpeg, and a vision model, run the visual
deep-dive on the top 3 creatives to add frame-level observations — full
procedure in `creativescope-best-practices` → `references/visual-deep-dive.md`.

> **Mandatory rule:** If you did not run this pipeline, do not write detailed
> shot-by-shot breakdowns — stick to metadata signals (`hook`, `scene_breakdown`
> with appropriate caveats).

---

## Phase 4: Market Context (optional)

### Step 7 — Genre creative rankings

If the user wants market context, call `get_creative_rankings` with
`type: "rising"`, `period: "week"`, and the game's `genre`. Note whether the
competitor appears and which competing creatives are out-ranking theirs.
If the user explicitly asks to skip market context, omit this step.

> Note: `get_creative_rankings` measures **creative performance** (which ads are
> gaining traction), not **app store download rank**. The two are related but
> not the same — a game can have weak creatives but strong organic/ASO-driven
> store rank, or vice versa.

---

## Output format

```markdown
# Creative Audit — <Advertiser/Game name>

## 1. Overview
Markets, platforms, creative volume, recent activity (from profile).

## 2. Where they run
Platform split + target countries, in one compact table.

## 3. App Store performance
### 3a. Chart presence
Which countries/categories/stores the game is ranking in, best/latest rank,
ranked-day count.

### 3b. Trend
Best-rank daily curve over last 90 days. Notable spikes, trend direction.
Free vs grossing split: where is the monetizing power?

## 4. What their creatives look like
Formats/orientation mix, recurring themes, art style.

## 5. Top creatives & why they work
One short block per creative: hook signal, scene pattern, what makes it work.
Include `media_url` link for each (stable OSS direct link, downloadable).
If visual deep-dive was performed, add frame observations.

## 6. Patterns & gaps
3–5 bullet takeaways: what they repeat, what they under-use, and 2–3 concrete
angles the user could test against them.

## 7. Method & caveats
- Sample: N creatives analyzed from M returned (after filtering out X off-target).
- Time window: "last_30_days" = currently active during the window, not "newly launched".
- Hook confidence: stated per creative; low/none confidence hooks are flagged, not treated as fact.
- Scene descriptions (`scene_breakdown`) are metadata boilerplate — do not quote them as authentic shot breakdowns.
- Thumbnail URLs expire in ~15 minutes; use `media_url` for persistent access.
- App Store data: iPad rankings excluded; only Google Play + iPhone App Store.
```

## Guidelines

- Keep the report skimmable: tables over paragraphs, max ~500 words before the
  "Top creatives" section.
- Never fabricate metrics. If a field is missing from a tool result, say
  "not available" rather than guessing.
- Send queries in the user's original language; the service expands concepts
  internally — do not append translations.
- **Narrow questions trigger partial execution:**
  - "Which platforms is X scaling on?" → answer from Phase 1 only.
  - "What's Royal Match's store rank in Japan?" → run Phase 1 + Phase 2 only.
  - "Show me their top creatives" → Phase 1 + Phase 3 only.
