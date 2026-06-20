# quiet-tunnel
**Description:** Install cloudflared, create a named tunnel, register as a launchd daemon for persistent public URL
**Started:** 2026-06-20

## Context

Cloudflare Tunnel gives the local server a stable public HTTPS URL without port forwarding. Free tier is sufficient. The URL produced here is what gets entered into the Slack app's Event Subscriptions config ([[bold-channel]]).

**Prerequisites:**
- A Cloudflare account (free)
- A domain on Cloudflare, OR use the auto-assigned `*.trycloudflare.com` URL (no domain needed for personal use)

**Steps:**
1. Install: `brew install cloudflare/cloudflare/cloudflared`
2. Authenticate: `cloudflared tunnel login`
3. Create a named tunnel: `cloudflared tunnel create chowder-slack`
   - This writes a credentials JSON to `~/.cloudflared/<tunnel-id>.json`
4. Create a config file at `~/.cloudflared/config.yml`:
   ```yaml
   tunnel: chowder-slack
   credentials-file: /Users/mbryant/.cloudflared/<tunnel-id>.json
   ingress:
     - hostname: chowder.yourdomain.com   # or omit for trycloudflare URL
       service: http://localhost:3000
     - service: http_status:404
   ```
5. Route DNS (if using custom domain): `cloudflared tunnel route dns chowder-slack chowder.yourdomain.com`
6. Register as launchd daemon so it starts on boot:
   `sudo cloudflared service install`
   This writes a plist to `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`

**Output of this task:**
- A stable public URL (e.g. `https://chowder.yourdomain.com` or assigned subdomain)
- That URL + `/slack/events` → entered in Slack portal ([[bold-channel]])
- `cloudflared` running as a persistent launchd service

**Notes:**
- The free `trycloudflare.com` URL is randomly assigned and changes if the tunnel restarts. Use a named tunnel with a custom domain for stability, or accept the random URL if the domain isn't set up yet (can always update Slack later).
- Port 3000 assumed; adjust to match whatever [[crisp-server]] listens on.

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written
