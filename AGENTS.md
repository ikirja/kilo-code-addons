# AGENTS.md

Personal Kilo code-addons repo: global skills and slash commands for the user's Kilo setup. Pure Markdown — no build, test, lint, or package step.

## Layout and deploy model

- `skills/<name>/SKILL.md` — skills, loaded only if mirrored to `~/.config/kilo/skills/<name>/SKILL.md`.
- `commands/<name>.md` — slash commands (`/tests-docs`, `/commit`, `/review-loop`, `/release`, `/tag-push`, `/analyze-video`, `/analyze-docs`); mirror to `~/.config/kilo/commands/`.
- Changes here are NOT picked up automatically. After editing a skill or command, copy it to `~/.config/kilo/skills/<name>/SKILL.md` or `~/.config/kilo/commands/<name>.md` (the copy is a separate file, not a symlink) — that is what Kilo actually loads at runtime. E.g. `skills/vision` → `~/.config/kilo/skills/vision`, `skills/bitrix24` → `~/.config/kilo/skills/bitrix24`, `skills/ecosystem-analysis` → `~/.config/kilo/skills/ecosystem-analysis`, `commands/review.md` → `~/.config/kilo/commands/review.md`.

## Skill format

Each SKILL.md needs YAML frontmatter with `name` and `description`; the description is what the skill picker shows. Skills must be invoked via the `skill` tool — an agent should not paste skill contents into prompts.

## Platform notes (vision skill)

- `skills/vision/SKILL.md` is now cross-platform: temp files use `tempfile.gettempdir()`, and the API call is pure python3+urllib (no curl). Still macOS-only: `sips` for image compression and `brew install ffmpeg`.
- **ffmpeg is NOT installed on this Linux host** — the skill's video workflow will exit with `MISSING_FFMPEG` until installed (`sudo apt install ffmpeg` on Debian/Ubuntu; Fedora needs RPM Fusion — see README).

## Secrets

`SKILL.md` contains no API keys. The vision skill resolves the RouterAI key at runtime from the `ROUTERAI_API_KEY` env var or from `~/.config/kilo/kilo.jsonc` (`provider.routerai_ru.options.apiKey`, outside this repo). The bitrix24 skill resolves its webhook from the `BITRIX24_WEBHOOK` env var or `~/.config/kilo/bitrix24.json` (outside this repo). Never commit or paste keys or webhooks into new files, commits, or chat.
