# Phase 6 — Evaluate (dev mode only)

**Gate:** run this phase only when dev mode is active (user said "dev mode" this run, or `REPO_VISUALS_DEV=1`). In every other run — Manual, Semi-auto, Auto — stop after Phase 5. Do not ask user-rating questions, do not write run logs, do not mention this phase.

Score the **final artifact**, not the process. In dev mode, always runs at the end.

---

## 6.1 Criteria (four rater types)

Each criterion is rated by exactly one of: **User**, **Claude** (subjective, chat-blind), **Code** (deterministic script), or **AI** (Claude re-reads the final GIF/HTML with vision, blind to prior chat).

**User-rated (3) — viewer-side truth.**

| Criterion | What it measures | Signal for low score |
|---|---|---|
| **Hero moment delivery** | Does a cold viewer "get it" in ~10 seconds — both *what the repo is* and *why they'd reach for it*? | After one loop, viewer still can't state the repo's purpose or motivation |
| **Visual impact** | Does the artifact make the viewer *want* to try the repo? | Looks fine but feels generic; no pull |
| **Ship-worthiness** | Gut check: would the user paste this into the repo's README today, as-is? | User hesitates, wants "one more pass" |

**Claude-rated (1) — repo-fit judgment.**

| Criterion | What it measures | Signal for low score |
|---|---|---|
| **Repo fidelity** | Do on-screen text, terminology, and vibe feel like this specific repo's own voice? | Headlines read like generic marketing; terminology drifts from README |

**Code-evaluated — `scripts/evaluate.js` runs automatically after export. Rows depend on format.**

| Criterion | Applies to | Pass rule | Fail signal |
|---|---|---|---|
| **File size** | GIF + PNG | GIF: ≤ 10 MB target / ≤ 15 MB cap. PNG: ≤ 500 KB target / ≤ 1 MB cap. | Over target → 3; over cap → 1 |
| **Dimensions** | GIF + PNG | Matches spec (e.g. 1200×675). PNG at 2× device pixel scale is also acceptable. | Wrong size → 1 |
| **Loop duration** | GIF only | 15–25 s (hero default) | Outside band → 2 |
| **Loop seam** | GIF only | First-frame vs last-frame pixel diff under ~2% | Visible jump on loop → 2 |
| **Palette size** | GIF only | Palette ≤ 256, no visible banding on solid regions | Banding detected → 2 |

**AI-evaluated (4) — Claude re-opens the exported artifact with vision, blind to prior chat. Prompt in §6.3.**

| Criterion | What it measures | Signal for low score |
|---|---|---|
| **Legibility** | Every headline readable at native render size, no sub-pixel smear | Any headline unreadable → 2 |
| **Scene clarity** | Each scene conveys one idea in its airtime | Two scenes blur together or one feels like filler → 3 |
| **Voice match** | Headlines match tone and terminology of the repo's README | Drift from repo's own language → 2 |
| **Intent delivery** | After one loop, can a cold viewer state *why* to reach for this repo? | Demos *what* without delivering *why* → 3 |

---

## 6.2 Scale (1–5, labeled)

| Score | Label | Meaning |
|---|---|---|
| 1 | Poor | Falls apart on the criterion |
| 2 | Weak | Noticeably misses |
| 3 | OK | Gets there, nothing more |
| 4 | Strong | Clearly delivers |
| 5 | Excellent | Best-in-class for this repo |

Use the labels, not bare numbers. A "3" alone is noise; "3 / OK" is meaningful.

---

## 6.3 Hand-off scorecard

Assemble in four steps:

1. **Run code eval** — `node scripts/evaluate.js <path-to-hero.gif-or-png>` → emits a JSON scorecard.
2. **Run AI eval** — extract 4–6 keyframes from the GIF first (`ffmpeg -ss <t> -i hero.gif -update 1 -frames:v 1 frame.png` at evenly spaced times), then re-read each frame with vision, blind to prior chat. Use this fixed prompt — **anchored, evidence-required, and fed the repo's real inventory** so factual drift is catchable:

   > You are evaluating a hero GIF for the repo `<owner/repo>`.
   >
   > **Repo ground truth (from §1.3 scan):**
   > - README excerpt (first 40 lines): `<excerpt>`
   > - Real inventory count: `<N>` (e.g. "30 tools", "12 commands", "N/A"). Specific names: `<list if applicable>`.
   > - Stated hero moment (from §1.6 brief): `<one sentence>`
   >
   > **Rating protocol — read carefully:**
   > - **Default every score to 3 / OK.** A 3 means "gets there, nothing more." Only move up with specific visual evidence from the frames; only move down with specific visual evidence of a problem.
   > - **4 / Strong requires one concrete observation** from the frames that the criterion is clearly delivered (cite it in the note).
   > - **5 / Excellent requires two concrete observations AND that you cannot name a realistic improvement.** If you can name one, cap at 4.
   > - Do not grade on effort, intent, or potential. Grade only what is visible in the frames.
   > - Compare on-screen claims against the repo ground truth. If the hero says *"all"* / *"every"* / shows a grid of N items but the repo has more, **cap Intent delivery at 2 / Weak** and note the undercount. An unverified claim is weaker than a verified one.
   >
   > **Rate each of Legibility, Scene clarity, Voice match, Intent delivery (1–5 Poor/Weak/OK/Strong/Excellent)** with a one-sentence justification citing specific frame evidence. Do not reference any prior conversation — judge only what you see in the frames and read in the ground truth above.
   >
   > **At the end, for any score ≥ 4, answer: "what specific change would push this to 5?" If you have an answer, lower the score by one.**

