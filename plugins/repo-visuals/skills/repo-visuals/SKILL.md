---
name: repo-visuals
description: Create hero visuals — animated GIF or static PNG — for GitHub repositories. Runs a structured discovery conversation (scan repo → recommend format → propose creative scenarios → agree on a brief), then designs bespoke HTML, previews it in the browser, and exports.
---

# repo-visuals

Turn a repo (GitHub URL, local folder, or free-text brief) into a hero visual that a viewer sees at the top of the README and instantly understands *what this project does and why it's interesting*. The hero may be an **animated GIF** or a **static PNG** — the skill recommends one based on the repo's identity, the user picks.

The skill's quality comes from the **discovery dialog**, not from templates. Every hero is bespoke.

## Phases

1. **Discovery** — pick operating mode (Auto / Semi-auto / Manual, §1.1a), scan the repo, summarize findings, recommend a format (animated vs static), ask about vibe/audience/hero moment, propose 2–3 scenarios, converge on a brief. The mode controls *how many* of these the user answers vs. Claude decides silently — it does not skip any of the craft checks.
2. **Build** — write HTML/CSS/JS for the chosen scenario. For static, design one decisive frame; for animated, a loop. No storyboard step.
3. **Preview & iterate** — open the HTML in the user's default browser, iterate in text until the user says ship.
4. **Export** — animated → Puppeteer screencast + ffmpeg palette pipeline to GIF. Static → Puppeteer `page.screenshot` to PNG (with `deviceScaleFactor: 2` for retina crispness).
5. **Output** — place the hero in the target repo; optionally open a PR that embeds it in the README.

**Dev mode (author-only):** Phase 6 — Evaluate exists but **only runs in dev mode**. Dev mode is for the skill's author iterating on the skill itself; it collects scorecard data and writes run logs under `./evaluations/` in the user's current working directory. Enable it only when the user explicitly says "dev mode" (or sets `REPO_VISUALS_DEV=1`). In every normal run — including Manual, Semi-auto, and Auto — the skill ends after Phase 5. Do not mention Phase 6 to end users.

---

## Phase 1 — Discovery

Discovery runs **before any HTML is written**. Its job: go from a vague ask ("make a hero GIF for my repo") to a specific, committed creative brief.

### 1.1 Activation

Enter discovery mode whenever the user invokes the skill, **unless** they explicitly say "skip discovery, I have the brief ready."

### 1.1a Operating mode (ask first, once per run)

Before anything else — before the scan summary, before any direction questions — ask the user which operating mode they want for this run. Use the `AskUserQuestion` tool so the choice is structured and visible, not buried in free text. Offer three modes; describe each in concrete terms (what gets asked, what gets decided silently) and include pros/cons so the user can pick with open eyes. Default recommendation: **Semi-auto**.

> Note: this "operating mode" is a skill-level concept about how many questions the skill asks. It is distinct from the Claude Code harness's own auto-mode; the two do not replace each other.

Question shape:

```
AskUserQuestion({
  questions: [{
    header: "Run mode",
    question: "How involved do you want to be in this run? This affects how many decisions I ask you to make before we ship an artifact.",
    multiSelect: false,
    options: [
      {
        label: "Semi-auto (Recommended)",
        description: "I decide vibe, audience, scenario, dimensions, copy. You decide: output format (GIF/PNG/HTML) and one preview-and-iterate review before export. ~2 decisions. Pros: fast, keeps production-grade gate, keeps your taste in the loop on the things that matter most. Cons: you miss input on smaller creative calls."
      },
      {
        label: "Manual",
        description: "I ask you at every decision point — scenario pick, vibe confirmation, brief approval, preview iteration rounds, export ship-intent. I still make suggestions and recommendations at each step. Pros: max control, highest ceiling on quality. Cons: slow — 8–12 back-and-forths before an artifact."
      },
      {
        label: "Auto",
        description: "I make every decision and go straight to the exported artifact (GIF or PNG, my call). Pros: hands-off, 0 decisions, fastest path to a shippable draft. Cons: lower quality ceiling, more risk of missing your taste or the repo's real scope; expect to redirect after seeing the result."
      }
    ]
  }]
})
```

Rules that apply to **every** mode regardless of choice (craft non-negotiables):

