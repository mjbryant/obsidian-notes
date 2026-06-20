# bold-channel
**Description:** Configure the Slack app in api.slack.com: bot scopes, event subscriptions URL, signing secret
**Started:** 2026-06-20

## Context

This is external configuration work, not code. Do it at https://api.slack.com/apps before writing any server code — the signing secret and bot token produced here are required env vars for [[crisp-server]].

**Steps:**
1. Create a new app ("From scratch") in the target workspace
2. Under *OAuth & Permissions → Bot Token Scopes*, add:
   - `chat:write` — post replies back to the user
   - `files:read` — download image attachments
   - `im:history` — read DMs to the bot (if using DM mode)
   - `channels:history` — read messages in channels (if using a channel instead)
3. Under *Event Subscriptions*:
   - Enable events
   - Set the Request URL to the Cloudflare tunnel URL + `/slack/events` (get this from [[quiet-tunnel]] first)
   - Subscribe to bot events: `message.im` (DMs) or `message.channels`
4. Under *Basic Information*, copy the **Signing Secret**
5. Install the app to the workspace, copy the **Bot User OAuth Token**

**Decisions to make:**
- DM the bot directly, or post to a dedicated `#recipes` channel? DM is simpler (no channel membership needed). Channel gives a visible log of everything added.
- Should the bot support being added to multiple channels, or locked to one?

**Output of this task:**
- `SLACK_SIGNING_SECRET` — goes in server env
- `SLACK_BOT_TOKEN` — goes in server env
- The public Request URL (from [[quiet-tunnel]]) entered in the portal

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written

### 2026-06-20 — Session 2
- Task marked in-progress
- Resolved DM vs channel decision: **DM mode** (simpler, no channel membership, `message.im` event, `im:history` scope)
- Produced `setup-guide.md`: step-by-step checklist for the Slack portal covering scopes, events, signing secret, app manifest option, and credential storage
- Produced `event-payloads.md`: full JSON shapes for all four event types `crisp-server` will receive (URL text, image file, plain text, url_verification challenge)
- Key gotcha documented: `files:read` scope alone does not grant file download access — files must be fetched via `files.info` API or the `url_private_download` field using the bot token as Bearer auth
- Key gotcha documented: Slack sends `message_replied` subtypes and bot echoes; server must filter on `subtype` absence and `bot_id` absence
- Key gotcha documented: URL verification challenge must respond within 3 seconds; file downloads should be deferred async
- Scope `im:read` is needed (in addition to `im:history`) for the bot to enumerate DM channels — include it defensively
- `reactions:write` noted as optional but useful for acknowledging receipt with an emoji before async processing completes