3. **Fill Claude's repo-fidelity row** with a one-sentence justification.
4. **Ask the user** for the 3 User rows via the `AskUserQuestion` tool — **in Manual and Semi-auto only**. In **Auto** mode, skip the User rows entirely; compute the overall average from Code + AI + Claude rows alone and flag in the run file that User ratings were not collected (so the evaluations index can weight it correctly).

   Use four questions in one call: Hero moment delivery, Visual impact, Ship-worthiness, and a fourth free-text-via-Other for the one-line feedback. Structure each rating question with four labeled options matching the scale (omit 5 to fit the 4-option max; users can pick "Other" to enter 5/Excellent or a custom score). Example shape:

   ```
   AskUserQuestion({
     questions: [
       {
         header: "Hero moment",
         question: "Hero moment delivery — after one loop, would a cold viewer get *what* this repo is *and why* they'd reach for it?",
         options: [
           { label: "2 / Weak", description: "Noticeably misses" },
           { label: "3 / OK", description: "Gets there, nothing more" },
           { label: "4 / Strong", description: "Clearly delivers" },
           { label: "1 / Poor", description: "Falls apart" }
         ],
         multiSelect: false
       },
       { header: "Visual impact", question: "Visual impact — does it make you want to try the repo?", options: [/* same 4 */], multiSelect: false },
       { header: "Ship-worthiness", question: "Ship-worthiness — would you paste this into the README today, as-is?", options: [/* same 4 */], multiSelect: false },
       {
         header: "Feedback",
         question: "One line of free-text feedback — the single most useful signal for next time.",
         options: [
           { label: "Nothing to add", description: "Skip this round" },
           { label: "Add a comment", description: "Pick 'Other' to type your line" }
         ],
         multiSelect: false
       }
     ]
   })
   ```

   The "Other" escape hatch covers 5 / Excellent and any other custom response. Capture the returned labels + any `annotations.notes` or "Other" text into the scorecard.

Display the completed table, grouped by rater, then compute an overall simple average. Keep the full table in the run file.

```
| Criterion            | Rater  | Score        | Note                                                   |
|----------------------|--------|--------------|--------------------------------------------------------|
| Hero moment delivery | User   | (ask user)   | (ask user)                                             |
| Visual impact        | User   | (ask user)   | (ask user)                                             |
| Ship-worthiness      | User   | (ask user)   | (ask user)                                             |
| Repo fidelity        | Claude | 4 / Strong   | Mirrors README phrasing; tagline could be tighter.     |
| File size            | Code   | 5 / Excellent| 2.4 MB (10 MB target).                                 |
| Dimensions           | Code   | 5 / Excellent| 1200×675 matches spec.                                 |
| Loop duration        | Code   | 5 / Excellent| 20.1 s inside 15–25 s band.                            |
| Loop seam            | Code   | 4 / Strong   | 1.3% first/last-frame diff (threshold 2%).             |
| Palette size         | Code   | 5 / Excellent| 212 colors, no banding flagged.                        |
| Legibility           | AI     | 4 / Strong   | All 7 headlines readable at native size.               |
| Scene clarity        | AI     | 3 / OK       | Cron and JWT scenes blur into each other.              |
| Voice match          | AI     | 4 / Strong   | Matches README's "one-off utility" framing.            |
| Intent delivery      | AI     | 3 / OK       | Shows *what* each tool does, not *why* a user needs it.|
```

---

## 6.4 Evaluation log (two-tier)

Write all evaluation files under `./evaluations/` in the **user's current working directory** — NOT the plugin cache, which is wiped on `/plugin update`. Create the directory if it doesn't exist on first dev-mode run.

**Tier 1 — curated aggregate (committed):** `./evaluations/index.md`
- Rolling stats per criterion across runs
- Notable lessons learned
- Recurring failure modes
- Edited by the meta-skill (Phase 6.5) during retros

**Tier 2 — raw per-run files (gitignored by default):** `./evaluations/runs/<YYYY-MM-DD>-<slug>.md`
- The brief
- The scorecard
- User's free-text feedback
- Path to the archived `hero.gif` / HTML if user opts to keep them

User can opt in to commit specific runs (typically OSS repos they're happy to publicize).

---

## 6.5 Meta-skill: `repo-visuals-retro`

A separate skill (not part of `repo-visuals`'s runtime) for improving the skill itself. See `../repo-visuals-retro/SKILL.md` for its own design. Invoked on-demand when you have enough samples to spot patterns — not automatically per run.