- **§1.3 inventory count is always collected.** Auto/semi don't skip it.
- **§4.0 scope-match rule is always enforced.** If the hero says "all" or shows a grid, on-screen reality must match the real inventory — regardless of mode.
- **Any mode can be upgraded mid-run.** If Auto drifts, user can say "stop, switch to semi" and we resume from the nearest decision point. Do not silently re-ask everything; pick up at the next unanswered question.

After the user picks a mode, commit it to memory for the run (e.g. "Mode: Semi-auto") and reference it when deciding whether to ask or decide silently at each subsequent step. In Auto and Semi-auto, make decisions with a brief one-line note ("going with the Product-UI marketing archetype per §1.4c — amplication-shape repo") so the user can redirect if they disagree.

### 1.2 Input triage

User may provide:

- **GitHub URL** → clone shallow, plus use `gh repo view` for stars/topics/languages/releases.
- **Local path** → scan the tree directly.
- **Free-text brief only** → proceed to direction questions immediately.
- **Nothing** → ask first for one of the above.

### 1.3 Scan checklist (collect a lot)

Gather as much as possible before asking the user anything. Goal: Claude should already have an opinion about what this repo is before the vibe conversation starts.

- README (full text) — extract tagline from first paragraph.
- Manifest(s): `package.json`, `Cargo.toml`, `pyproject.toml`, `go.mod`, `*.csproj`, etc. — description, keywords, dependencies, entry points.
- File tree (depth 3) + top 10 files by size.
- Language breakdown (LOC per language).
- Existing visuals in repo: `assets/`, `docs/`, `.github/`, any `*.png` / `*.gif` / `*.svg` near the root.
- `LICENSE`, `CHANGELOG.md` — maturity signal.
- Recent git log (last 10 commits) — what's active.
- If GitHub: stars, topics, releases, open issues count.
- If `ast-graph` is available locally and the repo has supported languages: run `scan` + `hotspots` + `symbol` on top-level exports for structural inventory.
- **Count the real inventory** — if the repo exposes a collection (tools, commands, pages, themes, plugins, routes), count them from source (route files, `src/tools/*`, CLI command registrations, etc.). Do not estimate from the README. This number is used later: any hero that says "all", "every", "the whole", or shows a grid meant to represent scope must match this count — or explicitly frame itself as a sample ("10 of 30 shown", scroll affordance, "30+"). An undercount reads as a broken promise.

Summarize findings back to the user in ~6–10 bullets so they can correct misreadings early. Include the inventory count when one exists. **If the scan finds an existing hero**, branch into §1.3a before continuing to direction questions.

### 1.3a Redesign vs greenfield (only if an existing hero is found)

If the scan surfaced an existing hero, branch into the redesign protocol *before* §1.4. Surface the existing file (path, format, age, README placement), then `AskUserQuestion` the entry mode — **Redesign (carry forward)** / **Fresh (ignore existing)** / **Replace in place** — and follow the matching steps. Full protocol (entry-mode question, critique structure, carry-forward rules, scenario constraints) in `craft/redesign.md`.

### 1.4 Direction questions (after the scan summary)

Ask these as a structured batch, with suggested defaults based on what the scan found so the user can just accept or redirect:

1. **Where will this hero live?** — README top / project website / social posts (Twitter, LinkedIn) / conference slide / internal demo / elsewhere. *This drives dimensions and loop length.*
2. **Output format?** — **animated GIF** / **static image (PNG)** / **HTML only (no export)**. Not every repo wants motion; some strong identities read better as one sharp frame. Scan the repo first and offer a recommendation (see §1.4c), then let the user override. Default: whichever §1.4c recommends. *(MP4/WebM and square social cards are listed in the future-formats appendix.)*
3. **Audience** — working devs / eng leaders / newcomers / prospective contributors / all
4. **Vibe** — minimal, playful, technical, cinematic, retro-terminal, brutalist, polished-SaaS
5. **Energy** — calm & meditative / steady / frenetic & showy
6. **Hero moment** — the single thing a viewer should take away
7. **Hard constraints** — brand colors, existing fonts, imagery to include/avoid, duration cap

### 1.4a Probing when the user doesn't know

Many users will answer "I don't know" to vibe / hero moment / audience. That's normal. Don't force them to guess — extract intent sideways:

- **"Show me a repo whose hero you liked"** — reverse-engineer what they respond to.
- **"What do you wish someone understood about this project in 10 seconds?"** — surfaces the hero moment without asking for it directly.
- **"Who's the person whose opinion about this repo matters most to you?"** — surfaces the audience.
- **"Is this repo a calm/serious tool or a fun/exciting one?"** — a binary that usually works for vibe.

