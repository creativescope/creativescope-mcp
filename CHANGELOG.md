# Changelog

All notable changes to this repository are documented here. Dates are UTC.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned

- README screenshot (workflow diagram)
- More example outputs in the README ("what the tools return")
- Localisation of remaining `ai_analysis` Chinese strings
- New skill: TBD (track via issues)

## [1.1.0] - 2026-10-14

### Added

- **New skill: `creativescope-store-charts`** — workflow, decision table, 6 store-chart-specific pitfalls, and a Chinese→English `rank_name` / `category` label reference (`references/rank-labels.md`) for the `get_game_rank_markets` and `get_game_rank_trend` tools.
- Release v1.1.0 on GitHub.

## [1.0.0] - 2026-10-14

### Added

- Initial public release.
- 12 MCP tools exposed via `https://mcp.creativescope.ai/mcp` (Streamable HTTP, OAuth or API key).
- 5 Agent Skills under `skills/`:
  - `creativescope-best-practices` — foundation layer with 12 cross-tool pitfalls
  - `competitor-creative-audit` — full creative-strategy audit
  - `weekly-inspiration-brief` — shareable weekly creative brief
  - `image-similarity-scan` — async reference-image similarity search
  - `mobile-ua-creative-strategy` — UA creative strategy playbook
- `server.json` registered on the official MCP Registry (`ai.creativescope/creative-intelligence`).
- `glama.json` registered for Glama directory ownership.
- `LICENSE` (MIT) for the listing / metadata.
- Repository topics (14) for discoverability.
- About-page description and homepage.

[Unreleased]: https://github.com/creativescope/creativescope-mcp/compare/v1.1.0...HEAD
[1.1.0]: https://github.com/creativescope/creativescope-mcp/releases/tag/v1.1.0
[1.0.0]: https://github.com/creativescope/creativescope-mcp/releases/tag/v1.0.0
