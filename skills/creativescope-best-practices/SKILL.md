---
name: creativescope-best-practices
description: >-
  Best practices and pitfalls for querying the CreativeScope MCP server (mobile
  game ad creative intelligence). Use whenever working with CreativeScope tools
  and unsure which tool fits the task, how to write an effective query, what
  filter values are valid, or when a search returns empty/unexpected results
  and needs debugging.
---

# CreativeScope Best Practices

Reference card for using the 12 CreativeScope tools correctly. Load this before
blaming the data — most "no results" cases are query problems.

## Pick the right tool first

| User intent | Tool |
|---|---|
| Find creatives by keyword / concept / visual description | `search_creatives` |
| Deep-dive ONE creative (hook signals, scenes, why it works) | `get_creative_detail` |
| What's rising / what keeps working (creative leaderboards) | `get_creative_rankings` |
| More creatives like one I already have | `find_similar_creatives` (instant) |
| Match MY image against live ads | `submit_reference_image_search` (async — see image-similarity-scan skill) |
| Find an advertiser / studio / game | `search_advertisers` |
| An advertiser's markets, platforms, volume | `get_advertiser_profile` |
| Ready-made weekly inspiration roundup | `generate_weekly_creative_brief` |
| **App Store / Google Play ranking markets (where is the game charting?)** | **`get_game_rank_markets`** |
| **App Store / Google Play daily ranking trend over time** | **`get_game_rank_trend`** |

Do not improvise cross-purpose calls (e.g. do not use `search_creatives` to
profile an advertiser — chain `search_advertisers` → `get_advertiser_profile`).

## Query-writing rules

- Send queries in the **user's original language**, concepts only. The service
  expands genre/hook/mood/art-style concepts internally. **Never append
  generated translations** to the request.
- Natural language works: "UGC-style video of a player failing at a merge
  puzzle" is a valid `query`.
- For `find_similar_creatives` with a concept (no `creative_id`), send the
  concept as `query`; the service picks an explainable seed itself.

## Filter cheatsheet

- `platform`: facebook, instagram, messenger, audience_network, **threads,** tiktok,
  youtube, admob, **mintegral, unity, applovin, ironsource, vungle, pangle**
  (exact set may grow — if a value errors, drop it).
- `country`: ISO 3166-1 alpha-2 (US, JP, KR, GB…), not full names.
- `date_range`: `last_7_days` / `last_30_days` / `last_90_days` or
  `YYYY-MM-DD..YYYY-MM-DD`.
- `limit`: max **20** everywhere (creative tools); max **200** for
  `get_game_rank_markets` (defaults to 100).
- `format`: image / video / playable. `orientation`: portrait / landscape /
  square.

## App Store ranking tools

- `get_game_rank_markets`: Lists countries, categories, stores, and rank types
  (free / store_paid / grossing) where a game appeared on a chart. Returns best
  rank, latest rank, and ranked-day count per dimension.
  - `game_id`: required. Same as `advertiser_id` from `search_advertisers`.
  - `store`: `all` (default) / `google_play` / `app_store`.
  - `rank_types`: array of `free`, `store_paid`, `grossing`. Defaults to all three.
  - Date range defaults to last 90 days; **cannot exceed 365 days**.
  - Default `limit` is 100 (max 200). If the returned item count equals the
    limit, the list may be truncated — disclose "at response cap" in the report.
  - **iPad App Store is excluded.** Only Google Play + iPhone App Store.

- `get_game_rank_trend`: Daily ranking series over a date range.
  - `mode`: `best` (default) — best daily rank across all countries/categories.
    Each point carries the `country` / `category` / `store` that achieved that
    day's best rank — the winning market **varies day to day**, so do not read
    one series as one market's trajectory.
    `exact` — one specific country + category combination.
  - For `exact` mode: provide `country` (ISO, defaults US) and `category`.
  - Date range defaults to last 90 days; **cannot exceed 365 days**.
  - Returns one series per rank type under `data.series`.
  - **A rank type the game is not charting on returns an empty series**:
    `points: []`, `ranked_days: 0`, and **no `best_rank` / `latest_rank` keys
    at all**. Absence of these keys is not an error — it means "not charting".

> **Rank labels come back in Chinese.** `rank_name` and `category` are
> Chinese strings: 畅销榜 = grossing, 免费榜 = free, 商店付费榜 = store paid;
> 所有 = All apps, 益智 = Puzzle, 休闲 = Casual. Normalize to English before
> quoting them in a report — never paste the raw Chinese labels into
> English-language output.

> **Creative rankings ≠ App Store rankings.** `get_creative_rankings` measures
> ad creative traction (which ads are gaining impressions). `get_game_rank_*`
> measures store chart position (downloads / revenue). They are related but not
> interchangeable — do not confuse them in reports.

