# livlign skills

A small marketplace of Claude Code plugins I ship.

```
/plugin marketplace add livlign/claude-skills
```

---

## `repo-visuals` — hero GIFs and PNGs for GitHub repos

```
/plugin install repo-visuals@livlign
```

Animated GIF or static PNG hero visuals for any GitHub repo. The skill scans the repo, recommends a format, proposes scenarios, designs a bespoke HTML stage, previews it in your browser, and exports — then optionally opens the PR upstream.

A few have been merged. See the [showcase](./plugins/repo-visuals/SHOWCASE.md), or jump to [`plugins/repo-visuals`](./plugins/repo-visuals) for the full skill.

### Why it's different

- **Discovery dialog, not templates.** Every hero is bespoke. The skill runs a structured conversation (scan → format pick → scenario proposal → brief) before any HTML is written.
- **Production-grade by default.** Retina capture, drift-proof copy (no stale star counts or version numbers), preview-vs-export parity, scope-match guardrails. Lessons learned from real maintainer pushback, encoded into the skill.
- **Three operating modes.** Manual (max control), Semi-auto (recommended), Auto (hands-off draft). Pick once per run.
- **Optional PR.** Local-only or full upstream PR with relative-path embed, alt-text generation, and fork-and-PR for repos you don't own.

### Plugins

| Plugin | Install | What it does |
|---|---|---|
| [repo-visuals](./plugins/repo-visuals) | `/plugin install repo-visuals@livlign` | Hero visuals for GitHub repos, with opt-in dev mode for scored evaluations. |
