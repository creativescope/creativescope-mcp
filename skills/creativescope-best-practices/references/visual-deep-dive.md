# Visual Deep-Dive: Frame Extraction + Multi-Modal Analysis

Reference procedure for analyzing what a video/image creative *actually*
contains, beyond metadata labels. Load this only when you intend to run the
pipeline — it requires a shell, ffmpeg, and a vision-capable model.

## When to use

- A scenario skill calls for deep analysis of specific creatives (typically the
  top N by estimated exposure; **default N = 3** to control context size).
- Metadata signals (`hook`, `scene_breakdown`) are insufficient or low
  confidence and the question hinges on what is visually happening.

## Prerequisites

- `media_url` from the creative (stable OSS direct link; unlike
  `thumbnail_url` it does **not** expire in 15 minutes).
- ffmpeg on PATH. **No system ffmpeg?** Use the bundled binary:
  ```bash
  pip install imageio-ffmpeg
  python -c "import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())"
  ```
  Use the printed path as the ffmpeg executable.
- A vision-capable model to read the extracted frames.

## Procedure

1. **Download** the creative via `media_url` (typical video: ~5 MB / ~60 s /
   portrait). Playables and images have their own OSS paths; download as-is.
2. **Extract frames** with ffmpeg:
   - Overview grid (whole-video structure):
     ```bash
     ffmpeg -i input.mp4 -vf "fps=1/3,tile=3x3" -frames:v 1 grid.png
     ```
     One 9-frame tile = a coarse storyboard of the full video.
   - Hook analysis (the opening decides the scroll-stop):
     ```bash
     ffmpeg -i input.mp4 -t 3 -vf "fps=1" hook_%d.png
     ```
     Dense 1 fps extraction over the first 3 seconds.
3. **Read the frames with the vision model.** Note: layout/composition,
   on-screen text, UI elements, gameplay vs live-action footage, the
   opening-second pattern, and the ending (CTA, fail-state, reward reveal).
4. **Enrich the report** with what was actually seen, attributed per creative.
   Frame observations outrank metadata labels when they conflict — say so
   explicitly when they do.

## Guardrails (non-negotiable)

- **No pipeline run → no shot-by-shot claims.** If any prerequisite is missing
  (no shell / no ffmpeg / no vision model), fall back to metadata signals and
  state that visual analysis was not performed.
- **Cap the sample.** Default top 3 creatives. Each grid + hook set is real
  context cost; do not batch-analyze 20 videos.
- **`scene_breakdown` stays boilerplate** even after deep-dive — never cite it
  as the source of a visual observation; cite the frames.
- Videos can fail to download (expired/OSS errors). If download fails, say so
  and skip that creative rather than analyzing its thumbnail.
