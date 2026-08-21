# Kilo Code-Addons

Personal add-ons for the user's Kilo setup: **skills** (`skills/`) and **slash commands** (`commands/`). Pure Markdown — no build, test, lint, or packaging step.

## Layout

- `skills/<name>/SKILL.md` — a skill. Kilo loads it only after it is mirrored to `~/.config/kilo/skills/<name>/SKILL.md`.
- `commands/<name>.md` — slash commands (`/name`). Mirror to `~/.config/kilo/commands/`.

## Installing / syncing

Kilo does not read this repo directly. After editing a skill or command here, copy it to the global config (the copy is a separate file, not a symlink):

```bash
cp -r skills/vision ~/.config/kilo/skills/vision
cp -r skills/bitrix24 ~/.config/kilo/skills/bitrix24
cp -r skills/ecosystem-analysis ~/.config/kilo/skills/ecosystem-analysis
cp commands/*.md ~/.config/kilo/commands/
```

## Slash commands

All commands expect a git + npm repo (the session's project) and read its root `AGENTS.md` for the repo's own conventions (pre-commit script, commit style, changelog rules).

Note: Kilo ships a built-in `/review` command (single-pass review of uncommitted/staged/unpushed/branch/commit/PR changes). This repo's `/review-loop` is the automated fix loop described below — the names intentionally differ so they don't collide.

| Command | What it does |
|---|---|
| `/tests-docs` | Runs the test suite, updates stale tests; audits maintained docs (`README`, `AGENTS`, `CLAUDE`, `CHANGELOG`, `docs/`) and updates what is stale. No commit. |
| `/commit` | Runs `npm run pre-commit`, fixes every failure until green, commits with a conventional message. No push. |
| `/review-loop [N]` | Iterative review: reviews the last N commits (default 1), fixes all findings, runs pre-commit, commits fixes, and repeats with N+1 until clean (max 4 rounds). |
| `/release [patch\|minor\|major\|X.Y.Z]` | Bumps version (CHANGELOG `[Unreleased]` → `[X.Y.Z]` + link refs, `package.json`), runs pre-commit, commits the version bump (`release:` in rankup-app/landings, `chore:` in widget-pro/orbitron-hub — per repo style). Never touches `package-lock.json`. No tag. |
| `/tag-push [version]` | Creates `vX.Y.Z` on HEAD and pushes the tag (triggers CI/CD deploy). |
| `/analyze-video [path]` | Reads videos in `temp/`/`temp-docs/` (or path) via the vision skill, then follows the shared `ecosystem-analysis` skill to categorize into bugs/updates/feature suggestions and write a plan for the current app (`.kilo/plans/`) plus ecosystem info (`.kilo/plans/TEMP.md`). |
| `/analyze-docs [path]` | Same as `/analyze-video` but for `.docx`/`.pdf`/`.doc` (text extraction via unzip/python3/pandoc/libreoffice, no vision skill). |

## Vision skill

`skills/vision/SKILL.md` reads images and video through the RouterAI Gemini API (`google/gemini-3.1-flash-lite`) when the main model lacks vision. It encodes media to base64 and calls `https://routerai.ru/api/v1/chat/completions` via `python3`+`urllib`.

### Prerequisites

- `python3` and `openssl` (preinstalled on mainstream distros).
- `ffmpeg` — required for video only:
  - Debian/Ubuntu: `sudo apt update && sudo apt install -y ffmpeg`
  - Fedora (ffmpeg is not in the default repos — enable RPM Fusion first):
    ```
    sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf install -y ffmpeg
    ```
  - macOS: `brew install ffmpeg`

### API key

The skill needs a RouterAI API key and resolves it at runtime in this order:

1. `ROUTERAI_API_KEY` environment variable, or
2. the `apiKey` value under `provider.routerai_ru` in `~/.config/kilo/kilo.jsonc` (Kilo's own RouterAI provider config — the default on this machine), or
3. it exits with `MISSING_API_KEY: set ROUTERAI_API_KEY`.

So on a machine with the standard Kilo RouterAI provider config, the skill works out of the box. To use the env var instead (e.g. on machines without that config), export it — for example in `~/.bashrc`:

```bash
export ROUTERAI_API_KEY=sk-REPLACE_WITH_YOUR_KEY
```

**Secret handling:** no API key is committed in this repo. Use only the env var or the kilo.jsonc config path above — never hardcode a key into the skill, README, or any other file here.

### Behavior notes

- Images: batch up to 3 per call; ~2MB each works. Compress larger images first (see the skill's constraints section for ffmpeg/sips one-liners).
- Video: one video per call, compressed to ≤720p H.264, ≤2 minutes, mono AAC — the Gemini endpoint handles frames and audio natively.
- Use `openssl base64 -in`, not `base64 -i` — macOS `base64` adds line breaks that break the data URI.

## Bitrix24 skill

`skills/bitrix24/SKILL.md` sends a text message to a Bitrix24 chat/channel via the `im.message.add` REST API, using an outbound webhook and `python3`+`urllib` (no `curl`).

### Prerequisites

- `python3` only — no ffmpeg or openssl needed.

### Webhook (secret)

The skill needs an outbound webhook URL and resolves it at runtime in this order:

1. `BITRIX24_WEBHOOK` environment variable:

   ```bash
   export BITRIX24_WEBHOOK=https://bitrix24.example.com/rest/<id>/<token>/
   ```

2. The `webhook` value in `~/.config/kilo/bitrix24.json`:

   ```json
   {"webhook": "https://bitrix24.example.com/rest/<id>/<token>/"}
   ```

3. Otherwise it exits with `MISSING_BITRIX24_WEBHOOK`.

The webhook URL embeds both the integration user id and the token in its path, so it is a secret: never commit it to this repo or paste it into chat. The integration user id is part of the URL — no separate parameter is needed.

### Channels

`DIALOG_ID` is always passed by the agent. Example:

* `chat123456` — «Пример канала»

### Usage

Ask the agent, e.g.: "Отправь в канал «Пример канала» (chat123456): обновление вышло, смотри README." The agent loads the skill via the `skill` tool and runs the pipeline.

**Secret handling:** no webhook is committed in this repo. Use only the `BITRIX24_WEBHOOK` env var or the config file above.

## Ecosystem-analysis skill

`skills/ecosystem-analysis/SKILL.md` holds the shared rules used by both `/analyze-video` and `/analyze-docs`: mapping findings to the current app vs. sibling ecosystem repos (`rankup-app`, `widget-pro`, `orbitron-hub`, `rankup-landing`, `orbitron-landing`), categorizing into bugs / updates / feature suggestions, and writing the plan (`.kilo/plans/<epoch-ms>-<slug>.md`) and ecosystem file (`.kilo/plans/TEMP.md`). It is loaded via the `skill` tool, so the two analyzers share a single source of truth.

## Adding a new skill or command

- Skills need YAML frontmatter with `name` and `description`; the description is what the skill picker shows. Invoke skills via the `skill` tool, never by pasting their contents into prompts.
- Commands are Markdown files in `commands/` with a `description` frontmatter (the filename minus `.md` is the command name), mirrored to `~/.config/kilo/commands/`. Body supports `$1`–`$N`, `$ARGUMENTS`, `@file`, and `` !`cmd` `` templates.
