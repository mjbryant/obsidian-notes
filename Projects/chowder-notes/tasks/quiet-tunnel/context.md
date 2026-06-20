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

## Resolved Decisions

### Named tunnel without a custom domain is viable
A named tunnel (created with `cloudflared tunnel create`) produces a stable UUID-based URL at `<tunnel-id>.cfargotunnel.com`. This URL does NOT change between restarts — it is permanently bound to the tunnel's UUID. A custom domain adds a human-readable CNAME on top but is not required for stability. Recommended starting point: named tunnel with the UUID URL, add a custom domain later if desired.

This is distinct from a **quick tunnel** (`cloudflared tunnel --url localhost:3000`), which gets a new random `trycloudflare.com` URL on every run and is unsuitable for Slack's Event Subscriptions URL.

### Free Cloudflare account is sufficient
Named tunnels and launchd service install work on the free Cloudflare Zero Trust plan. No paid plan required. The only cost is an optional domain ($10-15/yr through any registrar — just needs to be added to Cloudflare DNS).

### Homebrew tap, not core
Install via `brew install cloudflare/cloudflare/cloudflared` (the official tap), not any formula in Homebrew core. The tap formula is maintained by Cloudflare and tracks current releases.

### `cloudflared service install` works correctly on macOS (including SIP)
The command writes to `/Library/LaunchDaemons/`, which is not protected by SIP (only `/System/Library/` is). No SIP exceptions, `csrutil disable`, or workarounds needed. Running with `sudo` is sufficient.

### Config file path for daemon
When running as a launchd daemon (as root), cloudflared needs an explicit `--config` flag pointing to the user's config file. The plist uses:
```
--config /Users/mbryant/.cloudflared/config.yml
```
Without this, the daemon would look in root's home (`/var/root/.cloudflared/`), which is wrong.

### Plist location
- `/Library/LaunchDaemons/` — runs at boot, as root, before any user login (correct for a server)
- `/Library/LaunchAgents/` — runs when a user logs in (wrong for this use case)

The tunnel must survive reboots and run without anyone logged in, so LaunchDaemon is correct.

### Verification: 502 = tunnel is working
If `curl https://<tunnel-id>.cfargotunnel.com` returns a 502 when the local server is not running, that is correct and expected — it means the tunnel itself is up and routing traffic, but the backend (port 3000) is down. A 502 is a pass for "is the tunnel working?"

## Deliverables

- `tasks/quiet-tunnel/setup-guide.md` — full step-by-step setup guide with exact commands, config YAML, and verification steps
- `tasks/quiet-tunnel/com.cloudflare.cloudflared.plist` — the launchd plist that `cloudflared service install` generates, with annotations

## Session Log

### 2026-06-20 — Session 1
- Session started
- Initial context written

### 2026-06-20 — Session 2
- Task marked in-progress
- Researched cloudflared setup (no-domain approach, launchd behavior, SIP, config path)
- Wrote `setup-guide.md` with complete step-by-step instructions, two config options (with/without domain), and full verification section
- Wrote `com.cloudflare.cloudflared.plist` with annotations explaining each key
- Resolved: named tunnel without a domain produces a stable UUID URL — custom domain optional
- Resolved: explicit `--config` flag needed in plist to point at user's home dir (not root's)
- Outstanding: user must decide domain vs. no-domain before running Step 5 (DNS routing)
