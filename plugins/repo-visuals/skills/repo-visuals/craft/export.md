# Export pipeline — full recipes

`SKILL.md` Phase 4 picks the format and gates ship-readiness; this file holds the actual capture and encode recipes. Read this when you're about to run a capture, not before.

The capture script is at `scripts/capture.js`; the static screenshot wrapper is at `scripts/screenshot.js`. Both already accept `--width` / `--height` flags and default to 2× retina. Pass whatever the brief locked in; don't hardcode 1200×675 in export commands.

---

## Capture script CLI

```
node scripts/capture.js --html <path-to-index.html> --out <frames-dir> --duration 20700 --width 1200 --height 675
```

`--scale 2` is the default and produces 2× frames.

---

## GIF pipeline (the proven recipe)

Use as-is unless there's a specific reason to deviate.

### Capture (2× retina — mandatory for anything with text)

**Preferred path (current Chromium honors `deviceScaleFactor` during screencast):**

- Launch Puppeteer (`headless: true` — `'new'` is deprecated).
- `setViewport({ width: W, height: H, deviceScaleFactor: 2 })`. Screencast frames emit at `2W × 2H`.
- `page.goto(file://...)` with `waitUntil: 'networkidle0'`.
- `await page.evaluateHandle('document.fonts.ready')`.
- Small real-time settle (300–400 ms).
- `await page.evaluate(() => window.runLoop())` to reset animation to t=0.
- Subscribe to `Page.screencastFrame`, save each frame as PNG. Verify the first saved frame is `2W × 2H` — if it's `W × H`, `deviceScaleFactor` is being ignored and you need the fallback below.
- `client.send('Page.startScreencast', { format: 'png', everyNthFrame: 1 })`.

This is what `scripts/capture.js --scale 2` (default) now does.

**Fallback (if a future Chromium/Puppeteer silently ignores `deviceScaleFactor` in screencast):** enlarge the viewport to 2× the target dimensions and apply `zoom: 2` to the body so the stage renders at 2× density natively. Use this only if the preferred path produces 1× frames:

- `setViewport({ width: W * 2, height: H * 2, deviceScaleFactor: 1 })`.
- Override the preview-only media query so the stage stays flush at `(0, 0)`:
  ```js
  await page.addStyleTag({ content: `
    html, body { margin: 0 !important; padding: 0 !important; }
    body { display: block !important; align-items: flex-start !important; justify-content: flex-start !important; }
  `});
  ```
- `await page.evaluate(() => { document.body.style.zoom = '2'; })`.
- `await page.evaluateHandle('document.fonts.ready')`.
- Small real-time settle (300–400 ms, slightly longer than 1× captures because zoom + font reflow take a moment).
- `await page.evaluate(() => window.runLoop())` to reset the animation to t=0.
- Create a CDP session: `page.target().createCDPSession()`.
- Subscribe to `Page.screencastFrame` — save each frame as PNG, record `metadata.timestamp` (seconds). Verify the first saved frame is `2W × 2H` pixels; if it's still `W × H` the zoom/viewport step didn't apply and you're about to ship a 1× render.
- `client.send('Page.startScreencast', { format: 'png', everyNthFrame: 1 })`.
- Capture until `elapsedMs > DURATION_MS` (no extra padding — anything past `TIMELINE.loopEnd` is the start of the next cycle and will break the loop seam). `DURATION_MS` must equal `TIMELINE.loopEnd` exactly.
- Close browser. Write an ffmpeg **concat manifest** with per-frame durations from timestamp deltas — preserves the real paint cadence.

**Why screencast, not screenshot-loop or virtual-time:** real screencast records exactly what the compositor paints, including CSS transitions. Screenshot loops drift under load; virtual-time (`Emulation.setVirtualTimePolicy`) freezes the compositor and captures stale frames.

### Encode (two-pass palette)

