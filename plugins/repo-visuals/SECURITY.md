# Security notes

## Tessl / Snyk "Critical" flag

Tessl's registry surfaces a Snyk "Critical — review before installing" badge. The skill itself executes nothing surprising: the runtime is HTML rendered in a headless Chromium plus an `ffmpeg` invocation on locally-captured frames, both invoked via `spawnSync` with array args (no shell interpolation). The flag is almost certainly tracking the runtime dependency, not the skill code.

## What runs at install / use time

| Component | Source | Risk surface |
|---|---|---|
| `puppeteer` (npm) | `package.json` pins `24.42.0` — pulls a bundled Chromium (~170 MB) on `npm install` | Standard npm + browser supply chain. Pinned exact; bumps are manual (quarterly cadence). |
| `ffmpeg` (system) | User's package manager (`brew`, `apt`, `choco`) — or the Windows fallback in §4.1 of `SKILL.md` | The Windows fallback downloads a portable build from `https://github.com/GyanD/codexffmpeg/releases/latest`. That URL is in the recipe text, not auto-executed; the user is asked before installation. |
| `node scripts/capture.js` / `screenshot.js` / `evaluate.js` | Skill-local | All `child_process` calls use `spawnSync(cmd, [...args])` — no shell, no string interpolation. |
| HTML rendered | Authored per-run by Claude in `repo-visuals-work/<repo>/index.html` | Loaded `file://`, no network beyond Google Fonts `@import` (default). Determinism rules in §2.4 forbid `Math.random()` / `Date.now()`-based logic but don't restrict outbound resources — review before trusting a generated `index.html` to run offline. |
| `gh` CLI | User-invoked in Phase 5 for PR opens | Uses the user's existing `gh auth` session; the skill never sets credentials. |

## What it does not do

- No telemetry, no network calls from the skill itself outside `puppeteer`'s normal page loads.
- No automatic install — every `npm install` / `brew install` / portable-binary download is gated on an explicit user prompt (§4.1).
- No write access to the target repo until Phase 5, after the user has previewed the artifact (§4.0).
- No co-author or provenance commits without explicit opt-in (Phase 5 defaults).

## Hardening you can do

- `puppeteer` is pinned exactly. Bump manually with `npm install puppeteer@latest` then smoke-test `scripts/capture.js` and `scripts/screenshot.js`.
- Run `npm audit` after install; the skill has no other runtime npm deps.
- Skip the Windows portable-ffmpeg path and install via package manager if the third-party release URL is a concern.
