# Craft rules — full versions

The headlines of these rules live in `SKILL.md` §2.4 as one-liners. This file holds the full incident-grounded rationale. Read the matching section here when you're actually doing the work the rule applies to (writing layout, embedding an asset, exporting, etc.) — not eagerly.

---

## No drift-prone claims in on-screen copy

Never hardcode anything that becomes stale the moment upstream ships or the clock ticks forward. The hero outlives any single release — a visible `v1.9.2` turns obsolete the next Tuesday, a visible `6.4k stars` turns obsolete the next weekend.

**Banned as exact values:** version numbers (`v1.9.2`, `2.0.0-rc3`), language/runtime floors (`Node 20+`, `Python 3.10+` — these bump every ~2yrs), star counts (`6.4k stars`), download counts (`12M weekly`), contributor counts (`52 contributors`), "last updated" / release dates, benchmark figures pulled from a specific build.

**Allowed timeless claims:** package name, license, architecture primitives (e.g. "MCP", "CLI + API"), domain nouns ("semantic search", "static analysis"), scope words ("multi-platform", "zero-config").

**Soft-cap rule for drift-prone stats:** if a number genuinely helps the pitch, phrase it as **`<next lower round number>+`** so upstream movement doesn't falsify it. Examples: `44 rules → "40+ rules"`, `6,421 stars → "6k+ stars"`, `127 contributors → "100+ contributors"`, `12.3M weekly downloads → "10M+ weekly downloads"`. Round **down** to the nearest meaningful milestone — never round up, and never pick a cap the number is about to cross (a `9k+ stars` claim on a 9.1k repo is a landmine; use `9k+` only when the repo is comfortably past that for months, otherwise drop to `5k+`).

**Precise counts acceptable only when frozen:** finished tutorial series, explicitly versioned snapshots, historical data — contexts where the count is *designed* not to move.

---

## Left-column layout — don't pin content to edges

In split-stage heroes with a brand/tagline column on the left, avoid `grid-template-rows: auto 1fr auto` or any pattern that parks the bottom row flush against the stage edge — it reads as cramped / "glued-in" at export. Either (a) center the column content vertically (`align-content: center` on the grid) with explicit gaps, or (b) use `grid-template-rows: auto auto 1fr` with meaningful breathing room (≥32px equivalent) below the last content row. Default to option (a) unless the column genuinely has more content than vertical space.

---

## Match density across columns

Don't let the left go empty while the right fills the stage. Split-stage composition failure: the title block on the left ends by the time the right-side product mock is only ~40% of the way down. The user reads it as a lopsided stage with a dead zone in the lower-left. Fix by placing secondary content (stats strip, language chips, quality gate, CTA pill, *anything* supporting the pitch) right under the title block in the left column, sitting on its own rule or divider — instead of bottom-pinning it with `bottom: Xpx`, which creates a large vertical gap *and* jams the footer against the edge.

---

## Bottom-anchored elements need real clearance, not 16–24 px

A stats strip, repo slug, or CTA anchored with `bottom: 18px` / `bottom: 24px` reads as "jammed against the edge" at both HTML preview and export. Bottom-pinned footer-style elements want **≥40 px** from the stage bottom edge, and the content immediately above them wants **≥32 px** vertical gap so the footer doesn't kiss the preceding row. If the math says the bottom anchor has less than that, reduce the stage height (trim dead space) before nudging the footer down — clamped bottom clearance is a stage-sizing problem disguised as a positioning problem.

Real incident: a Terminal.Gui hero shipped with `bottom: 22px` and the bottom line visibly hugged the edge; fix was `stage height 675 → 580` plus `bottom: 28px`, not just "move it".

---

## Scope rules to their region

If a rule applies only in one region, scope the selector to that region. Classes like `.row`, `.item`, `.card`, `.stat` are ubiquitous shortcuts — don't style them as plain `.row { display: grid; grid-template-columns: 80px 1fr }` at the top level, because every unrelated `.row` elsewhere in the stage (tree row, menu row, metric row) will silently inherit that grid and render broken. Scope the selector: `.form .row`, `.sidebar .item`, `.metrics .card`.

Real incident: a Terminal.Gui mock had the tree pane's rows render as stacked two-cell grids (arrow on one line, label on another) because the form's `.row` grid leaked globally.

---

## Preview-vs-export parity

The HTML the user sees is the PNG/GIF they get. A recurring failure mode: `index.html` looks perfect in the browser, but the exported PNG/GIF is cropped or shifted because the browser's natural centering (`body { display: grid; place-items: center }`) + the stage's `box-shadow` + any body padding collectively move the stage away from the `(0, 0)` origin the capture pipeline samples from. Rules to prevent this:

