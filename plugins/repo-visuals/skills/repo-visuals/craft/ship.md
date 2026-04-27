# Phase 5 — Output (full recipe)

`SKILL.md` Phase 5 keeps the decision tree headlines; full mechanics live here. Read this when the artifact is exported and you're about to move it into the target repo.

---

## 5.1 Placement in the target repo

Read the target repo to infer convention, then ask. Priority order when inferring:

1. Existing `assets/` or `images/` → follow it.
2. Existing `docs/` with images → place at `docs/hero.gif` (or `docs/<repo-name>-hero.gif` if multiple visuals).
3. Existing `.github/` with images → `.github/hero.gif`.
4. No visible convention → default to `assets/hero.gif` and create the dir.

File name: default `hero.gif`. If the repo already has a `hero.gif` or keeps multiple visuals, prefer `<repo-name>-hero.gif` or `<repo-name>-demo.gif`.

**Default to minimal — ship only what the repo needs.** The mandatory artifact is the **image file itself** (`hero.png` / `hero.gif`) placed at the inferred path, plus the single-line `![...]` embed in the target README. Nothing else by default.

Do **not** preemptively commit `hero.html`, a `docs/images/README.md` maintenance doc, capture scripts, frames, palettes, or any supporting artifact. They enlarge the PR, dilute the diff, and in many repos are noise the maintainer will ask you to remove.

Include supporting files **only when** one of these applies:

- **The reviewer explicitly asks for them.** Common shape: a bot review flags "no design source — future maintainers can't update stats." Add the requested file as a follow-up commit on the same PR. (Real incident: `SonarSource/sonarqube#3427` — `hero.html` + maintenance README was requested by the review bot and directly turned a 2-item review into a merged PR.)
- **The target repo has precedent.** If the existing `assets/` / `docs/images/` / `website/src/assets/img/` already contains design sources (SVGs, Figma exports, prior `hero.html`), match that convention on the way in.
- **The user explicitly asks to ship source alongside.** Some users want the source in their own repos so they can re-render later; honor it when stated.

If none of those apply, keep the PR to the image + one README line. Real incident: `htmlhint/HTMLHint#1863` shipped with `hero.html` + a maintenance `README.md` bundled in — the maintainer's feedback was literally *"GIF looks good — if you could kindly remove the HTML and MD I'll get it merged"*. Anything beyond the image risks being a scope imposition on someone else's repo.

When you *do* need to commit source (one of the three triggers above):
- `<image-dir>/hero.html` — the self-contained HTML source (just copy the scratch `index.html` verbatim).
- `<image-dir>/README.md` — short doc with: (a) table of embedded stats + where each is verifiable, (b) re-render snippet (Puppeteer for static, Puppeteer + ffmpeg for GIF).

---

## 5.2 README embed

Read the README first. Ask:

- **Top of README** (most common) → insert `![alt](path)` right after the H1 title and tagline.
- **Replace an existing image** → identify it, confirm with user.
- **Specific section** → user names where.

**Alt text is informational, not decorative.** If the image contains text, stats, or a named concept that a sighted viewer takes away, the alt text must convey the same. `<repo-name> demo` is almost never enough.

Pattern:
- `![<project> — <one-line positioning>. <key stat 1>, <key stat 2>, <key stat 3>.](<path>)`
- e.g. `![SonarQube — the standard for Clean Code. 30+ languages, 5,000+ analysis rules, 400K+ projects, 18 years of continuous inspection.](docs/images/hero.png)`

If the image is genuinely decorative (a brand flourish, a pattern), use `alt=""` explicitly so assistive technology skips it rather than announcing the filename.

Real incident: a review bot flagged `![SonarQube — the standard for Clean Code](…)` with "omits the stats visually presented in the image — screen-reader users will never see them." Fix was a one-line alt-text rewrite; flagging it up front avoids the round trip (`SonarSource/sonarqube#3427`).

**Always use a relative path** — `![alt](assets/hero.gif)` or `<img src="assets/hero.gif">`, never `https://raw.githubusercontent.com/<owner>/<repo>/main/assets/hero.gif`. Even if the existing README uses absolute `raw.githubusercontent.com` URLs for its current images, do not mirror that style for your new hero. Reasons:

