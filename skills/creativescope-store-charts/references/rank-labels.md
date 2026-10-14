# Store ranking labels (Chinese → English)

The two store-ranking tools return `rank_name` and `category` as Chinese
strings. Translate to English before quoting in English-language output. Keep
the original in parentheses if the user might want to search for it on
Chinese-language stores.

## Rank names (`rank_name`)

| Chinese | English |
|---|---|
| 全部 | All |
| 免费榜 | Free |
| 商店付费榜 | Store Paid |
| 畅销榜 | Grossing |

## Categories (`category`)

Categories vary by store and country. The MCP server returns the localized
label. Common ones observed:

| Chinese | English |
|---|---|
| 全部 | All |
| 益智 | Puzzle |
| 休闲 | Casual |
| 策略 | Strategy |
| 冒险 | Adventure |
| 动作 | Action |
| 模拟 | Simulation |
| 角色扮演 | Role Playing (RPG) |
| 棋牌 | Board / Card |
| 家庭聚会 | Family |
| 音乐 | Music |
| 体育 | Sports |
| 文字 | Word |

> The exact category set depends on the store (App Store and Google Play
> don't use the same categories). If the server returns a label not on
> this list, translate by meaning and add a footnote.

## Tips

- A "Best" rank in a `get_game_rank_trend` response is across all categories
  in that country/store. If the user wants the category-specific trend,
  pass `category` explicitly in `mode: "exact"`.
- The `mode: "best"` response carries the winning `category` per point —
  use it, don't guess.
- Never paste raw Chinese labels into an English report. Translate
  consistently — if a report mentions "畅销榜" five times, write
  "Grossing" five times (or, on first use, "Grossing (畅销榜)").

## How to translate consistently

If you need to add a label not on this list:

1. Use the same translation style as the entries above (short noun phrase,
   capitalized).
2. Add it to this file with both the Chinese and English on a single row.
3. Reference the change in your commit message so other agents stay
   consistent.
