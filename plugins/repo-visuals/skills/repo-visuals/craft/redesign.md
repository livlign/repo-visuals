# Redesign vs greenfield branch

Triggered from `SKILL.md` §1.3a when the scan finds an existing hero. Many repos already have one (the README's first image, an `assets/hero.*`, a banner SVG). Don't silently overwrite it — the existing hero is signal about what the maintainer already chose, and "redesign" is a meaningfully different shape from "fresh".

Run this branch *before* §1.4.

---

## Step 1 — surface what's there

In the scan summary, call out the existing hero explicitly: file path, format (PNG / GIF / SVG / video), rough age (last-modified date), and whether the README links to it from the top. Don't editorialize yet; just put it in front of the user.

## Step 2 — ask the entry mode

Use `AskUserQuestion`:

```
AskUserQuestion({
  questions: [{
    header: "Entry mode",
    question: "There's already a hero at <path>. How do you want to approach this run?",
    multiSelect: false,
    options: [
      { label: "Redesign (carry forward)", description: "Critique the existing hero, keep what works (palette, wordmark, archetype), iterate from there. Preserves brand equity. Best when the existing hero is recent or maintainer-loved." },
      { label: "Fresh (ignore existing)",  description: "Treat the repo as greenfield — propose archetypes from scratch. Best when the existing hero is stale, off-brand, or the user explicitly wants a reset." },
      { label: "Replace in place",          description: "Keep the existing file's exact dimensions, palette anchors, and placement contract; produce a drop-in successor. Best when downstream consumers (docs site, social cards) depend on the current file's shape." }
    ]
  }]
})
```

If the user picks **Fresh**, exit this branch and proceed to §1.4 normally.

## Step 3 — critique pass (Redesign or Replace-in-place)

Before asking direction questions, write a short critique of the existing hero — 4–6 bullets, evidence-grounded:

- **What works.** The palette, the archetype, the wordmark treatment, a specific scene beat — anything worth carrying forward. Be concrete; "the type pairing" beats "the typography".
- **What's tired.** Drift-prone claims that have aged out (a `v1.9.2` from two years ago, a `2.4k stars` that's now 12k), generic AI-design tells (NY Times pastiche, see §2.4), legibility issues at GitHub's render size, archetype mismatch with the repo's current scope.
- **What's missing.** The hero moment that would land *now* but didn't when the original was made — new capabilities, a re-pitched audience, a stronger inventory count.
- **Brand-equity items to preserve.** Palette swatches (hex), wordmark glyph or font, any visual anchor README readers already associate with the repo. These become hard constraints in §1.4 Q7.

Show the critique to the user. Ask one yes/no: *"Does this match how you see the existing hero?"* If they push back, take the correction (their read of the existing hero outranks Claude's) before moving on.

## Step 4 — carry forward into the brief

When you reach §1.4, pre-fill the direction batch with what the critique committed to:

- **Hard constraints (Q7)** include the brand-equity items from Step 3.
- **Vibe (Q4) and Energy (Q5)** default to the existing hero's read, unless the critique specifically argued for a shift.
- **Output format (Q2)** defaults to the existing hero's format under **Replace-in-place**; under **Redesign**, run §1.4c normally — the existing format is data, not a constraint.
- **Dimensions (§1.7)** default to the existing hero's exact dimensions under **Replace-in-place**; under **Redesign**, allow §1.7 to argue for a different shape.

## Scenario proposal under redesign (§1.5)

At least one of the 2–3 scenarios should be an *evolution* of the existing hero (same archetype, refreshed execution). The other(s) can diverge. Don't propose three "different direction" scenarios when the user picked Redesign — they signaled they want continuity.

## Hand-off note

Mention in the §5.5 hand-off summary that this was a redesign of the prior `<path>`, not a fresh build — useful when the PR description is auto-drafted.
