---
name: bitrix24
description: Send a text message to a Bitrix24 channel (chat) via the im.message.add REST API using an outbound webhook. Resolves the webhook from BITRIX24_WEBHOOK env var or ~/.config/kilo/bitrix24.json.
---

# Bitrix24 Channel Message Skill

This skill lets the main agent send a text message to a Bitrix24 chat/channel via the `im.message.add` REST API using an outbound webhook.

## Architecture

Main Agent (DeepSeek)
-> resolves the webhook at runtime (env var or local config, never hardcoded)
-> form-encodes DIALOG_ID + MESSAGE
-> POSTs to `<webhook>/im.message.add` via python3+urllib
-> receives the message id or an error
-> reports to the user

## Webhook resolution

The webhook URL is a secret — it embeds the integration user id and token in its path. Resolved at runtime in this order:

1. `BITRIX24_WEBHOOK` environment variable, or
2. the `webhook` value in `~/.config/kilo/bitrix24.json` (a local file outside this repo), or
3. fails with `MISSING_BITRIX24_WEBHOOK: set BITRIX24_WEBHOOK or ~/.config/kilo/bitrix24.json (see README.md)`.

Never hardcode or commit the webhook in this repo. See README.md for the config file format and the channel table.

## Workflow: send a message

1. **Determine the target channel**: use the channel table in README.md or the DIALOG_ID from the user's request. If ambiguous, ask the user.
2. **Build the message**: keep it short and plain. Multiline text and Cyrillic work.
3. **Write the message and the target channel id to temp files** (e.g. `/tmp/kilo-bitrix24-message.txt` and `/tmp/kilo-bitrix24-dialog-id.txt`, plain UTF-8). Neither value is ever interpolated into the shell command — shell metacharacters (`$`, backticks, quotes) in a message or channel id would otherwise be executed by the shell.
4. **Run this bash command**:

```
python3 -c "
import os, json, sys, urllib.request, urllib.parse

dialog_id = open(sys.argv[2], encoding='utf-8').read().strip()
message = open(sys.argv[1], encoding='utf-8').read().rstrip('\n')

webhook = os.environ.get('BITRIX24_WEBHOOK') or ''
if not webhook:
    try:
        webhook = json.load(open(os.path.expanduser('~/.config/kilo/bitrix24.json'))).get('webhook', '')
    except Exception:
        pass
if not webhook:
    print('MISSING_BITRIX24_WEBHOOK: set BITRIX24_WEBHOOK or ~/.config/kilo/bitrix24.json (see README.md)')
    sys.exit(1)

url = webhook.rstrip('/') + '/im.message.add'
data = urllib.parse.urlencode({'DIALOG_ID': dialog_id, 'MESSAGE': message}).encode()

try:
    resp = json.load(urllib.request.urlopen(urllib.request.Request(url, data=data)))
    if 'error' in resp:
        print(f'BITRIX24_ERROR: {resp[\"error\"]}: {resp.get(\"error_description\", \"\")}')
        sys.exit(1)
    print(f'SENT: message id {resp.get(\"result\")}')
except urllib.error.HTTPError as e:
    print(f'HTTP {e.code}: {e.read().decode()}')
" /tmp/kilo-bitrix24-message.txt /tmp/kilo-bitrix24-dialog-id.txt
```

5. **Report**: relay `SENT: message id <id>` to the user, or the error output verbatim.

## Response handling

- Success: `im.message.add` returns `{"result": <message_id>}` — the script prints `SENT: message id <id>`.
- API error: returns `{"error": "...", "error_description": "..."}` — the script prints `BITRIX24_ERROR: <error>: <description>` and exits 1.
- Transport error: the script prints `HTTP <code>: <body>`.

## Security constraints

- The webhook is a secret (contains the integration user id + token). Never echo it into chat, files, or README placeholders.
- Never inline the message or the channel id into the shell command — write them to temp files first. Values containing `$`, backticks, or `"` would otherwise be executed by the shell.
- Test sends post a real, visible message to the target channel — keep them short and few.
- Only one webhook is configured; override it per-call via `BITRIX24_WEBHOOK` if needed.
