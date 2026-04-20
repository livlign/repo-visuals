# Resume prompt

Paste the block below into Claude Code on your local machine (opened in the cloned `repo-visuals` dir) to continue where the last session stopped.

---

I'm continuing work on the `repo-visuals` Claude Code skill. Context:

- **What this repo is:** a skill that produces animated hero GIFs for GitHub repos via a structured discovery dialog → bespoke HTML → Puppeteer + ffmpeg export. v1 ships GIF only; MP4/static/social-card are v2+. Read `SKILL.md` end-to-end before doing anything.
- **Companion skill:** `repo-visuals-retro/SKILL.md` — retrospective meta-skill that reads `evaluations/runs/` and proposes edits to `SKILL.md`. Not run per session — only invoked when ≥5 runs have accumulated.
- **Craft library:** `craft/headlines.md` (imperative+invariant and narrative-arc patterns) + `craft/templates/ast-graph-v1.html` and `ast-graph-v2.html` as structural references.
- **Proven export pipeline:** `scripts/capture.js` uses CDP `Page.startScreencast` for real-time capture; encodes with ffmpeg two-pass palette (`stats_mode=diff` + `bayer:bayer_scale=5`). Target GIF size ≤ 10 MB, hard cap 15 MB.

**Current state (mid-run, dogfood on `emtyty/devtool`):**
- Scratch dir: `work/emtyty-devtool/`
- `index.html` — 7-scene hero (SQL → JSON → Diagram → Cron → Color → JWT → tool-grid finale) with per-scene headline strip (imperative-plus-invariant voice). 1200×675, 20 s loop.
- `hero.gif` — 2.4 MB export. Stopped before user shipped; open for iteration or proceed to Phase 5 (placement + PR to `emtyty/devtool`).
- **Phase 6 (Evaluate)** — scorecard not yet filled in for this run.

**Dependencies not in repo (gitignored):**
- `node_modules/` → `npm install puppeteer` (or `cd work/emtyty-devtool && npm i puppeteer`).
- `ffmpeg` → system install or portable `ffmpeg.exe` in a `bin/` dir.

**Pending decisions / open threads:**
- Whether the current devtool hero copy/pacing ships as-is or needs more iteration.
- Phase 5: which placement in `emtyty/devtool` repo (`assets/` preferred per skill), README embed location, PR via `gh` as account `livlign`.
- Phase 6: fill scorecard and write the first `evaluations/runs/2026-xx-xx-emtyty-devtool.md`.

Start by re-opening `work/emtyty-devtool/index.html` in my browser so I can re-evaluate, then ask me what to iterate on — or if I say "ship it", proceed to Phase 5 (placement + PR to `emtyty/devtool` as `livlign`, no Claude co-author).

---
