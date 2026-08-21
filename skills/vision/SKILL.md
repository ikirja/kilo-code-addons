---
name: vision
description: Pipeline for reading images and video via RouterAI Gemini API when the main model lacks vision. Handles local filesystem media and email attachment media via python3+urllib direct API calls.
---

# Vision Pipeline Skill

This skill works around the main model's lack of vision support by dispatching images and video through the RouterAI Gemini API directly from the main agent.

The `vision-reader` subagent approach DOES NOT WORK — Kilo's Read tool in subagents cannot pipe media data into multimodal API requests. Instead, the main agent encodes media to base64 and calls RouterAI's Gemini endpoint via python3+urllib.

## Architecture

Main Agent (DeepSeek, no vision)
  -> finds media (local filesystem or email MCP)
  -> for images: encodes to base64 (openssl base64 -in file.png | tr -d '\n')
  -> for video: compresses with ffmpeg, then encodes to base64
  -> calls RouterAI Gemini API via python3+urllib with base64 in request body
  -> receives text description
  -> continues work

## Prerequisites

- **python3** and **openssl** (present on all mainstream distros).
- **ffmpeg** is required for video processing (image processing works without it). Install per platform:
  - Debian/Ubuntu: `sudo apt update && sudo apt install -y ffmpeg`
  - Fedora (ffmpeg is not in the default repos — enable RPM Fusion first):
    ```
    sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf install -y ffmpeg
    ```
  - macOS: `brew install ffmpeg`
- Without ffmpeg, video processing fails gracefully with a clear error message.

## API key

The RouterAI API key must be supplied, otherwise the pipeline exits with `MISSING_API_KEY`. The key is resolved at runtime in this order:

