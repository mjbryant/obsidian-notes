# bold-channel: Slack Event Payload Reference

JSON structures received by `crisp-server` at `POST /slack/events`. All payloads are JSON with `Content-Type: application/json`.

All real event payloads (non-challenge) have two outer wrapper fields before the inner `event` object:

```
token        — legacy verification token (ignore; use X-Slack-Signature instead)
team_id      — workspace ID (e.g. "T01234ABCDE")
api_app_id   — your app ID (e.g. "A01234ABCDE")
event        — the actual event object (see below)
type         — always "event_callback" for real events
event_id     — unique ID for this delivery (e.g. "Ev01234ABCDE")
event_time   — Unix timestamp of when the event occurred
```

---

## 1. URL Verification Challenge

Sent by Slack when you save or update the Request URL in the portal. Must be handled **before** any real events arrive.

**Trigger:** Slack POSTs this immediately when you enter/save the Request URL in Event Subscriptions.

**Payload:**

```json
{
  "token": "Jhj5dZrVaK7ZwHHjRyZWjbDl",
  "challenge": "3eZbrw1aBm2rZgRNFdxV2595E9zWhEN3",
  "type": "url_verification"
}
```

**Required response:**

```json
{
  "challenge": "3eZbrw1aBm2rZgRNFdxV2595E9zWhEN3"
}
```

HTTP status 200, `Content-Type: application/json`. The `challenge` value must be echoed back verbatim.

**Timing:** Must respond within **3 seconds**. No signature verification is needed for this payload type (the `type` field is `url_verification`, not `event_callback`, so you can branch on type before checking the signature — though verifying anyway is harmless).

**Server logic:**

```python
body = request.json()
if body.get("type") == "url_verification":
    return {"challenge": body["challenge"]}
```

---

## 2. DM with a URL in the text

The user sends a message like `https://www.seriouseats.com/pasta-e-fagioli-recipe` directly in the DM with the bot.

**Fields to inspect:**
- `event.type` == `"message"`
- `event.channel_type` == `"im"` — confirms this is a DM, not a channel or group
- `event.text` — the raw message text; extract the URL from here
- `event.subtype` — **absent** (no subtype means a standard user message)
- `event.bot_id` — **absent** (present on bot messages; absence confirms a human sent this)

**Full payload:**

```json
{
  "token": "XXYYZZ",
  "team_id": "T01234ABCDE",
  "api_app_id": "A01234ABCDE",
  "type": "event_callback",
  "event_id": "Ev01234ABCDE",
  "event_time": 1718841600,
  "event": {
    "type": "message",
    "channel": "D01234ABCDE",
    "channel_type": "im",
    "user": "U01234ABCDE",
    "text": "https://www.seriouseats.com/pasta-e-fagioli-recipe",
    "ts": "1718841600.000100",
    "event_ts": "1718841600.000100"
  }
}
```

**Input detection logic (server):**

```python
text = event.get("text", "").strip()
if re.match(r'https?://', text):
    input_type = "url"
    url = text
```

Note: `text` may contain surrounding whitespace or Slack's angle-bracket URL formatting (`<https://example.com>`). Strip `<` / `>` if present.

---

## 3. DM with an image file attached

The user sends a photo (e.g., a photo of a recipe card, a screenshot of a recipe) via the DM. This arrives as a `message` event with `subtype: "file_share"` and a `files` array.

**Fields to inspect:**
- `event.type` == `"message"`
- `event.subtype` == `"file_share"` — distinguishes file messages from text messages
- `event.channel_type` == `"im"`
- `event.files` — array of file objects (usually one element for a single upload)
- `event.files[0].mimetype` — e.g. `"image/jpeg"`, `"image/png"`, `"image/heic"`
- `event.files[0].url_private_download` — URL to download the file bytes (requires Bearer auth)
- `event.files[0].id` — file ID (can be used with `files.info` API for full metadata)
- `event.text` — may be empty `""` or contain a caption the user typed

**Full payload:**

```json
{
  "token": "XXYYZZ",
  "team_id": "T01234ABCDE",
  "api_app_id": "A01234ABCDE",
  "type": "event_callback",
  "event_id": "Ev01234ABCDE",
  "event_time": 1718841700,
  "event": {
    "type": "message",
    "subtype": "file_share",
    "channel": "D01234ABCDE",
    "channel_type": "im",
    "user": "U01234ABCDE",
    "text": "",
    "ts": "1718841700.000200",
    "event_ts": "1718841700.000200",
    "files": [
      {
        "id": "F01234ABCDE",
        "created": 1718841695,
        "name": "recipe-card.jpg",
        "title": "recipe-card",
        "mimetype": "image/jpeg",
        "filetype": "jpg",
        "pretty_type": "JPEG",
        "user": "U01234ABCDE",
        "size": 284302,
        "mode": "hosted",
        "is_public": false,
        "url_private": "https://files.slack.com/files-pri/T01234ABCDE-F01234ABCDE/recipe-card.jpg",
        "url_private_download": "https://files.slack.com/files-pri/T01234ABCDE-F01234ABCDE/download/recipe-card.jpg",
        "thumb_360": "https://files.slack.com/files-tmb/T01234ABCDE-F01234ABCDE/recipe-card_360.jpg",
        "permalink": "https://yourworkspace.slack.com/files/U01234ABCDE/F01234ABCDE/recipe-card.jpg",
        "permalink_public": null
      }
    ]
  }
}
```

