# livlign skills

A small marketplace of Claude Code plugins I ship.

```
/plugin marketplace add livlign/claude-skills
```

---

## `repo-visuals` — bespoke GitHub README heroes

```
/plugin install repo-visuals@livlign
```

A Claude Code plugin that designs README hero visuals — animated GIF or static PNG — for GitHub repositories. The skill scans the target repo, recommends a format that fits its identity, proposes scenarios, designs a bespoke HTML stage, previews it in your browser, and exports a retina-quality artifact — then optionally opens the upstream PR.

### Recent heroes

<table>
<tr>
<td width="50%" valign="top">
  <a href="https://github.com/htmlhint/HTMLHint"><img src="https://raw.githubusercontent.com/htmlhint/HTMLHint/main/website/src/assets/img/hero.gif?v=retina" alt="HTMLHint hero"></a>
  <sub><a href="https://github.com/htmlhint/HTMLHint"><b>htmlhint/HTMLHint</b></a> · merged <a href="https://github.com/htmlhint/HTMLHint/pull/1863">#1863</a></sub>
</td>
<td width="50%" valign="top">
  <a href="https://github.com/gui-cs/Terminal.Gui"><img src="https://raw.githubusercontent.com/gui-cs/Terminal.Gui/develop/docfx/images/hero.gif" alt="Terminal.Gui hero"></a>
  <sub><a href="https://github.com/gui-cs/Terminal.Gui"><b>gui-cs/Terminal.Gui</b></a> · merged <a href="https://github.com/gui-cs/Terminal.Gui/pull/5058">#5058</a></sub>
</td>
</tr>
<tr>
<td width="50%" valign="top">
  <a href="https://github.com/rullerzhou-afk/clawd-on-desk"><img src="https://raw.githubusercontent.com/rullerzhou-afk/clawd-on-desk/main/assets/hero.gif" alt="clawd-on-desk hero"></a>
  <sub><a href="https://github.com/rullerzhou-afk/clawd-on-desk"><b>rullerzhou-afk/clawd-on-desk</b></a> · merged <a href="https://github.com/rullerzhou-afk/clawd-on-desk/pull/187">#187</a></sub>
</td>
<td width="50%" valign="top">
  <a href="https://github.com/affaan-m/everything-claude-code"><img src="https://raw.githubusercontent.com/affaan-m/everything-claude-code/main/assets/hero.png" alt="everything-claude-code hero"></a>
  <sub><a href="https://github.com/affaan-m/everything-claude-code"><b>affaan-m/everything-claude-code</b></a></sub>
</td>
</tr>
</table>

[Full showcase →](./plugins/repo-visuals/SHOWCASE.md) · [Plugin source →](./plugins/repo-visuals)

### Why it's different

- **Discovery dialog, not templates.** Every hero is bespoke. The skill runs a structured conversation (scan → format pick → scenario proposal → brief) before any HTML is written.
- **Production-grade by default.** Retina capture, drift-proof copy (no stale star counts or version numbers), preview-vs-export parity, scope-match guardrails. Lessons learned from real maintainer pushback, encoded into the skill.
- **Three operating modes.** Manual (max control), Semi-auto (recommended), Auto (hands-off draft). Pick once per run.
- **Optional PR.** Local-only or full upstream PR with relative-path embed, alt-text generation, and fork-and-PR for repos you don't own.

### Plugins

| Plugin | Install | What it does |
|---|---|---|
| [repo-visuals](./plugins/repo-visuals) | `/plugin install repo-visuals@livlign` | Hero visuals for GitHub repos, with opt-in dev mode for scored evaluations. |