If they still can't articulate, offer Claude's best guess based on the scan and ask them to confirm or push back. Be willing to **commit first, confirm second** — a concrete wrong guess triggers better reactions than another open-ended question.

### 1.4c Format recommendation (animated vs static)

Before asking Q2, form an opinion from the scan. **First consult `craft/reference-gallery.md`** — try to place the target repo in one of the catalogued archetypes (Terminal-demo GIF / Product-UI marketing / Brand-first logo / Banner/promo graphic / Diagram-as-hero) and cite the match by name when recommending. If no clean match, fall back to the heuristics below.

The repo's identity tells you which format fits:

**Lean static (PNG) when:**
- The repo's pitch is a **surface**, not a process — a landing page, a typography choice, an API shape, a single striking screenshot.
- Existing visuals in `assets/` / `docs/` are already static and strong.
- The vibe is brutalist, editorial, documentation-first, or "the screenshot is the product" (design systems, component libs shown in a grid, CLI themes whose appeal is one frame).
- Audience skews toward skim-then-click (social posts shared as preview cards, directory listings).
- README opens with one crisp hero sentence and a screenshot — mirror that, don't animate it.

**Lean animated (GIF) when:**
- There's a **before→after transformation**, a step sequence, or a multi-stage pipeline — the motion *is* the pitch.
- The repo is a tool with visible runtime behavior (CLI output, live reload, computation unfolding).
- The hero moment is "watch this happen," not "see this exists."
- The scan surfaces multiple distinct capabilities worth walking through (then: animated; or: static grid).

**Lean HTML-only when:**
- The user will embed on a site they control and wants the live animation.
- The user explicitly wants to iterate further with their own tools.

Present the recommendation forcefully: *"Based on the scan, I'd go **static** — this repo's identity is [X], and motion would add noise not signal. Want me to proceed with static, or would you rather animated?"* Then honor whatever they pick.

### 1.5 Scenario proposal

Offer **2–3 concrete scenarios** (two if the viable directions are closely related, three if they diverge meaningfully), each with:

- **Concept** — one line
- **Scene beats** — 3–5 bullets describing what happens second-by-second
- **Vibe anchor** — a visual reference ("think Linear changelog", "think `htop`")
- **Why it suits this repo** — one paragraph grounding it in the scan findings
- **Risk** — one line on what could make this one weaker

Then **Claude's recommendation**: pick one and argue for it forcefully in 1–2 sentences. Frame it as something the user can still redirect.

### 1.6 Convergence signal — "enough info"

Move to the build phase when all six are settled:

- [ ] Chosen scenario (one of the 2–3 proposed, possibly with edits)
- [ ] Vibe locked
- [ ] Hero moment named in one sentence
- [ ] Audience named
- [ ] Hard constraints captured (or confirmed "none")
- [ ] Output format chosen (animated GIF / static PNG / HTML only)
- [ ] Placement agreed (README / website / social / slide / other)
- [ ] Duration chosen — animated only (typical: 15–25s loop; default 20s); skip for static
- [ ] Dimensions confirmed (see §1.7 — default 1200×675; tailor when the repo spirit calls for it)
- [ ] Real inventory count captured from §1.3 scan (if the repo has a countable collection)
- [ ] Operating mode recorded from §1.1a (Auto / Semi-auto / Manual)
- [ ] Paddings are symmetric — top = bottom, left = right — unless a deliberate asymmetry is named in the brief (§2.4 symmetry rule)

Write the brief back to the user in a compact block. Wait for **"go"** before writing any HTML — *unless* the mode is **Auto**, in which case proceed directly to Phase 2 with a brief summary line and no wait. In **Semi-auto**, show the brief but proceed after a brief pause unless the user interjects. In **Manual**, wait explicitly for "go".

### 1.7 Dimensions — default with room to tailor

**Default: 1200×675 (16:9).** It's the safe pick when the repo gives no strong signal — GitHub's ~1000px README column renders it at ~560h (comfortable but not dominant), and the skill's capture/eval scripts are calibrated around that width. Start here unless the repo *wants* something else.