```bash
# Pass 1: palette built from 2× frames, lanczos-downscaled to target size
ffmpeg -y -f concat -safe 0 -i frames.txt \
  -vf "fps=24,scale=<W>:<H>:flags=lanczos+accurate_rnd+full_chroma_int,palettegen=stats_mode=full:max_colors=256" \
  palette.png

# Pass 2: apply palette with sharp-text dither, same lanczos downscale
ffmpeg -y -f concat -safe 0 -i frames.txt -i palette.png \
  -lavfi "fps=24,scale=<W>:<H>:flags=lanczos+accurate_rnd+full_chroma_int [x]; [x][1:v] paletteuse=dither=bayer:bayer_scale=3:diff_mode=rectangle" \
  -loop 0 hero.gif
```

- **`scale=<W>:<H>:flags=lanczos+accurate_rnd+full_chroma_int` in *both* passes.** Frames from `capture.js --scale 2` are `2W × 2H`; this downscales them to the target dimensions cleanly. Omitting the downscale ships a final GIF at doubled dimensions (acceptable on retina but bloats file size ~4×).
- **`stats_mode=full`** — analyzes every pixel, giving chrome text equal weight to motion regions. Use `stats_mode=diff` only if the hero is mostly-static with a small moving region and text sharpness isn't a concern.
- **`bayer:bayer_scale=3`** — sharper than `bayer_scale=5`; try `dither=none` if text is still blurry and gradients are minimal (~30% file-size cost).
- **`fps=24` is a good default** (20s × 24 = 480 frames). `fps=30` looks buttery but bloats file size; drop to `fps=20` or `fps=18` if over the 10 MB target.

### GIF size budget

- **Target: ≤ 10 MB** — default cap. Renders cleanly on GitHub, mobile / slow connections don't suffer.
- **Hard max: 15 MB** — only if the content genuinely requires it *and* the user has confirmed they accept the bigger file. Never exceed silently.
- Reduction ladder when over 10 MB: drop to `fps=20` → drop to `fps=15` → `max_colors=192` → `max_colors=128` → shorten loop. Apply in order; stop as soon as under budget.

---

## Static PNG pipeline

For the static format, skip ffmpeg entirely — a single crisp screenshot is enough.

### Capture via Puppeteer

- Launch Puppeteer (`headless: 'new'`).
- `setViewport({ width, height, deviceScaleFactor: 2 })` — retina-crisp for README rendering at native size.
- `page.goto(file://...)` with `waitUntil: 'networkidle0'`.
- `await page.evaluateHandle('document.fonts.ready')`.
- Settle 500ms so any entrance transition finishes.
- `await page.screenshot({ path: 'hero.png', type: 'png', omitBackground: false, clip: { x: 0, y: 0, width: W, height: H } })`.

If the HTML has an animation that evolves over time, either:
- Design the static HTML to render its final/decisive frame at t=0 (preferred), or
- Before the screenshot: `await page.evaluate(() => window.seekTo(<seconds>))` if the source HTML exposes such a hook.

**Optional compression:** run `pngquant --quality=80-95 hero.png --output hero.png --force` if the PNG is over ~500 KB. Static heroes rarely need it.

### PNG size budget

Target ≤ 500 KB for a 1200-wide retina PNG. If over 1 MB, compress.

**Ship script:** `node scripts/screenshot.js --html <path> --out hero.png --width 1200 --height 675` — thin wrapper around the steps above.

---

## Output file name

- Animated → `repo-visuals-work/<repo-name>/hero.gif`
- Static   → `repo-visuals-work/<repo-name>/hero.png`

Keep in the scratch dir until Phase 5 (Output) moves it into the target repo.

---

## Future formats (not yet wired)

- **MP4 / WebM loop** — `libx264 -crf 18` or `libvpx-vp9 -crf 32`. Higher fidelity, smaller files. Note: GitHub renders `.mp4` uploaded via issue/PR drag-and-drop but not `.mp4` checked into the repo and linked in markdown.
- **Square social card (800×800 or 1200×1200)** — variant of static PNG, often reframed rather than cropped, with headline overlay tuned for LinkedIn / Twitter preview rendering.
- **9-frame contact sheet** — helper for picking the decisive moment of a long animation before static export.
