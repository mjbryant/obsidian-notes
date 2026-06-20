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