**Tailor when the repo's spirit points elsewhere.** Don't force a 16:9 crop onto a project whose identity is clearly another shape. Signals to deviate:
- **Slim banner (e.g. 1200×300 / 1600×400):** one-line wordmark-led brands, docs sites with an existing slim header, terminal-themed repos where the pitch is a single ribbon of text. Banners read as "logo-bar," not "hero scene" — use shorter loops (10–15s) and fewer scenes.
- **Square / portrait (1:1, 4:5):** social-first repos, mobile SDKs, anything whose primary surface is a phone. Don't build these for a README header — they're a separate deliverable.
- **Wider than 16:9 (2:1, 21:9):** data-viz / dashboard / timeline repos whose pitch is lateral scan; spatial tools (maps, CAD, audio waveforms). Only go this wide if the content genuinely fills it — empty wide heroes look lonelier than cramped 16:9 ones.
- **Taller than 16:9 (4:3, 3:2):** repos with dense stacked panels (notebook screenshots, multi-pane IDEs, documentation browsers) where 675h would truncate the story.

**How to decide:** during the scan (§1.3), note the existing hero image's dimensions and the repo's format archetype (from `craft/reference-gallery.md`). If they point toward a non-16:9 shape *and* the brief's hero moment fits that shape, propose the tailored size in the direction-questions batch (§1.4) — show the user the default and the tailored option, cite the reason, let them pick. If nothing argues against 1200×675, don't introduce the choice; just use it.

**Craft rules scale with whatever dimensions are chosen** — the font-size floors, bottom-clearance minimums, and column-density rules later in this document are written around 1200×675 but are expressed as ratios (% of stage height, px relative to canvas width) so they port. When you deviate, recompute: e.g. at 1200×400 the body-text floor is still ~2.5% of stage height × retina headroom, not a fixed 17px.

**The capture pipeline already supports arbitrary dimensions** — `scripts/screenshot.js` and `capture.js` both take `--width` and `--height` flags. Pass whatever the brief locked in; don't hardcode 1200×675 in export commands.

---

## Phase 2 — Build

