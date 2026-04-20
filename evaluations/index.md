# repo-visuals · evaluation index

Curated aggregate across runs. Raw per-run scorecards live in `runs/` (gitignored).

## Rolling stats (n=2)

| Criterion | Avg | Runs |
|---|---|---|
| Hero moment delivery | 2.5 | 2 |
| Visual impact        | 3.5 | 2 |
| Ship-worthiness      | 2.5 | 2 |
| Repo fidelity        | 4.5 | 2 |
| Technical polish / Code rows | 4.6 | 1 (v2 only — expanded rubric) |
| AI rows              | 4.25 | 1 (v2 only) |
| **Overall**          | 3.8 | 2 |

Breakdown — v1 under old 5-criterion rubric (3.6/5); v2 under the expanded 4-rater rubric (4.0/5).

## Recurring signals (2 runs, tentative)

- **Intent delivery is the weak spot for devtool-style repos.** Both runs landed high on Repo fidelity / Technical polish but low on User Hero-moment. v1 demoed *what*; v2 demoed *what-in-one-tab*; neither fully delivered *why reach for it*.
- **User-rated criteria do not track Claude/AI/Code ratings.** In v2: Claude 5.0, AI 4.25, Code 4.6, User 2.33. Automated raters can miss factual drift (e.g. hero claims "8 tools" when repo has 30) and production-grade polish that only a human notices.

## Notable lessons

- **`2026-04-20 · emtyty/devtool v1`** — Visual polish ≠ intent delivery. A scene-per-tool walkthrough demos *what* without delivering *why*. Open with pain-framing, fewer tool beats, unifying finale.
- **`2026-04-21 · emtyty/devtool v2`** — A strong layout thesis ("one tab, all tools") is not a ship-ready hero on its own. Two new rules:
  1. **Count matters.** If the hero says "all" or shows a grid, the grid must actually carry the repo's real inventory (or explicitly say "sample of N"). A repo with 30 tools cannot be heroed with 8 cards.
  2. **Don't race to export.** Going script→capture→encode without a real preview-iterate beat (§3) leaves obvious scope issues uncaught. For users who value production-grade, slow passes beat fast ones.

## Skill-edit candidates (for retro meta-skill)

- **§6.3 AI-eval prompt** should include the target repo's actual tool/feature inventory, so the AI rater can flag factual drift (undercounts, wrong terminology) before the user has to.
- **§3 preview-iterate** should be an explicit gate before Phase 4 (Export), not a soft step. Consider a "have we actually previewed and iterated" checkbox in §1.6 convergence.

## Runs

- 2026-04-20 · emtyty/devtool v1 — 3.6 / 5 — intent not landing; visual strong
- 2026-04-21 · emtyty/devtool v2 — 4.0 / 5 — layout thesis works; undercounts the repo; rushed export