- **Body must pin the stage at (0, 0) for export.** Either drop the centering entirely (`body { margin: 0; padding: 0 }` with the stage at flow position) or wrap centering in `@media (min-height: 800px)` so headless Chrome at the capture viewport lands at origin. Never rely on `place-items: center` alone if the capture viewport equals the stage size exactly.
- **`box-shadow` on the stage is for the HTML preview only.** Shadows extend outside the 1200×675 box and get guillotined by the capture crop — fine, expected. But don't rely on the shadow as a visual boundary; the content inside the stage must read as complete without the shadow.
- **Never let content exceed the stage.** Any absolute-positioned element with `top: -N` or `right: -N` to create an "overlap" effect will be clipped in export. If you want a visual that bleeds past the stage edge, reposition it inside or fake the bleed with a gradient — do not let real content live outside.
- **Puppeteer screenshot must use explicit `clip`.** Always pass `clip: { x: 0, y: 0, width: 1200, height: 675 }` to `page.screenshot()` rather than relying on viewport-equals-capture. This guarantees the crop, regardless of scroll offset or body margin that slipped in.
- **Validate parity before declaring done.** After export, open the exported PNG/GIF side-by-side with `index.html` in the browser. If they don't match visually, the HTML is wrong (not the pipeline) — fix the layout, don't fix the pipeline.

---

## Retina quality is mandatory

Never ship a 1× render for anything with text. The final artifact must look like a native MacBook retina screenshot, not a downscaled print. Heroes are viewed almost entirely on high-DPI laptops and phones, and GitHub adds a ~0.83× column downscale on top — a 1× capture that looks fine locally will render visibly *fuzzy* in production. Symptoms: text edges blur, rule-chips / small badges turn mushy, thin rules and borders get eaten.

- **Capture at 2× always.** PNG: `setViewport({ ..., deviceScaleFactor: 2 })` — Puppeteer's `page.screenshot()` respects it. GIF: `deviceScaleFactor` is silently ignored by Chromium's screencast API — see `craft/export.md` for the mandatory workaround (viewport at 2× dimensions + `document.body.style.zoom = 2` + preview-media-query override).
- **Source font-size floor on a 1200-wide canvas: ~15.5 px for body text, ~12.5 px for small metadata (rule-chips, locations).** After GitHub's column downscale, those land at ~13 CSS px and ~10 CSS px — the lower bound where text stays readable. Drop below ~13 px source and rendered legibility craters. Headlines / wordmarks scale proportionally up from there (the HTMLHint v3c refresh used 42 px wordmark / 15.5 px body and that is the lower bound, not the target).
- **Own benchmark: "like a MacBook retina screenshot."** Put the exported hero next to any other UI chrome on the README (badges, code blocks, surrounding text). If the hero looks softer or stretchier, it's wrong — do not ship. Re-capture at 2× and/or raise source font sizes until the hero is visibly parity with the rest of the page.

Real incident: `htmlhint/HTMLHint#1861` shipped a 1× 1200×200 marquee at 11.5 px body text; visual "looks small / fuzzy" feedback required a full retina re-render.

---

## Symmetry is load-bearing

Check paddings, margins, and anchors deliberately. Before ship, verify top-padding = bottom-padding and left-margin = right-margin. When shrinking a right-anchored element (e.g., `shot-wrap` with `right: 24px`), preserve its **left-edge position**, not its right margin — otherwise the left whitespace gap grows as the element narrows, creating an orphan composition.

Real incident (Terminal.Gui hero, April 2026): initial layout had `top: 56px` on the title block and `bottom: 26px` on the footer — user called it out; later the demo shrank from 788×528 to 686×460 while `right: 24` stayed fixed, so the left gap grew from 44 → 146 px. Fix combined (a) equalize top/bottom padding, (b) equalize left/right margins by trimming the stage width, (c) preserve the demo's left-edge position when resizing.

---

## Embedding a user- or reviewer-provided asset

When a demo GIF, MOV, screenshot, or logo must appear *inside* the outer hero frame (not as chrome around it):

- **Loop alignment.** The inner asset's loop length must equal the outer loop, or be a clean divisor of it. If the source is mismatched (e.g. a 37.7 s demo inside a 20 s outer loop), re-encode with `ffmpeg -filter_complex "setpts=<ratio>*PTS"` to speed-trim *before* embedding. A seamless inner = seamless outer.
- **Native-source resolution.** Keep the inner at its native source dimensions — don't pre-downscale. The browser needs the most pixels it can get to sample from during the outer retina capture; pre-downscaling compounds with the outer's own downscale and visibly blurs the final.
- **Palette budget.** A colorful inner + colorful outer chrome will blow the 256-color GIF palette. Keep the outer chrome monochrome or low-chroma so the inner's colors pop; don't compete with the inner for palette slots.
- **Reviewer assets are sacred.** If a reviewer or upstream contributor supplied the asset, do not silently replace it with a self-generated recording — that re-opens their review. If a replacement genuinely is better, flag the scope change explicitly and ask before swapping.

---

## "Reduce" means reduce — directional verbs are load-bearing

When the user pairs a directional verb (*reduce*, *smaller*, *tighten*, *cut*, *shrink*, *trim*) with a relational clause (*"so that padding A equals padding B"*, *"to match X"*), treat the directional verb as the binding constraint. If the easy execution of the relational clause would violate the direction — e.g., equalizing padding by growing the demo — stop and restate rather than execute.

Real incident: user said *"reduce the height so the padding Cross-platform to top equal MIT to bottom"* — I executed as "equalize padding" and grew the demo from 749×502 to 788×528; user had to correct *"I meant reduce the size, not increase it to bigger."* The literal word was clear; the misread cost a capture-and-encode round trip.