Once the brief is locked and the user says "go", write the HTML in **one file** (`index.html` in a working directory for this repo's hero).

### 2.1 Working directory

Default layout — always nest per-repo under a subdirectory named after the target repo (e.g. `last30days-skill`, `my-cli`):

```
<current-dir>/repo-visuals-work/<repo-name>/
  index.html        # the hero animation source
  assets/           # any images/SVGs the scenario needs
```

Use `repo-visuals-work/<repo-name>/` so the scratch files stay obviously separate from the target repo **and** runs for different repos don't clobber each other. Never write directly into `repo-visuals-work/` — always into the repo-named subfolder. If no obvious repo name, use a short slug derived from the brief.

### 2.2 Re-read the README before writing

Open the target repo's README once more right before writing HTML. Mirror its actual phrasing, headings, and technical terms in the animation's on-screen text. The hero should feel like it was written *by* the repo author, not *for* them.

### 2.2a Consult the craft library

Before writing any scene copy or layout, read:

- `craft/headlines.md` — headline patterns (imperative-plus-invariant, narrative arc), voice rules, anti-patterns.
- `craft/templates/*.html` — full working heroes from past runs, **all upstream-merged** (the maintainers of the target repos accepted them). Read end-to-end to see how a complete scene system is composed (stage + browser chrome + tool-body + rotating hero text + progress indicator + timeline scheduler). Don't copy verbatim — steal patterns. Match by archetype, not by preference ranking.

Every scene needs a headline. A demo without copy delivers no meaning.

### 2.3 Structure of `index.html`

Single self-contained file. No build step. Sections:

1. **Stage** — a fixed container sized to the chosen dimensions (§1.7). Everything absolutely positioned inside. Outside the stage: a subtle page background so the capture crop is unambiguous.
2. **Fonts & tokens** — import fonts at the top (Google Fonts `@import`), define CSS custom properties for the palette (`--bg`, `--accent`, `--text`, `--muted`, etc.) derived from the vibe/constraints.
3. **Scene DOM** — the HTML elements that each scene uses. Build all of them upfront; scenes show/hide via classes.
4. **Timeline object** — a JS `TIMELINE` with named keys (e.g. `scene2Start: 3500`) so pacing is tweakable in one spot. Include `loopPauseAt` so the animation has a clean loop boundary.
5. **Timer helpers** — `schedule(t, fn)` wrapping `setTimeout` into an `animTimers` array, plus a `runLoop()` that clears and restarts. This is what the export pipeline calls to reset to t=0 deterministically.
6. **Scenes** — each scene is a plain function that flips classes / starts tweens. Keep CSS transitions for simple motion; use small rAF-driven tweens for counters or typing effects.

### 2.4 Rules of thumb

- **Durations**: match the chosen loop length (default 20s). Each scene ~3–6s, never so fast the viewer can't read it. Shorter dimensions (banner / slim) naturally want shorter loops — 10–15s.
- **Motion**: use `transform` and `opacity` only — never `top/left/width/height` (janky, expensive).
- **Type**: scale the minimum body size to stage height — ~2.5% of stage height, bumped up for GIF legibility. At 675h that's ~17px; at 300h that's ~14px minimum. Never below 13px.
- **Palette**: pick 4–6 colors. More kills GIF compression *and* looks busy.
- **Loop seam**: the last frame should visually match the first (or transition back smoothly). `runLoop()` is the hard reset if needed.
- **Determinism**: no `Math.random()` without a seed, no `Date.now()`-based logic. Everything must replay identically each capture.
- **Avoid the NY Times / editorial-newspaper theme**: no serif-headline + rule-lines + dateline + column-grid pastiche. It's become a default AI-design shortcut and reads as generic. Pick a visual language that actually suits the repo's domain (terminal, data-viz, spatial, playful, brutalist, neon, etc.) — not a broadsheet aesthetic unless the repo is explicitly about journalism or publishing.
- **No drift-prone claims in on-screen copy.** No version numbers, star counts, download counts, contributor counts, language-floor versions, or release dates as exact values — they go stale fast. Use `<next lower round number>+` ("40+ rules", "6k+ stars") when a number helps the pitch. Full banned/allowed list and soft-cap reasoning in `craft/rules.md`.
- **Layout discipline (split-stage, anchors, scoping).** The bullets that bite during build:
  - Don't pin a left-column's bottom row flush to the edge — center the column or leave ≥32 px breathing room.
  - Don't let one column run empty while the other fills — match density with secondary content right under the title block, not bottom-pinned 200 px below.
  - Bottom-anchored footer elements need ≥40 px clearance and ≥32 px gap above. If the math doesn't work, shrink stage height — don't just nudge.
  - Scope generic class selectors to a region (`.form .row`, not bare `.row`) so they don't leak into the rest of the stage.
  - On-screen labels must read cold — no cryptic abbreviations a first-time viewer can't decode in two seconds.
  - Symmetry is load-bearing — top-pad = bottom-pad, left-margin = right-margin. When shrinking a right-anchored element, preserve its left-edge position, not its right margin.
  - Full incident write-ups for each in `craft/rules.md`.
- **Preview-vs-export parity.** The HTML the user previews must equal the exported pixel grid: pin the stage at `(0, 0)`, treat `box-shadow` as preview-only, never let content exceed the stage, always pass an explicit `clip` to `page.screenshot()`, and visually diff the export against the HTML before declaring done. Detail in `craft/rules.md`.
- **Retina quality is mandatory.** Capture at 2× always (PNG via `deviceScaleFactor: 2`; GIF via the zoom-fallback in `craft/export.md`). Source font floor on a 1200-wide canvas is ~15.5 px body / ~12.5 px metadata after GitHub's ~0.83× downscale. The own benchmark is "looks like a native MacBook retina screenshot" sitting next to other chrome on the README.
- **Embedding a user- or reviewer-provided asset has its own rulebook.** Loop must equal or cleanly divide the outer loop; keep inner at native resolution; budget palette by keeping outer chrome low-chroma; never silently replace a reviewer's asset. Full version in `craft/rules.md`.

### 2.5 When to stop writing and preview

First preview as soon as:

- All scenes are implemented (even if polish is rough)
- Timing hits the full duration target
- No console errors

Don't polish before first preview. User's reaction on overall shape is more valuable than Claude's local polish loop.

---

## Phase 3 — Preview & iterate

Keep this phase conversational. The goal is to converge on a version the user loves before spending time on export.

### 3.1 Open in browser

After writing `index.html`, **give the user the command** to open it themselves (their browser, their timing):

- **Windows**: `start repo-visuals-work/<repo-name>/index.html`
- **macOS**: `open repo-visuals-work/<repo-name>/index.html`
- **Linux**: `xdg-open repo-visuals-work/<repo-name>/index.html`

Tell them to watch **one full loop** (the animation restarts automatically via `runLoop()`).

### 3.2 What to ask after first preview

Keep first-preview questions focused on **shape**, not polish:

- Does the *hero moment* land?
- Does the overall pace feel right (too fast? too slow?)
- Does any scene feel confusing or unnecessary?
- Does the loop seam look clean?

Don't ask about colors/fonts/spacing on the first round. Polish comes after shape is right.

### 3.3 Iteration rhythm

- Each round: Claude edits, user refreshes, one sentence of reaction.
- After shape converges, switch to polish rounds: type hierarchy, color calibration, micro-timing, loop seam.
- If user gives vague feedback ("feels off"), ask **one pointed question** to narrow it — don't guess.
- **Style stuck? Invoke `frontend-design`.** If the user has gone back-and-forth on *style* (palette, type, overall aesthetic, visual language) for **3+ rounds** without converging — or says something like "still not it" / "try a totally different direction" — stop tweaking in place. Invoke the `frontend-design` skill via the `Skill` tool to get a fresh, high-quality design pass. Pass it the brief, the repo scan summary, the current `index.html`, and a plain-language description of what's not working. Use its output as the new starting point, then return to this iteration loop. Don't invoke it for pacing, timing, or animation-logic feedback — only when the blocker is *visual design quality*.

### 3.4 Mid-build GIF sanity check *(only if GIF is a target output)*

GIF quantization causes specific failures that HTML preview hides: small text blurs, near-identical hues posterize, fine gradients band. If GIF was selected in §1.4, run the export pipeline **once** at ~70% polish so these surface before the final polish rounds. Check:

- Is small text still legible?
- Do any colors posterize badly?
- Does the animation feel the same speed as the HTML?

If problems appear, tune the HTML (bigger type, fewer palette neighbors, simpler gradients) and keep iterating on the preview.

**Skip this section entirely if the output is not a GIF** (MP4/WebM preserve HTML fidelity; static outputs are sampled directly).

### 3.5 Stop signal

Stop iterating when the user says "ship it" (or equivalent). Don't invite more rounds — excess polish is real cost. If Claude thinks there's still a clear improvement available, mention it *once* and let the user decide.

### 3.6 Deliver all the intent, even unstated

The user may not fully know what they want. Keep watching for mismatches between the scan/brief and the HTML behavior:

- Did the README say "fast"? The hero should feel quick, not luxurious.
- Is the audience newcomers? Don't assume they know jargon — the hero's text should echo README phrasing.
- Did the brief say "calm"? Check for accidental frenetic motion (fast cuts, snappy eases).

When you spot a mismatch, flag it proactively ("the README leans 'fast' but the current pacing is slower — intentional, or should we tighten?"). Deliver intent, not just instructions.

**"Reduce" means reduce — directional verbs are load-bearing.** When the user pairs a directional verb (*reduce*, *smaller*, *tighten*, *cut*, *shrink*, *trim*) with a relational clause (*"so A equals B"*, *"to match X"*), treat the directional verb as the binding constraint. If executing the relational clause would violate the direction — e.g., equalizing padding by growing the demo — stop and restate rather than execute. Incident detail in `craft/rules.md`.

---

## Phase 4 — Export

### 4.0 Entry gate — do not skip

Before you export anything, confirm — the exact gate depends on operating mode (§1.1a):

| Gate item | Manual | Semi-auto | Auto |
|---|---|---|---|
| User has **seen the artifact running** (open in browser or rendered screenshot) | required | required | skipped |
| At least **one iteration round** after first look | required | required | skipped |
| **Scope match** — hero claims (*"all"*, *"every"*, grid/list representing the repo) match real inventory from §1.3, or explicitly framed as sample ("10 of 30", "30+", scroll affordance) | **required** | **required** | **required** |
| User has explicitly said ship / go / export | required | required (after 1 preview) | not required |

The scope-match rule is **non-negotiable in every mode** — it's a craft check, not a human-involvement question. The human-in-the-loop items are what modes toggle.

Export is the **last** step, not a midpoint. A capture→encode→evaluate cycle feels productive but burns the user's patience on a draft that wasn't ready. When in doubt between "export this pass" and "one more iteration", prefer the iteration — except in Auto mode, where the point is to deliver a first draft fast and let the user redirect.

### 4.1 Prerequisites (ask before installing)

Check presence, list missing, ask the user before installing:

- **Node.js + `puppeteer`** — via `node --version` and looking for `node_modules/puppeteer`. If missing: `npm install puppeteer` (~170MB Chromium download).
- **`ffmpeg`** — via `ffmpeg -version`. If missing:
  1. Prefer system package manager (`choco install ffmpeg`, `brew install ffmpeg`, `apt install ffmpeg`) — needs admin.
  2. On Windows without admin, download **portable** ffmpeg from `https://github.com/GyanD/codexffmpeg/releases/latest` and extract `ffmpeg.exe` into the skill's `bin/` dir. Do NOT add to PATH; call by absolute path.

Never install silently. Show the plan, ask, then act.

### 4.2 Branch on chosen format

Based on the format decided in §1.4c:

- **Animated GIF** → run the GIF pipeline. Recipe in `craft/export.md`.
- **Static PNG** → run the static pipeline. Recipe in `craft/export.md`.
- **HTML only** → skip Phase 4 entirely; the hand-off is `repo-visuals-work/<repo-name>/index.html`.

`scripts/capture.js` (GIF) and `scripts/screenshot.js` (PNG) wrap the recipes; both default to 2× retina and accept `--width` / `--height`. Don't hardcode 1200×675 in invocations — pass what the brief locked in.

### 4.3 Output file name

- Animated → `repo-visuals-work/<repo-name>/hero.gif`
- Static   → `repo-visuals-work/<repo-name>/hero.png`

Keep in the scratch dir until Phase 5 (Output) moves it.

---

## Phase 5 — Output

Move `hero.gif` / `hero.png` from the scratch dir into the target repo, update the README, optionally open a PR. Full mechanics in `craft/ship.md`. Decision-tree headlines:

- **Placement.** Infer from convention (`assets/` → `docs/` → `.github/` → default `assets/hero.gif`); ask before creating dirs. Default to **image + README line only** — no `hero.html`, no maintenance docs, unless the reviewer asks, the repo has precedent, or the user opts in. (Premature source-bundling is a top reviewer complaint — see `craft/ship.md` for incident links.)
- **README embed.** Insert at the top by default; replace existing only with confirmation. **Alt text is informational** — convey the stats / positioning the image shows, not just `<repo-name> demo`. **Always use relative paths** — never `raw.githubusercontent.com/...`, even if the rest of the README does (legacy debt; doesn't render in forks/PR previews).
- **Commit & PR.** Branch `docs/add-hero-gif` unless repo convention differs. Co-author footer **default OFF** — only on explicit opt-in. User-owns-repo → push to origin; doesn't-own → `gh repo fork --clone=false` then `gh pr create --repo <upstream> --head <user>:<branch>`. Not authed correctly → stop and ask.
- **Hand-off.** Report PR URL, file size, placement path, one-line README diff. End with a single-line `SHOWCASE.md` invite — never let it crowd the hero hand-off, never re-surface it.
- **Local-only opt-out.** If no PR wanted, leave the file in `repo-visuals-work/<repo-name>/` and print the path; don't touch the target repo.

Read `craft/ship.md` end-to-end before running this phase the first time on a given repo.

---

## Phase 6 — Evaluate *(dev mode only — skip in normal runs)*

**Gate:** run this phase only when dev mode is active (user said "dev mode" this run, or `REPO_VISUALS_DEV=1`). In every other run — Manual, Semi-auto, Auto — stop after Phase 5. Do not ask user-rating questions, do not write run logs, do not mention this phase.

In dev mode, score the **final artifact** (not the process) across four rater types — User (3 rows), Claude (1 row), Code (auto via `scripts/evaluate.js`), AI (vision pass on extracted keyframes, blind to prior chat). Use the labeled 1–5 scale (`5 / Excellent` style). Skip User rows in Auto mode and flag accordingly.

Write run logs to `./evaluations/runs/<YYYY-MM-DD>-<slug>.md` in the user's working directory (NOT the plugin cache — wiped on `/plugin update`). Curated rolling stats live in `./evaluations/index.md`, edited by the `repo-visuals-retro` meta-skill.

Full criteria tables, the fixed AI-eval prompt (with anchored ground-truth and the "default to 3, lower-by-one if you can name an improvement" protocol), the `AskUserQuestion` shape for User rows, and the scorecard template all live in `craft/evaluate.md`. Read it before running this phase.
