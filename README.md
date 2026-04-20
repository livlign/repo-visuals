# repo-visuals

A Claude Code skill for producing hero visuals — **animated GIF** or **static PNG** — for GitHub repositories.

The skill scans the target repo, recommends a format (animated vs static) based on the repo's identity, then generates bespoke HTML per repo via a structured discovery dialog, previews it in the browser, and exports to an optimized GIF or retina PNG. The user picks an operating mode at the start (Auto / Semi-auto / Manual) to control how many decisions the skill asks before shipping.

## Getting started

1. Invoke the skill and give it a target: a GitHub URL, a local path, or a free-text brief.
2. Pick an operating mode when prompted.
3. Follow the discovery dialog through to export. The skill handles scan → scenario → brief → build → preview → export → evaluate.

## Layout

- [`SKILL.md`](./SKILL.md) — main skill definition (discovery → build → preview → export → output → evaluate).
- [`craft/`](./craft/) — craft library consulted during builds.
  - [`craft/headlines.md`](./craft/headlines.md) — headline patterns, voice rules, anti-patterns.
  - [`craft/reference-gallery.md`](./craft/reference-gallery.md) — catalogued archetypes from real-world repo heroes; consulted during format recommendation (§1.4c).
  - [`craft/templates/`](./craft/templates/) — full working heroes to reference when composing scene systems.
- [`scripts/`](./scripts/) — export pipeline + evaluator.
  - `capture.js` — animated capture via Puppeteer `Page.startScreencast` + ffmpeg palette recipe.
  - `screenshot.js` — static capture via Puppeteer `page.screenshot` at `deviceScaleFactor: 2`.
  - `evaluate.js` — Phase 6 code-evaluated scorecard rows (format-aware).
- [`repo-visuals-retro/`](./repo-visuals-retro/) — retrospective meta-skill that reads evaluation logs and proposes edits to `SKILL.md`. Invoked on-demand, not per run.
- `evaluations/runs/` — gitignored per-run raw scorecards.
- `evaluations/index.md` — curated aggregate across runs.

## Dependencies

- Node.js
- `puppeteer` — auto-installed via `npm install` (~170 MB Chromium on first run).
- `ffmpeg` + `ffprobe` — system install (`brew install ffmpeg`, `apt install ffmpeg`, `choco install ffmpeg`) or portable binaries in `./bin/`. See `SKILL.md` §4.1.
- `gifsicle` — optional; enables GIF palette-size checks in `scripts/evaluate.js`.
