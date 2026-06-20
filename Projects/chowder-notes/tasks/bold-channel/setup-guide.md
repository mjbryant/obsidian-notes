# bold-channel: Slack App Setup Guide

Step-by-step checklist for creating the chowder Slack app at https://api.slack.com/apps.

**Decision already made:** Use **DM mode**. The user DMs the bot directly. This requires only `im:history` (not `channels:history`), no channel membership, and the bot subscribes to `message.im` events only.

**Prerequisite:** Complete [[quiet-tunnel]] first so you have a stable public HTTPS URL to enter as the Request URL. The URL will be `https://<your-tunnel-hostname>/slack/events`.

---

## Step 1 — Create the app

1. Go to https://api.slack.com/apps
2. Click **Create New App**
3. Choose **From scratch**
4. App Name: `chowder` (or whatever you want users to see)
5. Pick your workspace (the one where you'll DM the bot)
6. Click **Create App**

You land on the app's **Basic Information** page. Keep this tab open — you'll return here for the Signing Secret.

---

## Step 2 — Add bot token scopes

Navigate to **OAuth & Permissions** in the left sidebar. Scroll to **Scopes → Bot Token Scopes**. Add each of the following:

| Scope | Why |
|---|---|
| `chat:write` | Post reply messages back to the user in DMs |
| `im:history` | Read messages in DM conversations with the bot |
| `im:read` | List and access DM channel metadata (defensive; needed by some SDK flows) |
| `files:read` | Access metadata for files shared with the bot |
| `reactions:write` | Add emoji reactions to messages (used to ACK receipt before async processing) |

**Do not add** `channels:history`, `channels:read`, or `groups:*` — those are for channel-based bots and are not needed here.

After adding all five scopes, the page should list them all under Bot Token Scopes.

---

## Step 3 — Enable Event Subscriptions

Navigate to **Event Subscriptions** in the left sidebar.

1. Toggle **Enable Events** to **On**
2. In the **Request URL** field, enter:
   ```
   https://<your-tunnel-hostname>/slack/events
   ```
   Slack will immediately send a `url_verification` challenge to this URL. Your server (`crisp-server`) must be running and must respond correctly for this field to save. See [[event-payloads]] for the exact challenge/response format.

   **Start the server before entering this URL:**
   ```bash
   cd /Users/mbryant/bed/chowder
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   cp .env.example .env        # then fill in SLACK_SIGNING_SECRET and SLACK_BOT_TOKEN
   uvicorn slack_ingest.server:app --host 127.0.0.1 --port 3000
   ```

3. Once the URL shows a green **Verified** checkmark, scroll down to **Subscribe to bot events**
4. Click **Add Bot User Event** and add:

| Event | Scope it requires |
|---|---|
| `message.im` | `im:history` |

That is the only event needed. Do **not** add `message.channels`, `file_created`, or others — `message.im` covers both text and file-share messages sent to the bot in DMs.

5. Click **Save Changes**

---

## Step 4 — Copy the Signing Secret

Navigate to **Basic Information** in the left sidebar.

Scroll to **App Credentials**. Find **Signing Secret** and click **Show**. Copy this value.

This is `SLACK_SIGNING_SECRET`. Store it now — you need it before `crisp-server` can verify any incoming webhook.

---

## Step 5 — Install the app to your workspace

Navigate to **OAuth & Permissions** in the left sidebar.

Click **Install to Workspace**. Slack will show you the list of permissions and ask you to authorize. Click **Allow**.

After install, you are returned to the OAuth & Permissions page. Under **OAuth Tokens for Your Workspace**, copy the **Bot User OAuth Token** (starts with `xoxb-`).

This is `SLACK_BOT_TOKEN`. Store it now.

---

## Step 6 — Find the bot in Slack and send a test DM

1. Open your Slack workspace
2. In the search bar, find the app by name (e.g., `chowder`)
3. Click its name to open a DM
4. Send a test message

If `crisp-server` is running, it should receive the event and respond.

---

## Credentials to save

Store both values in your server environment (e.g., a `.env` file at the `crisp-server` root, never committed):

```
SLACK_SIGNING_SECRET=<value from Step 4>
SLACK_BOT_TOKEN=xoxb-<value from Step 5>
```

These are the two env vars [[crisp-server]] requires at startup.

---

## Optional: App Manifest (shortcut)

If you want to recreate this app config in another workspace later, or want to avoid clicking through the UI, you can use an App Manifest. After completing the above steps, go to **App Manifest** in the left sidebar — Slack will show you the YAML/JSON representation of everything you configured. Save this to `tasks/bold-channel/app-manifest.yaml` for reference.

The relevant section of the manifest will look like:

```yaml
oauth_config:
  scopes:
    bot:
      - chat:write
      - im:history
      - im:read
      - files:read
      - reactions:write

settings:
  event_subscriptions:
    request_url: https://<your-tunnel-hostname>/slack/events
    bot_events:
      - message.im
  interactivity:
    is_enabled: false
  org_deploy_enabled: false
  socket_mode_enabled: false
```

---

## Gotchas

- **Request URL verification happens immediately.** Slack tries to verify the URL the moment you type it in. `crisp-server` must be running and reachable via the tunnel before you do Step 3. If the tunnel is not up, you cannot save the URL.

- **File downloads require Bearer auth.** The `files:read` scope lets the bot see file metadata, but to actually download the image bytes, you must GET the `url_private_download` field from the file object using your Bot Token as `Authorization: Bearer xoxb-...`. Without the token, the download returns a 302 redirect to a Slack login page.

- **The bot token is per-workspace.** If you ever reinstall or move workspaces, the `xoxb-` token changes. The signing secret stays the same unless you rotate it manually.

- **DM channel IDs start with `D`.** When the server needs to call `chat.postMessage` to reply, the `channel` parameter is the DM's `channel_id` from the event payload (e.g., `D01234ABC`). The bot does not need the user's ID to reply — just the channel.

- **Slack retries failed events.** If your server returns anything other than HTTP 200, Slack retries the event up to 3 times with exponential backoff. Always return 200 immediately and process async. See [[crisp-server]] for the async pattern.
