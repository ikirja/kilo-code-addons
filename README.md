# Kilo Code-Addons

Personal add-ons for the user's Kilo setup: **skills** (`skills/`) and **slash commands** (`commands/`, currently empty). Pure Markdown — no build, test, lint, or packaging step.

## Layout

- `skills/<name>/SKILL.md` — a skill. Kilo loads it only after it is mirrored to `~/.config/kilo/skills/<name>/SKILL.md`.
- `commands/` — slash commands; mirror to `~/.config/kilo/commands/`.

## Installing / syncing

Kilo does not read this repo directly. After editing a skill or command here, copy it to the global config (the copy is a separate file, not a symlink):

```bash
cp -r skills/vision ~/.config/kilo/skills/vision
```

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

## Adding a new skill or command

- Skills need YAML frontmatter with `name` and `description`; the description is what the skill picker shows. Invoke skills via the `skill` tool, never by pasting their contents into prompts.
- Commands are Markdown files in `commands/`, mirrored to `~/.config/kilo/commands/`.
