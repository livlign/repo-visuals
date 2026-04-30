# livlign skills

![livlign/claude-skills — bespoke heroes, honest audits. Two Claude Code plugins for working with GitHub repos: repo-visuals designs README hero visuals, repo-doctor audits READMEs and prescribes fixes.](./assets/hero.gif)

Claude Code plugins for working with GitHub repos — designing README heroes, auditing READMEs.

```
/plugin marketplace add livlign/claude-skills
```

## Plugins

| Plugin | Install | What it does |
|---|---|---|
| [repo-visuals](./plugins/repo-visuals) | `/plugin install repo-visuals@livlign` | Designs bespoke README hero visuals — animated GIF or static PNG — through a structured discovery dialog. |
| [repo-doctor](./plugins/repo-doctor) | `/plugin install repo-doctor@livlign` | Audits a repo's README against best-practice patterns and produces a prioritized P0/P1/P2 punch list of fixes. |

---

## `repo-visuals`

Scans the target repo, recommends a format that fits its identity, proposes scenarios, designs a bespoke HTML stage, previews it in your browser, and exports a retina-quality artifact — then optionally opens the upstream PR.

**What a run looks like:**

```
> /repo-visuals
  pick mode → semi-auto · scan repo → 30 tools detected
  recommend → animated GIF, 1200×675 · brief locked
  build HTML → preview in browser → ship
  export → hero.gif (2.4 MB) · open PR #42
```

**Why it's different**

- **Discovery dialog, not templates.** Every hero is bespoke. A structured scan-then-design conversation runs before any HTML is written.
- **Production-grade by default.** Retina capture, drift-proof copy (no stale star counts or version numbers), preview-vs-export parity, scope-match guardrails — encoded from real maintainer pushback.
- **Three modes.** Manual, Semi-auto (recommended), Auto.
- **Optional PR.** Local-only or full upstream PR with relative-path embed, alt-text generation, and fork-and-PR for repos you don't own.

[Skill source →](./plugins/repo-visuals)

---

## `repo-doctor`

Audits a README the way a senior maintainer would — does the "what is this" sentence land, is install-to-first-success short enough, does the audience-jargon match, are there drift signals (stale versions, badge sprawl, dead links). Read-only diagnostic by default; opens a PR only on explicit ask.

**What a run looks like:**

```
> /readme-doctor
  scan repo → classify category (CLI tool · library · framework · …)
  audit → score 9 criteria with evidence
  output → P0/P1/P2 punch list, each with a one-line fix
  health score: 69/100
```

[Skill source →](./plugins/repo-doctor)

---

## Contributing

Easiest entry points: submit a hero to the [showcase](./plugins/repo-visuals/SHOWCASE.md), report a real failure as an issue, or pick something from the [wishlist](./CONTRIBUTING.md#3-pick-something-from-the-wishlist).

Full guide → [CONTRIBUTING.md](./CONTRIBUTING.md)

Personal project, evolving fast — expect breaking changes pre-1.0.
