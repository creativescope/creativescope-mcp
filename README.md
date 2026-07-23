# CreativeScope MCP

[![Smithery](https://smithery.ai/badge/@creativescope/creative-intelligence)](https://smithery.ai/servers/creativescope/creative-intelligence)
[![Official MCP Registry](https://img.shields.io/badge/Official%20MCP%20Registry-ai.creativescope%2Fcreative--intelligence-blue)](https://registry.modelcontextprotocol.io)

**Mobile-game ad creative intelligence for any MCP-enabled agent.**

CreativeScope is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server that gives your AI agent live access to mobile-game advertising creative intelligence — creative search, Rising/Evergreen rankings, advertiser & studio profiles, similar-creative lookup, and AI hook analysis — compiled from **publicly available ad placements** across major SDK ad networks.

Works with Claude, Codex, Cursor, or any MCP-compatible client. Claude is used only as an example below.

- Website: https://creativescope.ai
- Get an API key (free tier available): https://creativescope.ai/mcp
- Pricing & limits: https://creativescope.ai/pricing

## Why CreativeScope

- **SDK ad-network coverage** — beyond the usual Meta Ad Library: AppLovin, Mintegral, Unity, ironSource, Vungle, Pangle, AdMob, TikTok and more.
- **Game-specialized** — genres, opening hooks, playables, UGC styles, art styles, moods, and creative patterns tuned for UA teams.
- **AI analysis, not just a feed** — representative hooks, confidence-scored insights, and "why it's relevant" reasoning returned as structured data your agent can act on.

## Connecting

CreativeScope is a **remote** MCP server.

- Endpoint: `https://mcp.creativescope.ai/mcp`
- Transport: Streamable HTTP (stateless)
- Auth: **OAuth** (recommended where supported) or API key (`Authorization: Bearer <api_key>`, also accepts `X-Api-Key: <api_key>`)

### Claude Desktop / Claude.ai (OAuth — no key needed)

**Settings → Connectors → Add connector**, paste `https://mcp.creativescope.ai/mcp`, then authorize on the CreativeScope page. You're connected — all tools available immediately.

### API key (other clients)

Get a free key at https://creativescope.ai/mcp — keys look like `cs_live_...`. Keep them secret; don't commit or share them.

### Cursor / Codex (via mcp-remote)

```json
{
  "mcpServers": {
    "creativescope": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.creativescope.ai/mcp",
        "--header",
        "Authorization: Bearer cs_live_YOUR_KEY"
      ]
    }
  }
}
```

Clients that support remote MCP servers natively can point directly at `https://mcp.creativescope.ai/mcp` and set the `Authorization` header to `Bearer cs_live_YOUR_KEY`.

## Tools

| Tool | What it does |
| --- | --- |
| `search_creatives` | Search game ad creatives by natural language, platform, country, genre, format, and date range |
| `get_creative_detail` | Full detail for one creative: delivery data, tag insights, and raw AI analysis |
| `find_similar_creatives` | Find creatives in the same cluster as a given `creative_id` |
| `search_advertisers` | Search advertisers, games, and studios |
| `get_advertiser_profile` | Profile for an advertiser or game |
| `submit_reference_image_search` | Start an async "find similar by reference image" job |
| `get_reference_image_search_status` | Poll a reference-image job's status |
| `get_reference_image_search_results` | Fetch results once a reference-image job completes |
| `generate_weekly_creative_brief` | Generate a weekly creative brief |
| `get_creative_rankings` | Rising / Evergreen creative rankings |

Most list tools return up to 20 items per call. Analysis fields are returned in English; some raw `ai_analysis` fields in `get_creative_detail` may still contain Chinese pending full localization.

## Example prompts

- "Show me the top rising match-3 ad creatives in the US over the last 30 days."
- "Find UGC-style ads with a fail-state opening hook for puzzle games."
- "Profile the advertiser behind this creative and show 5 similar creatives."

## Plans

Free, Pro, and Team plans differ in tool access and monthly quota. See https://creativescope.ai/pricing. Usage is metered per tool call.

## Data

Creative intelligence is compiled from **publicly available ad placements** and concerns businesses and their public advertising activity, not private individuals. Ad creatives, game titles, and trademarks belong to their respective owners. To request review or removal, contact support@creativescope.ai.

## Links

- Privacy Policy: https://creativescope.ai/privacy-policy
- Terms of Service: https://creativescope.ai/terms-of-service
- Support: support@creativescope.ai

## License

MIT — see [LICENSE](./LICENSE). This repository contains the public listing and connection metadata for the CreativeScope MCP service; the hosted service itself is proprietary.
