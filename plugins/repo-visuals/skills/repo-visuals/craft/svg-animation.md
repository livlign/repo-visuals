# Animated SVG — generator path (for typewriter / terminal demos)

`SKILL.md` §4.2 covers the **default** SVG path: hand-author `index.svg` in Phase 2, validate (strip scripts, embed fonts, no external images), ship. Use that path for logos, brand flourishes, simple loops — anything where the SVG is short enough to author directly.

This file covers a **second path**: when the SVG mirrors a complex HTML/CSS/JS hero (terminal demo, multi-command typewriter, sliding caret, per-line fade-in cascade), hand-authoring becomes unmaintainable. Use a small generator script (`build-svg.js`) that emits the SVG from the same content data the HTML uses. Read this only when both apply:

- The chosen format set includes animated SVG.
- The hero is a terminal/UI demo with **per-character typing** or other timing-driven motion that mirrors an HTML version.

For static or simple animated SVG (no typewriter), use §4.2's default path — none of the rules below apply.

## Why a generator instead of hand-authoring

A typewriter scene with N characters per command and M commands needs ~(N×M) keyTimes/values across SMIL `<animate>` elements. Hand-authoring those ~150+ keyTimes is tedious and error-prone, and any change to a command string requires recomputing them. A 200-line generator that takes a `COMMANDS` array and emits the SVG keeps the content data flat, mirrors the HTML's source structure, and re-runs in milliseconds.

The generator output is still a self-contained SVG file — same hand-off as §4.2's path. The script just sits next to it (`build-svg.js`) so future tweaks are one re-run away.

## Architectural rules — battle-tested

These came from the cmder hero rebuild. The first attempt rendered as a static end-state with no typing visible at all; the second produced a smooth left-to-right wipe; only the third — built to these rules — produced credible per-character typing.

1. **CSS keyframes only for opacity fades on `<text>` / `<g>` elements.** Per-line fade-in via CSS `@keyframes` is fine. Anything else, use SMIL.

2. **NEVER animate SVG `<rect>` `width`, `x`, or `y` via CSS** (e.g. `width: Npx;` keyframes). It looks correct on paper but does not animate in headless Chromium, in Puppeteer screenshots, or reliably on GitHub's image proxy. Use SMIL `<animate attributeName="width" ...>` inside the `<rect>` instead. **Single biggest mistake** on the first attempt — the SVG rendered as a static end-state.

3. **Use `calcMode="discrete"` for typewriter typing.** Linear interpolation produces a smooth wipe that does not read as typing. Discrete + per-character keyTimes (one entry per char with width = `i * CHAR_W`) produces the snap-snap-snap rhythm of a real terminal. For an N-char command typed between t_from and t_to, emit N+1 entries `{ keyTime: t_from + i*(t_to-t_from)/N, value: i*CHAR_W }`.

4. **Caret = nested `<g opacity="1"><rect opacity="0">...</rect></g>`.** Outer `<g>` does the 0.9 s blink via SMIL `calcMode="discrete"` on opacity. Inner `<rect>` does the show-window opacity AND the `x`-position, both via SMIL discrete. Effective visibility multiplies (parent.opacity × child.opacity). The `opacity="1"` on the `<g>` is **required** — without an initial value SMIL has no baseline and the caret never appears.

