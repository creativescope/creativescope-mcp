---
name: creativescope-store-charts
description: >-
  Look up where a mobile game is charting on the App Store and Google Play —
  which countries, categories, stores, and rank types (free, store paid, grossing)
  it appears on, plus daily ranking trends over time. Built on the CreativeScope
  MCP server's `get_game_rank_markets` and `get_game_rank_trend` tools. Use this
  whenever the user wants store-ranking context for a game (e.g. "is this game
  charting in Japan?", "show me the US iPhone grossing trend for the last 30
  days") and you need to avoid the pitfall of confusing creative rankings
  (ad traction) with store chart positions.
---

# CreativeScope Store Charts

End-to-end workflow for answering "where is this game charting?" and "how has
its ranking changed over time?" using the two CreativeScope MCP store-ranking
tools. Load `creativescope-best-practices` first if you're not yet sure which
tool fits — this skill assumes the foundation layer.

## Pick the right tool

| User intent | Tool |
|---|---|
| Which countries / categories / stores / rank types is a game charting on? | `get_game_rank_markets` |
| How has a game's chart position changed day-by-day in one market? | `get_game_rank_trend` (mode `exact`) |
| Best daily rank across ALL markets (winning market varies by day) | `get_game_rank_trend` (mode `best`, default) |

Never mix up store rankings (`get_game_rank_*`) with creative rankings
(`get_creative_rankings`) — they measure different things. See pitfall #1 in
`creativescope-best-practices`.

## Workflow

1. **Resolve the game's `game_id`.** This skill needs a `game_id`, not a free-text
   name. If you only have a name, use `search_advertisers` to look it up and
   pick the matching `advertiser_id` (== `game_id` in creative results). Confirm
   the result is the right game (publisher, store listings) before proceeding.

2. **Decide the question shape.**
   - "Where is it charting?" → `get_game_rank_markets`
   - "How did it trend in one market?" → `get_game_rank_trend` with
     `country` + `category` (use `mode: "exact"`)
   - "What's its best recent position across all markets?" → `get_game_rank_trend`
     default (`mode: "best"`)

3. **Set the date window.** Defaults are last 90 days; max 365 days. State the
   window you used.

4. **Call the tool. Read the response carefully:**
   - `get_game_rank_markets`: each item lists one (country, category, store,
     rank_type) combination with `best_rank`, `latest_rank`, and
     `ranked_days_count`. If the returned item count equals your `limit`, the
     list may be truncated — disclose "at response cap" in your answer.
   - `get_game_rank_trend`: one series per `rank_type` under `data.series`. A
     rank type the game is NOT charting on returns `points: []`,
     `ranked_days: 0`, and no `best_rank` / `latest_rank` keys at all.
     **Absence of these keys is not an error — it means "not charting"**.
     Don't manufacture a flat line at rank 0.

5. **Normalize labels for English output.** `rank_name` and `category` come
   back in Chinese. Translate before quoting them in English. See the table
   in `references/rank-labels.md`.

6. **State what you did and what you found.** Be honest about the window,
   the market scope, and any truncation at the response cap.

## Pitfalls specific to store charts

1. **iPad App Store is excluded.** Only Google Play + iPhone App Store.
2. **"Best rank" ≠ "current rank".** A `best_rank: 5` from two months ago tells
   you the game hit #5 once; `latest_rank` tells you where it is right now.
3. **`mode: "best"` mixes markets.** Each daily point carries the
   `country` / `category` / `store` that achieved that day's best rank, and
   the winning market varies day to day. Don't read one series as one
   market's trajectory — read it as "the best position the game achieved
   anywhere each day."
4. **`rank_type: "store_paid"` ≠ grossing.** They are different charts. The
   store_paid chart is what you see in App Store Connect under "Top Paid";
   grossing is revenue-based.
5. **Empty series ≠ broken path.** Before reporting "no data", check
   `request_echo` to confirm what was sent, and try widening the date
   window or switching `store` from `app_store` to `google_play` (or vice
   versa).
6. **Don't conflate the game-level chart with the per-creative
   `get_creative_rankings`.** A game can have great creatives and still not
   be charting (and vice versa).

## Honesty rules

- Never invent rankings, markets, or categories not in the response.
- "No data" is a real answer. If the game simply isn't charting on
  `store_paid` for Google Play in DE, say so — don't try to substitute
  grossing data.
- When you truncate at the response cap, say "at response cap" and the
  count you saw.
- When you translate Chinese labels to English, keep the original in
  parentheses if the user might want to search for it.

## Free vs Pro tier behavior

- **Free tier:** only `get_creative_rankings` is available. Both
  `get_game_rank_markets` and `get_game_rank_trend` return
  `PLAN_UPGRADE_REQUIRED` on the free tier. State this honestly and link to
  <https://creativescope.ai/pricing> — do not try to work around it.

## Example prompts

- "Where is Royal Match currently charting? Give me a list of countries."
- "Show me the last 30 days of US iPhone grossing rank for Gossip Harbor."
- "Which rank type is this game best at — free, store paid, or grossing?"

## Data scope

App Store + Google Play, iPhone only on Apple's side. iPad, Mac, and
regional storefronts are not covered. Rankings reflect public chart
position; they are not revenue or download estimates.
