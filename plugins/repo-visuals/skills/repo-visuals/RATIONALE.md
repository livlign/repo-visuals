# Rationale — design choices that look like flaws to a generic linter

`SKILL.md` deliberately makes a few choices that score poorly under "shorter is better, templates everywhere" rubrics. They're load-bearing, not oversights.

## Why there's no copy-paste HTML skeleton

The skill's stated thesis, line 10 of `SKILL.md`:

> The skill's quality comes from the **discovery dialog**, not from templates. Every hero is bespoke.

A 30-line skeleton would either be ignored (because it can't fit five archetypes — terminal-demo / product-UI / brand-first / banner / diagram-as-hero) or used by default, which collapses bespoke output toward sameness. That's the **generic AI-design** failure §2.4 explicitly warns against ("avoid the NY Times / editorial-newspaper theme — it's become a default AI-design shortcut").

What ships instead: `craft/templates/*.html` — full working heroes from past upstream-merged runs, archetype-tagged. Used as **patterns to steal**, not as starting frames. §2.2a tells Claude to read them end-to-end before drafting.

If a reviewer flags "no skeleton" as a defect, the answer is: that's the point.

## Why rationale stays inline (not relocated to `craft/rules.md` wholesale)

A linter that rewards short SKILL.md files will push toward terse rule statements with the *why* hidden in pointers. That's a known regression mode in this skill: a rule without its reason gets mechanically applied (or quietly dropped on edge cases) because the next operator can't tell what it's protecting against.

Concrete cases where the inline **why** is what made the rule survive contact with reality:

- **§2.3a stage-clip CSS** carried an inline incident note about SonarQube and everything-claude-code. Two later runs hit the same shape; the reason was right there for the agent to recognize.
- **§1.3 stat-rot rule** carries an inline incident about a hero merged with stale star counts. A pointer to "see rules.md" would have been read past.
- **§2.4 fake-app-UI rule** is hard, not soft, because two prior runs (Terminal.Gui, clawd-on-desk v1–v2) failed it. That's stated inline so it can't be downgraded to "soft preference" by a future condensation pass.

What *did* move to `craft/`:
- Long incident write-ups (`craft/rules.md`)
- Recipe boilerplate (`craft/export.md`, `craft/ship.md`, `craft/evaluate.md`)
- Reference patterns (`craft/headlines.md`, `craft/reference-gallery.md`, `craft/templates/`)
- Decision-time JSON shapes that are pure boilerplate (`craft/snippets/`)

What stays inline: the **why** behind every non-obvious rule, in one short sentence each. This is the explicit guidance the skill author works under (see memory: "Production-grade over speed" — rules need their reasons so edge cases stay judgable).

## What would change my mind on either of these

- Skeleton: a pattern that genuinely covers ≥80% of archetypes without flattening their identity. The current evidence from `craft/templates/` is that the five archetypes diverge structurally enough that a single skeleton would only fit one of them well.
- Rationale extraction: a delivery format where the agent reliably reads `rules.md` *before* applying any rule, not just on first cold start. Pointer-based docs lose to inline prose at the moment of decision; if that changes, the layout should change.