5. **One caret per command** (anchored at that command's line y), not one global caret animated across lines. A global caret would require animating the SVG `transform` (fragile) or `y` attribute (works but adds a third SMIL animation per caret with harder keyTimes).

6. **CHAR_W = font-size × 0.6** for monospace approximation (10.8 px at 18 px). Good enough for visible caret position with any monospace fallback. If the user demands pixel-perfect alignment with the rendered text, embed JetBrains Mono via base64 `@font-face` — but that adds ~50 KB and pushes past the §4.2 size cap.

7. **Each SMIL animation must snap back to its initial value at keyTime 1.0.** Otherwise the loop's second iteration starts mid-state and looks broken. Pattern: `values="0; 0; W; W; 0"` `keyTimes="0; tFrom; tTo; 0.995; 1"` — the final `0` at keyTime 1.0 is the loop-seam reset.

8. **Wrap dynamic content in a `<g clip-path="url(#termClip)">`** where `termClip` is a proper `<clipPath>` element with a `<rect>` defining the terminal body region. CSS `clip-path: inset(...)` on SVG groups is unreliable across renderers; SVG `<clipPath>` always works.

## Generator script outline (`build-svg.js`)

```js
// 1. Define palette + cmd sequence — mirror the HTML's COMMANDS array exactly.
//    Same paths, same outputs, same timing constants (CYCLE, TYPE_START, TYPE_END).
// 2. Compute layout constants: TERM_X, TERM_Y, BODY_X, LINE_HEIGHT, CHAR_W.
// 3. Build a `lines` list with per-entry {tokens, at, kind: 'plain'|'cmd', ...}.
// 4. For each line:
//    - Emit per-line CSS @keyframes for opacity fade-in (rule 1).
//    - If kind == 'cmd': emit a <clipPath> with a <rect> whose `width`
//      is animated by SMIL discrete with one keyTime per character (rules 2 + 3).
//    - If kind == 'cmd': emit a caret <g><rect> with SMIL `x` (discrete)
//      + SMIL show-window opacity (discrete) on the rect, plus SMIL blink
//      opacity on the wrapping <g> (rules 4 + 5).
// 5. Emit chrome (background gradients, terminal frame, titlebar, controls) as static SVG.
// 6. Wrap dynamic content in <g clip-path="url(#termClip)"> (rule 8).
// 7. Write to <scratch-dir>/hero.svg.
```

The generator and the HTML must use the same `COMMANDS` data structure. The expected workflow: edit a command in the HTML, mirror the change in `build-svg.js`, re-run `node build-svg.js`. (Future improvement: have the HTML and generator share a JSON file. Not done yet because the data is small and editing two places is faster than wiring up an import step in a single-file scratch HTML.)

## Verification protocol

Render multiple Puppeteer snapshots at varying wait times. Account for the ~500 ms page-load lag — `wanted_animation_time = wait_ms + load_lag`.

```bash
for ms in 700 1000 4500 6500 6800; do
  node scripts/late-shot.js path/to/hero.svg out-${ms}.png 1200 675 ${ms}
done
```

Look for:

- **Characters snapping in discretely**, not wiping smoothly. Compare two adjacent times — if the difference is "more text revealed by exactly N character-widths," it's correct discrete typing. If it's "a smooth horizontal extension," the SMIL is using linear interpolation (rule 3 not applied).
- **A visible white caret rect at the right edge of typed text on at least some captures.** Not all — the caret blinks at 0.9 s, so half of any random sample will land in the off-phase. Across 5+ captures, at least 2 should show the caret.
- **Correct loop seam.** Capture at t = LOOP_MS - 200 ms (just before reset) and t = LOOP_MS + 200 ms (just after). The transition should be clean — the second frame should look like the first frame of the loop, not a partial mid-state.

If every frame across multiple captures shows a complete end-state with no partial typing, **the SMIL didn't run** — most likely cause is using CSS `width: Npx;` instead of SMIL `<animate attributeName="width">` (rule 2).

## Size & validation

`SKILL.md` §4.2 sets the target at ≤ 200 KB and hard cap 500 KB. Generator-driven typewriter SVGs typically come in well under that:

- **Reference:** the cmder hero (5 commands, per-char typing, 6 carets) ships at ~18 KB without embedded fonts.
- **With embedded JetBrains Mono `@font-face` (base64):** add ~50 KB. Only do this if pixel-perfect alignment with the HTML rendering is required.
- If size grows past 25 KB without embedded fonts, the SVG markup is being too verbose — consolidate carets, drop a command, or shorten command strings.

Run §4.2's validation checklist on the generated file: strip any `<script>` tags (your generator shouldn't emit them anyway), confirm no external `<image href>`, confirm no `https://fonts.googleapis.com/...` imports, render via `<img src="index.svg">` in a throwaway `preview.html` and visually confirm.

## Output

- Ship file: `repo-visuals-work/<repo-name>/hero.svg`
- Generator: `repo-visuals-work/<repo-name>/build-svg.js` (kept next to the SVG so future tweaks can regenerate)

The SVG and any GIF/HTML siblings should be content-identical (same commands, same paths, same timing). Different formats serve different surfaces; they should never tell different stories.
