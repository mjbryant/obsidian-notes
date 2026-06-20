# clean-boot
**Description:** Install launchd daemons for cloudflared and the chowder server so both start on boot without manual intervention
**Started:** 2026-06-20

## Context

Two processes need to survive reboots:
1. **cloudflared** — the Cloudflare tunnel (exposes port 3000 at `https://chowder.sadboiphotos.com`)
2. **chowder slack-ingest server** — the FastAPI/uvicorn server at `http://127.0.0.1:3000`

### Part 1: cloudflared

The `quiet-tunnel` setup guide includes `sudo cloudflared service install` — check whether this was already run:

```bash
sudo launchctl list | grep cloudflared
```

If a PID appears in column 1, it's already running as a daemon and this part is done. If not, run:

```bash
sudo cloudflared service install
sudo launchctl start com.cloudflare.cloudflared
```

### Part 2: chowder server

The plist at `/Users/mbryant/bed/chowder/slack_ingest/launchd.plist` was written by the crisp-server agent but needs two fixes before installing:

**Fix 1 — uvicorn path:** The plist uses `/usr/local/bin/uvicorn` (system path) but the project uses a venv. Change it to:
```
/Users/mbryant/bed/chowder/.venv/bin/uvicorn
```

**Fix 2 — env vars:** The plist has empty strings for `SLACK_SIGNING_SECRET`, `SLACK_BOT_TOKEN`, and `ANTHROPIC_API_KEY`. Fill these in from the `.env` file before installing. (The plist is in `.gitignore` territory once it has real secrets — do not commit it with values filled in.)

**Install steps:**
```bash
# 1. Fix the plist (see above), then:
mkdir -p ~/Library/Logs/chowder
cp /Users/mbryant/bed/chowder/slack_ingest/launchd.plist \
   ~/Library/LaunchAgents/com.chowder.slack-ingest.plist
launchctl load ~/Library/LaunchAgents/com.chowder.slack-ingest.plist
launchctl start com.chowder.slack-ingest
```

**Verify:**
```bash
launchctl list | grep chowder
# PID in column 1 = running

curl http://127.0.0.1:3000/slack/events
# Should get a response (even a 405 Method Not Allowed = server is up)

tail -f ~/Library/Logs/chowder/slack-ingest.stdout
```

**Note:** LaunchAgents (in `~/Library/LaunchAgents/`) run as the current user when they log in. LaunchDaemons (in `/Library/LaunchDaemons/`) run as root at boot. The server should be a LaunchAgent — it needs access to iCloud Drive files under the user's home directory, which root can't always reach.

## Session Log

### 2026-06-20 — Session 1
- Task created
- Initial context written
