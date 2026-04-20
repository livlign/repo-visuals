# repo-visuals

A Claude Code skill for producing hero visuals — **animated GIF** or **static PNG** — for GitHub repositories. Later: social cards, MP4 demos.

The skill scans the repo, recommends a format (animated vs static) based on the repo's identity, and lets the user pick. It then generates bespoke HTML per repo via a structured discovery dialog, previews it in the browser, and exports to an optimized GIF or retina PNG.

## Layout

- [`SKILL.md`](./SKILL.md) — the main skill definition (discovery → build → preview → export → output → evaluate).
- [`craft/`](./craft/) — craft library referenced by the skill during builds.
  - [`craft/headlines.md`](./craft/headlines.md) — headline patterns (imperative-plus-invariant, narrative arc), voice rules, anti-patterns.
  - [`craft/reference-gallery.md`](./craft/reference-gallery.md) — catalogued archetypes from real-world repo heroes (freeCodeCamp, shallow-backup, amplication, etc.); consulted during Phase 1.4c format recommendation.
  - [`craft/templates/`](./craft/templates/) — full working heroes from past runs; reference them to see how scene systems compose.
- [`scripts/`](./scripts/) — export pipeline + evaluator. `capture.js` (animated: Puppeteer `Page.startScreencast` + ffmpeg palette recipe); `screenshot.js` (static: Puppeteer `page.screenshot` @2x); `evaluate.js` (Phase 6 code-evaluated scorecard rows, format-aware).
- [`repo-visuals-retro/`](./repo-visuals-retro/) — the retrospective meta-skill that reads evaluation logs and proposes edits to `SKILL.md`.
- [`work/`](./work/) — scratch working dirs per repo (git-ignored noise removed, HTML/GIF kept).
- `evaluations/runs/` — gitignored per-run raw scorecards.
- `evaluations/index.md` — curated aggregate (committed, public).

## Dependencies

- Node.js
- `puppeteer` (auto-installs ~170 MB Chromium on first run)
- `ffmpeg` + `ffprobe` (system or portable download — see `SKILL.md` §4.1)
- `gifsicle` (optional — enables palette-size check in `scripts/evaluate.js`)

## Status

Early. Dogfood-in-progress. First real run: `work/emtyty-devtool/` — see its `hero.gif` for what this skill currently produces.
