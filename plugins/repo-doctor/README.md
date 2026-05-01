# repo-doctor

A Claude Code plugin for **diagnosing GitHub README problems** — a structured audit against patterns that correlate with maintainer outcomes (hero presence, install-to-first-success length, "what is this in one sentence" clarity, audience-jargon match, drift signals), output as a prioritized punch list.

Currently ships one skill: **`readme-doctor`**. Read-only by default; opens a PR only when the user explicitly asks.

## Install

```
/plugin marketplace add livlign/claude-skills
/plugin install repo-doctor@livlign
```

After install, restart Claude Code (or start a new session) so the plugin's skills are discovered.

## How to run

> Audit the README at https://github.com/owner/name

Claude asks which operating mode to use (Auto / Semi-auto / Manual), scans the repo, infers its category and audience, walks a 9-criterion rubric, and produces a punch list grouped by priority:

- **P0 (blocking)** — issues that determine whether a stranger trying the repo bounces or stays.
- **P1 (high-leverage)** — fixable in an afternoon, meaningful payoff.
- **P2 (nice-to-have)** — polish.

Each item has one line on what's wrong (with line numbers), one line on why it matters for this repo's audience, and one line on a concrete fix.

By default the punch list lives in the chat. If you say *"open a PR for the P0/P1 items,"* Claude shows the diff first and then opens the PR upstream — never auto-applies.

## Operating modes

Same three modes as `repo-visuals` — Auto, Semi-auto (default), Manual. Mode affects how many decisions are silent vs surfaced; it does not skip rubric checks.

| Mode | What **you** decide | Typical back-and-forths |
|---|---|---|
| **Auto** | nothing | 0 |
| **Semi-auto** _(default)_ | category override (if Claude misclassified), which priority tier to act on | ~2 |
| **Manual** | every decision point | 5–8 |

## What the rubric covers

1. The "what is this" sentence
2. Hero presence above the fold
3. Install-to-first-success length
4. Concrete example before API docs
5. Audience-jargon match
6. Scannability
7. Drift signals (stale versions, dead links, badge sprawl, outdated screenshots)
8. "What this isn't" honesty (for non-1.0 / personal projects)
9. Contribution surface (for repos that want contributors)

Each criterion is scored 1–5 with cited evidence. Rubric weighting adapts to the inferred repo category (library, CLI, framework, app, spec, personal project).

## Layout

- [`skills/readme-doctor/SKILL.md`](./skills/readme-doctor/SKILL.md) — the audit skill (discovery → audit → punch list → optional PR).