1. `ROUTERAI_API_KEY` environment variable, or
2. the `apiKey` value under `provider.routerai_ru` in `~/.config/kilo/kilo.jsonc` (Kilo's RouterAI provider config — the default on this machine), or
3. fails with `MISSING_API_KEY: set ROUTERAI_API_KEY (see README.md)`.

Never hardcode or commit the key in this repo. See README.md for details.

## Workflow: Describe images from local filesystem

1. **Find images**: Use `glob` with patterns like `**/*.{png,jpg,jpeg,gif,webp,bmp,tiff}` in the target directory.
2. **Filter**: Skip icons, favicons, tiny UI elements unless user specifically wants them. Focus on content images (screenshots, photos, scans, diagrams).
3. **Batch**: Up to 3 images per API call (each image ~1MB base64).
4. **For each batch**, run this bash command (replace IMAGE_PATH and adjust the prompt):

```
python3 -c "
import json, subprocess, re, os, sys, urllib.request

img_path = 'IMAGE_PATH_HERE'
img_b64 = subprocess.check_output(['openssl', 'base64', '-in', img_path]).decode().replace('\n', '')

api_key = os.environ.get('ROUTERAI_API_KEY') or ''
if not api_key:
    try:
        _cfg = open(os.path.expanduser('~/.config/kilo/kilo.jsonc')).read()
        _m = re.search(r'\"apiKey\"\s*:\s*\"([^\"]+)\"', _cfg)
        api_key = _m.group(1) if _m else ''
    except Exception:
        pass
if not api_key:
    print('MISSING_API_KEY: set ROUTERAI_API_KEY (see README.md)')
    sys.exit(1)

ext = os.path.splitext(img_path)[1].lower().lstrip('.')
mime = {'jpg': 'jpeg', 'jpeg': 'jpeg', 'png': 'png', 'gif': 'gif', 'webp': 'webp', 'bmp': 'bmp', 'tiff': 'tiff'}.get(ext, ext)

body = {
    'model': 'google/gemini-3.1-flash-lite',
    'messages': [{
        'role': 'user',
        'content': [
            {'type': 'text', 'text': 'Describe this image in detail. Focus on content, layout, UI elements, text, and any actionable information.'},
            {'type': 'image_url', 'image_url': {'url': f'data:image/{mime};base64,{img_b64}'}}
        ]
    }]
}

req = urllib.request.Request(
    'https://routerai.ru/api/v1/chat/completions',
    data=json.dumps(body).encode(),
    headers={'Authorization': 'Bearer ' + api_key, 'Content-Type': 'application/json'}
)
try:
    resp = json.load(urllib.request.urlopen(req))
    print(resp['choices'][0]['message']['content'])
except urllib.error.HTTPError as e:
    print(f'HTTP {e.code}: {e.read().decode()}')
"
```

IMPORTANT: Use `openssl base64 -in` NOT `base64 -i` (macOS base64 adds line breaks that break the data URI).

5. **Present**: Relay the text description to the user.

## Workflow: Describe video from local filesystem

Supported formats: `.mp4`, `.webm`, `.mov`, `.avi`, `.mkv`.

Video is compressed with ffmpeg (max 720p, 2-minute cap, H.264/AAC) before encoding to keep base64 payload under API limits. The Gemini model processes video frames + audio natively — no separate audio extraction needed.

1. **Find videos**: Use `glob` with patterns like `**/*.{mp4,webm,mov,avi,mkv}` in the target directory.
2. **Pick ONE video per API call** (video payloads are much larger than images).
3. **Run this bash command** (replace VIDEO_PATH and adjust the prompt):

```
python3 -c "
import json, subprocess, re, os, sys, shutil, tempfile, urllib.request

file_path = 'VIDEO_PATH_HERE'
ext = os.path.splitext(file_path)[1].lower()
video_exts = {'.mp4', '.webm', '.mov', '.avi', '.mkv'}

if ext not in video_exts:
    print(f'UNSUPPORTED: {ext} is not a supported video format')
    sys.exit(1)

if not shutil.which('ffmpeg'):
    print('MISSING_FFMPEG: install ffmpeg (Debian/Ubuntu: sudo apt install ffmpeg, Fedora: see README, macOS: brew install ffmpeg)')
    sys.exit(1)

api_key = os.environ.get('ROUTERAI_API_KEY') or ''
if not api_key:
    try:
        _cfg = open(os.path.expanduser('~/.config/kilo/kilo.jsonc')).read()
        _m = re.search(r'\"apiKey\"\s*:\s*\"([^\"]+)\"', _cfg)
        api_key = _m.group(1) if _m else ''
    except Exception:
        pass
if not api_key:
    print('MISSING_API_KEY: set ROUTERAI_API_KEY (see README.md)')
    sys.exit(1)

tmp_dir = tempfile.gettempdir()
tmp_video = os.path.join(tmp_dir, 'vision_video.mp4')

subprocess.run([
    'ffmpeg', '-y', '-i', file_path,
    '-vf', \"scale='min(1280,iw)':'min(720,ih)':force_original_aspect_ratio=decrease\",
    '-c:v', 'libx264', '-crf', '30', '-preset', 'ultrafast',
    '-c:a', 'aac', '-b:a', '64k', '-ac', '1',
    '-movflags', '+faststart',
    '-t', '120',
    tmp_video
], check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)

orig_size = os.path.getsize(file_path)
comp_size = os.path.getsize(tmp_video)
print(f'Compressed: {orig_size / 1024 / 1024:.1f}MB -> {comp_size / 1024 / 1024:.1f}MB')

vid_b64 = subprocess.check_output(['openssl', 'base64', '-in', tmp_video]).decode().replace('\n', '')
os.remove(tmp_video)

body = {
    'model': 'google/gemini-3.1-flash-lite',
    'messages': [{
        'role': 'user',
        'content': [
            {'type': 'text', 'text': 'Describe this video in detail. Describe what is happening, the visual content, any people, objects, actions, text on screen, and the audio/dialogue if present. Provide a thorough description of the entire video.'},
            {'type': 'image_url', 'image_url': {'url': f'data:video/mp4;base64,{vid_b64}'}}
        ]
    }]
}

req = urllib.request.Request(
    'https://routerai.ru/api/v1/chat/completions',
    data=json.dumps(body).encode(),
    headers={'Authorization': 'Bearer ' + api_key, 'Content-Type': 'application/json'}
)
try:
    resp = json.load(urllib.request.urlopen(req))
    print(resp['choices'][0]['message']['content'])
except urllib.error.HTTPError as e:
    print(f'HTTP {e.code}: {e.read().decode()}')
"
```

4. **Present**: Relay the text description to the user.

## Workflow: Describe images from email attachments

1. **Read email**: Use `easy-mail-mcp_get_message` on the target email.
2. **Identify image attachments**: Check attachment filenames for image extensions.
3. **Extract image data**: If base64 content is returned from MCP, save to a temp file:
   ```
   echo 'BASE64_CONTENT' | base64 -d > /tmp/kilo/attachment.png
   ```
4. **Run the same python3 pipeline** from the local filesystem workflow above.
5. **Clean up**: `rm /tmp/kilo/attachment.png`
6. **Present**: Relay descriptions to the user.

## Workflow: Describe a single image on demand

When the user says "describe this image" or "what's in this screenshot":

1. Confirm the file path.
2. Run the python3 pipeline for images above with that path.
3. Return the description.

## Workflow: Describe a single video on demand

When the user says "describe this video" or "what's in this recording":

1. Confirm the file path.
2. Run the python3 pipeline for video above with that path.
3. Return the description.

## Important constraints

- The API uses `google/gemini-3.1-flash-lite` through RouterAI at `routerai.ru`.
- **API key**: resolved from the `ROUTERAI_API_KEY` env var, falling back to the `apiKey` in `~/.config/kilo/kilo.jsonc`. Never hardcode it in this repo.
- **Images**: up to ~2MB (compressed PNG) work fine. Larger images: compress first. macOS: `sips -Z 1200 input.png --out temp.png`; Linux: `ffmpeg -y -i input.png -vf scale=1200:1200:force_original_aspect_ratio=decrease temp.png`.
- **Video**: compressed to max 720p H.264, capped at 2 minutes, mono AAC 64kbps audio. Typical compressed output is 5–15MB (base64 ~7–20MB). Videos longer than 2 minutes are truncated by `-t 120`.
- **Video requires ffmpeg**: Debian/Ubuntu `sudo apt install ffmpeg`, Fedora needs RPM Fusion (see Prerequisites), macOS `brew install ffmpeg`. The pipeline exits with a clear error if ffmpeg is missing (image workflows are unaffected).
- Temp files go to the system temp dir (`tempfile.gettempdir()` — e.g. `/tmp` on Linux, `/var/folders/...` on macOS).
- The `vision-reader` subagent in `.config/kilo/agents/vision-reader.md` is DEPRECATED — do not use it. It returns empty results.
- When the user asks about media (images or video), always load this skill first via `skill` tool with `name: "vision"`.
- Video is sent as `image_url` content type (the OpenAI compatibility format) with `data:video/mp4;base64,...` MIME type. Gemini recognizes the MIME type in the data URI and processes it as video natively, including audio.
