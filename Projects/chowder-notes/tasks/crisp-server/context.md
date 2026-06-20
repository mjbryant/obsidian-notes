# crisp-server
**Description:** Build the local webhook server: Slack signature verification, input type detection, async response pattern, launchd plist
**Started:** 2026-06-20

## Context

The core of the pipeline. A small HTTP server running permanently on the Mac Mini. Slack POSTs events here; the server must reply 200 within 3 seconds (Slack's timeout), then do the actual work async and post the result back as a follow-up message.

**Language/runtime decision:** Python or TypeScript/Node. Either works well. Python has good libraries (`slack_sdk`, `httpx`, `anthropic`). TypeScript on Node has `@slack/bolt` which handles signature verification and async patterns cleanly. To be decided before starting.

**Key behaviors:**

1. **Signature verification** — every incoming request must have its `X-Slack-Signature` header verified against `SLACK_SIGNING_SECRET`. Reject anything that doesn't match. This is a security requirement (anyone can POST to the public URL).

2. **URL verification challenge** — when first setting up the Slack app, Slack sends a `url_verification` event. The server must echo back the `challenge` field. Handle this before any other logic.

3. **Input type detection** — inspect the event payload:
   - Message contains a URL → URL mode
   - Message has `files` array with image/PDF → image mode
   - Plain text → description mode
   - Order matters: a message can have both a URL and text; URL takes priority.

4. **Async pattern** — respond `HTTP 200 {"ok": true}` immediately, then process in a background thread/task. When done (or on error), call `chat.postMessage` to reply in the same thread.

5. **Environment variables required:**
   - `SLACK_SIGNING_SECRET`
   - `SLACK_BOT_TOKEN`
   - `ANTHROPIC_API_KEY`
   - `OBSIDIAN_RECIPES_PATH` — absolute path to the recipes folder

**launchd plist** — after the server is built, register it as a launchd user agent so it starts on login and restarts on crash:
- Plist location: `~/Library/LaunchAgents/com.chowder.slack-ingest.plist`
- `KeepAlive: true`
- `RunAtLoad: true`
- Stdout/stderr → log files in `~/Library/Logs/chowder/`

**Depends on:** [[bold-channel]] (for env vars), [[quiet-tunnel]] (for public URL)
**Calls into:** [[amber-extract]], [[gentle-writer]]

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written

### 2026-06-20 — Session 2
- Wrote `slack_ingest/` package at `/Users/mbryant/bed/chowder/slack_ingest/`
- `server.py`: FastAPI app with `POST /slack/events` — signature verification, url_verification challenge, bot/subtype filtering, input type detection (image/url/text), immediate 200 + BackgroundTasks for async processing, 👀 reaction for immediate user feedback, error reporting via postMessage
- `extractor.py`: stub raising NotImplementedError (amber-extract task)
- `writer.py`: stub raising NotImplementedError (gentle-writer task)
- `slack_ingest/launchd.plist`: informational plist for `~/Library/LaunchAgents/com.chowder.slack-ingest.plist`
- `requirements.txt` and `.env.example` added at project root
- Language/runtime decision: Python + FastAPI + uvicorn
