---
name: image-similarity-scan
description: >-
  Run a reference-image similarity search against live mobile game ads using
  CreativeScope's asynchronous image-search tools. Use when the user provides
  an image (URL) and asks to "find similar ads/creatives", "以图搜图", "search
  by image", or wants to know who is running ads that look like their reference
  creative. Handles the full async job lifecycle: submit → poll → results.
---

# Reference-Image Similarity Scan

CreativeScope's image search is **asynchronous**: submit a job, poll its status,
then fetch results. Follow this lifecycle exactly — never call the results
endpoint without submitting first, and never submit twice for the same image.
For general tool/filter rules, follow the **creativescope-best-practices** skill.

## Step 0: Validate the image URL (before submitting)

`submit_reference_image_search` requires a **directly downloadable public HTTPS
image URL**. It must return raw image bytes. Before submitting, check:

- ❌ Local file paths, base64 blobs → not supported. If the user only has a
  local file, ask them to host it (or host it for them if you have a file
  tool/upload channel available) and come back with an HTTPS URL.
- ❌ Google Drive / Dropbox / WeChat / Notion "share" links → these return HTML
  pages, not image bytes. Ask for a direct link (e.g. Dropbox `dl=1`) or a
  re-host.
- ❌ URLs behind login or hotlink protection.

If unsure, fetch the URL headers yourself first when you have a fetch tool:
the response must be `Content-Type: image/*` with HTTP 200. If you cannot
verify, submit once and let the service validate — then handle the failure
in step 2.

## Step 1: Submit the job

Call `submit_reference_image_search` with:
- `image_url` (required)
- Optional filters the user asked for: `country` (ISO code), `genre`,
  `platform`, `format`
- `limit`: default 10 (max 20)

**Plan wall:** The image-search tools (submit / status / results) and
`find_similar_creatives` are Pro-tier. On `PLAN_UPGRADE_REQUIRED`, say so
honestly and provide the upgrade link — do not simulate results with keyword
search and present them as image matches.

Save the returned `job_id`. Tell the user the search started and typically
takes under a minute — do not stay silent while polling.

## Step 2: Poll status — patiently

Call `get_reference_image_search_status` with the `job_id`.
- Poll at most every **10–15 seconds**, up to ~3 minutes total. Do not
  hammer the endpoint in a tight loop.
- If the status response includes polling guidance, follow it.
- If the job fails, read the `error_message` and act generically:
  - URL-related error ("fetch", "download", "URL") → go back to Step 0 and
    get a better URL from the user.
  - Image-format-related error ("format", "size", "invalid image") → ask
    for a different image (standard JPG/PNG, reasonable dimensions).
  - Transient/infrastructure error ("timeout", "queue", "retry") → one
    retry is reasonable; if it fails again, report the error and stop.
  - Do **not** hard-code specific error-code strings — the API may change
    them. Use the `error_message` text heuristically.
- If still running after ~3 minutes, report the job is taking long, give the
  user the `job_id`, and offer to check again later instead of polling forever.

## Step 3: Fetch and present results

When status shows complete, call `get_reference_image_search_results` with the
`job_id` (and the same `limit` if relevant).

**Present results honestly:**
- Results are returned in similarity order (first = most similar). **Do not
  invent percentage scores or numeric similarity values.**
- Cross-format matching is normal (e.g., an image seed can return video
  results) — do not flag this as unexpected.
- **Review each result for actual relevance.** If a result is clearly off-topic
  (e.g., a kids' song video returned for a match-3 search), discard it and
  note the discrepancy. If *most* results are irrelevant, say so honestly:
  "The search returned N results, but the majority do not appear visually
  related to your reference." Offer to widen filters or try a different seed.
- **Optional visual verification:** relevance drift usually happens across
  formats (image seed → video results). If the client has shell + ffmpeg + a
  vision model, you may visually check borderline matches — procedure in
  `creativescope-best-practices` → `references/visual-deep-dive.md`.

Present as a compact table or list: matched creative, advertiser/game,
platform, format, and **include `media_url` link for each** (stable OSS direct
link). Then offer the natural follow-ups:
- "Want a deep dive on any of these?" → `get_creative_detail`
- "Want more like the best match?" → `find_similar_creatives` with that
  `creative_id` (note: this second tool is instant — it uses the stored vector
  of an existing creative, no async job needed)

## Guidelines

- One image = one job. If the user has several images, submit them as separate
  jobs and track each `job_id` explicitly.
- If the user gives a *concept* instead of an image ("find ads like a match-3
  merge ad"), do **not** use this skill's submit flow — use
  `find_similar_creatives` with `query`, which picks an explainable seed and
  returns results immediately.
- Never fabricate matches. If the completed job returns few or zero results,
  say so and suggest widening filters (drop `country`/`genre`).