## Known pitfalls (learned the hard way)

1. **`search_creatives` has NO advertiser filter.** To get one advertiser's
   creatives, query by the game/studio name instead. **Always filter the
   returned results by `advertiser_id` from `search_advertisers`** — discard
   off-target matches and report how many were dropped.
2. **`get_creative_rankings` supports NO platform or country filters** —
   only `type`, `period`, `genre`, `limit`. Don't ask it for "US Facebook
   rankings"; it can't.
3. **Evergreen rankings are month-only.** Day/week periods are auto-normalized
   to month and disclosed in `request_echo` — tell the user when that happens.
   **`RANKING_NOT_FOUND` means no ranking issue matched your inputs** — most
   often a non-canonical `genre` slug. Retry with a short canonical slug
   (e.g. `match-3`, not "match clearing"), then check `resolved_genre` in
   `request_echo` to see what it normalized to. If it still fails, say so
   honestly — do not substitute external data.
4. **Image search is async and URL-strict** (public HTTPS, raw image bytes,
   no login/hotlink protection). Follow the image-similarity-scan skill; never
   poll in a tight loop.
5. **Empty results → widen in this order:** drop `country` → drop `format` →
   drop `platform` → broaden the query → widen `date_range`. State which
   filter you relaxed.
6. **Creative tag fields come back normalized to English** in brief/profile
   results — keep them English even in non-English conversations.
7. IDs are chainable: `creative_id` from any search/rankings tool works in
   `get_creative_detail` and `find_similar_creatives`; `advertiser_id` (or
   `game_id` in creative results) works in `get_advertiser_profile`,
   `get_game_rank_markets`, and `get_game_rank_trend`.
8. **`date_range` means "active during the window"** (filtered by end_date /
   active days), NOT "newly launched". A creative's `start_date` may be far
   earlier than the window. Never write "newly launched in the last 30 days"
   when you mean "currently active in the last 30 days".
9. **`thumbnail_url` expires in ~15 minutes.** For persistent access, use
   `media_url` (OSS direct link, stable, ~5 MB video / 60 s typical).
10. **`hook` fields carry honesty signals:** `hook_source`
    (source_label / metadata_inference / unavailable) + `hook_confidence`
    (high / low / none). **If confidence is low or none, do not present the
    hook as fact.** Flag it: "Hook signal present but confidence is low."
11. **`scene_breakdown` is metadata boilerplate.** Multiple creatives from the
    same advertiser may share identical `scene_breakdown` text. Do not quote it
    as an authentic shot-by-shot breakdown. For real visual analysis, use
    multi-modal frame extraction (see below).
12. **`request_echo` is your debug friend.** Every tool response includes
    `request_echo` showing exactly what was sent. Use it to diagnose unexpected
    results before blaming the API.

## Honesty rules

- Never invent metrics, creative titles, or advertisers not present in tool
  results. Missing field = say "not available".
- **"No data" ≠ "broken path."** Before concluding that data does not exist,
  check `request_echo` for what was actually sent, relax filters per pitfall
  #5, and try the chainable-ID route. Report "no results after widening X and
  Y", not just "no results" — the distinction decides whether the user trusts
  the empty answer.
- One image = one async job; track each `job_id` explicitly.
- **Credibility over completeness:** If data is weak, partial, or contradictory,
  say so. A thin but honest report beats a confident fabrication. As a rule of
  thumb: "宁可信空，不可信凑" — better to report emptiness than to pad with
  guesses.

## Multi-modal deep-dive (when environment supports it)

Metadata labels (`hook`, `scene_breakdown`) are a fraction of what a creative
actually contains — frame-level analysis is routinely an order of magnitude
richer. If the client has shell access, ffmpeg, and a vision model, run the
visual deep-dive on the top creatives (default: top 3, to control context).

**Full procedure: see `references/visual-deep-dive.md`.**

> **Mandatory guardrail:** If you did not run this pipeline, do not write
> detailed shot-by-shot descriptions. Stick to metadata signals (`hook`,
> `scene_breakdown` with the boilerplate caveat above).

## Free vs Pro tier behavior

- **Free tier:** Only `get_creative_rankings` is available. All other 11 tools
  return `PLAN_UPGRADE_REQUIRED`.
- When you hit `PLAN_UPGRADE_REQUIRED`, **state it honestly** and provide the
  upgrade link. Do **not** attempt to work around it, do **not** fabricate
  results, and do **not** pretend the tool returned data.
- Design pattern (good example): Return a structured error with
  `reason_code` + `recommended_action` + `fallback_policy` so the user knows
  exactly what happened and what to do next.
