---
name: mobile-ua-creative-strategy
description: >-
  Mobile game UA (user acquisition) ad creative strategy methodology — hook
  taxonomy, creative testing cadence, trend reading, and production briefing.
  Use when the user asks for a "creative strategy", "UA strategy", "买量策略",
  "素材测试计划", "hook 怎么写", "怎么起量", launch/soft-launch creative plans,
  or wants to build a creative testing roadmap for a mobile game. Pairs with
  CreativeScope MCP tools for evidence.
---

# Mobile UA Creative Strategy

A methodology playbook for planning and iterating mobile game ad creatives.
Use CreativeScope tools to ground every claim in live market evidence — this
skill provides the framework, the tools provide the proof.
For tool parameters, filter values, and empty-result debugging, follow the
**creativescope-best-practices** skill.

## Part 1: Hook taxonomy (first 3 seconds)

Classify every hook you write or analyze into one of these families:

1. **Fail / challenge** — player fails spectacularly ("99% can't beat level 3")
2. **Fake gameplay / misleading-core** — ad shows a simpler or different
   mechanic than the store page (high CTR, watch retention/regulation risk)
3. **UGC testimonial** — creator-style reaction, "I didn't believe it either"
4. **Before/after transformation** — weak base → empire, ugly → glamorous
5. **Comparison / choice** — two doors, "which would you pick", vs battles
6. **Satisfying / ASMR** — merge chains, cleanup, perfect sort loops
7. **Narrative twist** — story cliffhanger, betrayal, "wait for the end"
8. **IP / celebrity / seasonal** — borrowed familiarity, holiday tie-ins

When analyzing creatives (via `get_creative_detail` hook signals), map the
signals onto these families so patterns become comparable across advertisers.
**Respect hook confidence:** If `hook_confidence` is low or none, flag it as
uncertain — do not present a low-confidence signal as established fact.

## Part 2: Testing cadence (the 2-week sprint)

Standard early-stage structure; scale budgets to the user's reality:

- **Week 1 — explore:** 4–6 concepts × 2–3 hook variants each, broad
  placements, small equal budgets. Kill criterion: CTR clearly below the
  genre norm you established from rankings data.
- **Week 2 — exploit:** put 70–80% of budget behind the top 2 concepts,
  iterate winning hooks (new scenes, same hook family), retire the rest.
- **Rhythm:** new concepts enter every week; winners graduate to "evergreen"
  rotation and get re-cut monthly to fight creative fatigue.

## Part 3: Evidence gathering with CreativeScope

Before recommending anything, pull current data:

> **Plan wall:** Every tool in this part except `get_creative_rankings` is
> Pro-tier. On `PLAN_UPGRADE_REQUIRED`, say so honestly with the upgrade link
> and deliver the framework labeled "methodology only, not market-verified" —
> never fill the evidence sections with invented data.

1. `get_creative_rankings` (`type: "rising"`, `period: "week"`, user's genre)
   → what hook families are climbing *right now*.
   - Note: rankings support **no platform or country filters** — if the user
     asks for "US Facebook rising", explain the limitation and provide the
     genre-wide rising list instead.
2. `get_creative_rankings` (`type: "evergreen"`, `period: "month"`) → which
   patterns survive fatigue. **Evergreen is month-only**; day/week periods are
   auto-normalized to month (disclosed in `request_echo`). Say so if relevant.
3. `generate_weekly_creative_brief` (genre/platform/country as relevant) →
   condensed market read. **Treat this as a draft** — verify pattern counts
   before stating them as fact. Do not copy template phrasing verbatim.
4. Competitor check if the user names rivals: `search_advertisers` →
   `get_advertiser_profile` → their creative volume/markets. If analyzing
   their creatives, always **filter `search_creatives` results by `advertiser_id`**
   and cross-validate with `landing_url` package names.
5. `get_creative_detail` on 3–5 exemplars → hook signals mapped to Part 1
   families. Include `media_url` links so the user can view the actual creatives.

Genre matters: match-3/casual leans satisfying + fail hooks; SLG leans
narrative + transformation; RPG leans IP/character + comparison. State which
genre assumption you're using.

## Part 4: Output format

```markdown
# UA Creative Strategy — <game/genre> · <market> · <date>

## 1. Market read
What's rising, what's evergreen (evidence from rankings/brief).
Wording: "Based on N representative creatives..." — never imply sample = market.

## 2. Hook strategy
2–3 hook families to lead with, mapped to genre fit + live examples.
Include `media_url` links for cited exemplars.

## 3. Two-week testing plan
Concept list (each = hook family + core scene + CTA), budgets as %,
kill/scale criteria.

## 4. Measurement
Which metrics decide kill vs scale at each stage.

## 5. Production brief (for designers)
Per concept: hook (0–3s), scene beats, format/orientation, CTA.
Include `media_url` reference links where applicable.

## 6. Caveats
- Hook signals: confidence level noted per example; low/none = flagged, not fact.
- `scene_breakdown`: metadata boilerplate, not authentic shot breakdown.
- Time window: "last_7/30/90_days" = active during window, not "newly launched".
- Sample framing: N exemplars analyzed; frequencies are sample-observed, not
  market-distributed.
```

## Guidelines

- Every strategic claim cites evidence from a tool call this session —
  no evidence, no claim. If tools are unavailable, deliver the framework
  clearly labeled as "methodology only, not market-verified".
- Ask (or infer and state) the genre, target markets, and platforms before
  writing the plan — a match-3 US plan and an SLG JP plan differ fundamentally.
- Flag compliance risk when recommending fake-gameplay-style hooks;
  suggest "adjacent-but-real" variants where possible.
- Keep the plan producible: a designer should be able to start from section 5
  without a follow-up meeting.
- **Multi-modal deep-dive (optional):** If the environment supports shell +
  ffmpeg + a vision model, run the visual deep-dive on the top 3 exemplars to
  enrich the hook analysis — procedure in `creativescope-best-practices` →
  `references/visual-deep-dive.md`. **If you did not watch the video, do not
  write shot-level breakdowns — stick to metadata signals.**