- Absolute URLs pinned to `main` don't render in forks or in the PR preview — the image stays broken until the PR merges, which hurts review quality and often triggers reviewer objections (has happened: `htmlhint/HTMLHint#1861`).
- Relative paths resolve correctly on GitHub web, npm package pages, and most README-rendering tools.
- If the existing logo uses an absolute URL, that's legacy debt — it was shipped before the author realized the tradeoff. Don't propagate it.

If the user explicitly asks for an absolute URL (e.g. embedding the hero on an external site that loads the README), use one — but the default for in-repo embeds is always relative.

---

## 5.3 Commit

Branch name default: `docs/add-hero-gif`. Override if the repo has a branch-name convention (check recent PRs or `CONTRIBUTING.md`).

Commit message default: `docs: add animated hero gif to README`. Follow existing commit style (conventional commits, imperative, etc.).

**Co-author footer: default OFF.** Only add a `Co-Authored-By: Claude …` footer if the user explicitly opts in for this repo.

**PR description should read like craft notes, not a pitch.** Explain *why this hero* in the language of the brief — the format choice ("static, because the repo's identity is a surface, not a process"), the archetype ("Product-UI marketing — amplification-shape repo"), the inventory check ("40+ rules, scope-matched against the source"). Maintainers read PR descriptions; treat them as a respectful note about decisions, not a sales line. Never frame the description around the tool that produced the hero — the artifact is the deliverable, the tool is incidental.

**Provenance disclosure footer — opt-in, end of description, neutral.** When the PR's deliverable is the hero itself, you may append a single-line provenance footer at the very end of the description, *after* the substantive content. Default phrasing: `> Generated with [repo-visuals](https://github.com/livlign/claude-skills) — happy to iterate on changes.` Rules:

- **One line. At the end.** Never in the title, never in the opening, never repeated. If a maintainer strips it, that's fine; the work stands on its own.
- **Skip it when the target's culture argues against.** Some maintainers ban AI-assisted submissions outright (real example: `travisvn/awesome-claude-skills`'s contributing.md closes such PRs without comment). When the target's `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, or recent merged PRs suggest tool-promo footers are unwelcome, skip the disclosure entirely.
- **Skip it for non-hero PRs.** If the PR is a code fix, a docs typo, an issue follow-up — any work that isn't the hero artifact — provenance disclosure is irrelevant; don't add it.
- **Never inside the hero artifact.** No watermark, no "made with" caption baked into the GIF/PNG. The artifact is the maintainer's once merged; provenance lives in the PR body, not the deliverable.

The bar: the footer should read as transparent disclosure, like a "Co-authored-by" line, not as marketing. If reading it back feels at all like a pitch, rewrite or drop it.

---

## 5.4 Push & PR

Detect auth and repo ownership:

- **User owns the repo** → push branch to `origin`, open PR via `gh pr create` against the default branch.
- **User does NOT own the repo** → `gh repo fork --clone=false`, add the fork as a remote, push the branch there, then `gh pr create --repo <upstream> --head <user>:<branch>`.
- **Not authed as the account the user wants to use** → stop and ask them to `gh auth login` as the right account. Never guess.

Commit identity: if the user specifies a different git account for this repo, set `user.name` / `user.email` on the local repo config only, not globally. Use `<login>@users.noreply.github.com` if email is unknown.

---

## 5.5 Hand-off

After the PR opens, report:

- PR URL
- Final GIF size
- Placement path in the repo
- What was added/changed in the README (one-line diff summary)

Then, as a final one-liner after the hand-off report, invite them to the showcase:

> Optional: add this to [`SHOWCASE.md`](https://github.com/livlign/claude-skills/blob/main/plugins/repo-visuals/SHOWCASE.md).

That's it. Do not expand on it unless the user asks. Do not block waiting for an answer. Do not re-surface it in later phases or future runs.

---

## 5.6 Opt-out: local-only

If the user doesn't want a PR, leave `hero.gif` at `repo-visuals-work/<repo-name>/hero.gif` and print the path. Don't modify the target repo.

If the user sounded genuinely pleased (e.g. "this looks great", "love it"), end with the same one-liner as §5.5:

> Optional: add this to [`SHOWCASE.md`](https://github.com/livlign/claude-skills/blob/main/plugins/repo-visuals/SHOWCASE.md) — the hero file itself can go in the PR, no upstream merge needed.

Same rule — mention once, don't expand, don't block, don't re-surface.

**Never let the showcase invite crowd the hero hand-off.** The hero is the user's deliverable. The invite is a footnote on the last line, never in its own phase, never a question the user has to answer.