**Input detection logic (server):**

```python
if event.get("subtype") == "file_share":
    files = event.get("files", [])
    if files and files[0].get("mimetype", "").startswith("image/"):
        input_type = "image"
        download_url = files[0]["url_private_download"]
        # Download with: requests.get(download_url, headers={"Authorization": f"Bearer {SLACK_BOT_TOKEN}"})
```

**File download gotcha:** `url_private_download` requires the bot token as a Bearer auth header. Without it, Slack returns a redirect to a login page. The `files:read` scope is what grants access.

**HEIC files:** iPhones upload HEIC by default. If you pass HEIC to the Claude Vision API, convert first (e.g., via `pillow-heif` or ImageMagick). Check `mimetype == "image/heic"` and convert before sending to Claude.

---

## 4. DM with plain text (recipe description)

The user types a description like `Pasta with butter and parmesan, serves 2, takes 15 minutes`. No URL, no file.

**Fields to inspect:**
- `event.type` == `"message"`
- `event.subtype` — **absent**
- `event.bot_id` — **absent**
- `event.text` — non-empty, no URL pattern, no file

**Full payload:**

```json
{
  "token": "XXYYZZ",
  "team_id": "T01234ABCDE",
  "api_app_id": "A01234ABCDE",
  "type": "event_callback",
  "event_id": "Ev01234ABCDE",
  "event_time": 1718841800,
  "event": {
    "type": "message",
    "channel": "D01234ABCDE",
    "channel_type": "im",
    "user": "U01234ABCDE",
    "text": "Pasta with butter and parmesan, serves 2, takes 15 minutes",
    "ts": "1718841800.000300",
    "event_ts": "1718841800.000300"
  }
}
```

**Input detection logic (server):**

```python
# Falls through URL and file_share checks above
input_type = "text"
description = event.get("text", "").strip()
```

---

## Event filtering — what to ignore

The server will receive events it should silently ignore (return 200, do nothing):

| Condition | Meaning | Action |
|---|---|---|
| `event.bot_id` is present | Message sent by a bot (including your own bot's replies) | Ignore |
| `event.subtype == "message_changed"` | A user edited a previous message | Ignore |
| `event.subtype == "message_deleted"` | A user deleted a message | Ignore |
| `event.subtype == "message_replied"` | A thread reply notification | Ignore |
| `event.type != "message"` | Some other event type (shouldn't happen given subscriptions, but be safe) | Ignore |
| `event.text` is empty and no `files` | Empty message | Ignore |

**Recommended filter:**

```python
event = body.get("event", {})

# Ignore bot messages (including our own replies)
if event.get("bot_id"):
    return Response(status=200)

# Ignore all subtypes except file_share
subtype = event.get("subtype")
if subtype and subtype != "file_share":
    return Response(status=200)

# Now branch on input type
```

---

## Request verification (X-Slack-Signature)

Every real event delivery includes two headers:

```
X-Slack-Signature: v0=<hex-digest>
X-Slack-Request-Timestamp: <unix-timestamp>
```

**Verification algorithm:**

```python
import hmac
import hashlib
import time

def verify_slack_signature(body_bytes: bytes, timestamp: str, signature: str, signing_secret: str) -> bool:
    # Reject requests older than 5 minutes (replay protection)
    if abs(time.time() - int(timestamp)) > 300:
        return False

    sig_basestring = f"v0:{timestamp}:{body_bytes.decode('utf-8')}"
    expected = "v0=" + hmac.new(
        signing_secret.encode(),
        sig_basestring.encode(),
        hashlib.sha256
    ).hexdigest()

    return hmac.compare_digest(expected, signature)
```

Call this before processing any event. If it returns False, return HTTP 403.

**Important:** Use the raw request body bytes (before JSON parsing) for the HMAC. Parsing and re-serializing changes whitespace and key order, breaking the signature.

---

## Summary: input type decision tree

```
POST /slack/events
│
├── body.type == "url_verification"
│   └── Return {challenge: body.challenge}
│
└── body.type == "event_callback"
    ├── Verify X-Slack-Signature (reject with 403 if invalid)
    ├── event.bot_id present → ignore (return 200)
    ├── event.subtype present and != "file_share" → ignore (return 200)
    │
    ├── event.subtype == "file_share" and event.files[0].mimetype starts with "image/"
    │   └── input_type = "image" → download file, send to Claude Vision
    │
    ├── event.text matches URL pattern (https?://)
    │   └── input_type = "url" → fetch page, extract recipe
    │
    └── event.text is non-empty (no URL)
        └── input_type = "text" → send description to Claude
```
