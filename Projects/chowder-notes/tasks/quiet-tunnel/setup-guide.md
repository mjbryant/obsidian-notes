# quiet-tunnel: Cloudflare Tunnel Setup Guide

**Goal:** Expose the local chowder webhook server (port 3000) via a stable public HTTPS URL so Slack can POST events to it.

---

## Decision: Domain vs. No-Domain

This is the first thing to decide before running any commands.

| Approach | Stability | Requires | Cost |
|---|---|---|---|
| **Named tunnel + custom domain** | Permanent, human-readable URL (e.g. `https://chowder.yourdomain.com`) | Domain added to Cloudflare (can be any registrar, $10-15/yr) | Free on Cloudflare Zero Trust |
| **Named tunnel + `cfargotunnel.com` UUID** | Stable as long as tunnel exists (UUID never changes) | Nothing extra | Free |
| **Quick tunnel (`--url` flag)** | Changes every restart | Nothing | Free |

**Recommendation:** Use a named tunnel. Even without a custom domain, a named tunnel produces a stable `<uuid>.cfargotunnel.com` URL that does not change between restarts. This is different from a quick tunnel (`cloudflared tunnel --url localhost:3000`), which gets a new random `trycloudflare.com` URL every time.

If you have a domain on Cloudflare you can add a readable CNAME on top, but it is not required for stability.

---

## Step 1: Install cloudflared

Cloudflare maintains its own Homebrew tap. Do not use the formula in the default Homebrew core — use the tap:

```bash
brew install cloudflare/cloudflare/cloudflared
```

Verify:

```bash
cloudflared --version
# cloudflared version <X.Y.Z> (built ...)
```

---

## Step 2: Authenticate with Cloudflare

```bash
cloudflared tunnel login
```

This opens your browser to the Cloudflare dashboard. Log in and select the zone (domain) you want to associate with this tunnel. If you have no domain, you can still authenticate — just select any zone you own or skip; cloudflared will still write a certificate to `~/.cloudflared/cert.pem` that authorizes tunnel creation.

> **No domain?** You can still run this step. The cert is used to create named tunnels regardless of whether you route a hostname. You just won't run the DNS routing step later.

After completion, cloudflared writes:
```
~/.cloudflared/cert.pem
```

---

## Step 3: Create the named tunnel

```bash
cloudflared tunnel create chowder-slack
```

Output will look like:

```
Tunnel credentials written to /Users/mbryant/.cloudflared/<tunnel-id>.json.
Created tunnel chowder-slack with id <tunnel-id>
```

Note the `<tunnel-id>` UUID — you need it in the config file. You can always retrieve it later:

```bash
cloudflared tunnel list
```

---

## Step 4: Write the config file

Create `~/.cloudflared/config.yml`. The ingress rules tell cloudflared which hostnames map to which local services.

### Option A: With a custom domain

```yaml
# ~/.cloudflared/config.yml
tunnel: chowder-slack
credentials-file: /Users/mbryant/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: chowder.<your-domain.com>
    service: http://localhost:3000
  # Catch-all rule required — cloudflared rejects config without it
  - service: http_status:404
```

### Option B: No custom domain (UUID URL only)

```yaml
# ~/.cloudflared/config.yml
tunnel: chowder-slack
credentials-file: /Users/mbryant/.cloudflared/<tunnel-id>.json

ingress:
  # No hostname filter — all traffic to the tunnel goes to localhost:3000
  - service: http://localhost:3000
```

With Option B, your public URL will be:
```
https://<tunnel-id>.cfargotunnel.com
```

That URL is stable as long as the tunnel exists. Enter `https://<tunnel-id>.cfargotunnel.com/slack/events` in the Slack portal.

---

## Step 5: Route DNS (only if using Option A)

Skip this step if you are using Option B.

```bash
cloudflared tunnel route dns chowder-slack chowder.<your-domain.com>
```

This creates a CNAME in your Cloudflare DNS zone pointing `chowder.<your-domain.com>` to `<tunnel-id>.cfargotunnel.com`. You can verify it in the Cloudflare dashboard under DNS.

---

## Step 6: Test the tunnel manually (before installing the service)

Run the tunnel in the foreground to confirm it works before committing to a daemon:

```bash
# Start your local server first (or use a placeholder like `python3 -m http.server 3000`)
cloudflared tunnel run chowder-slack
```

You should see output like:
```
INF Starting tunnel tunnelID=<tunnel-id>
INF Registered tunnel connection connIndex=0 ...
INF Connection ... registered connIndex=1 ...
```

Hit `https://<tunnel-id>.cfargotunnel.com` (or your custom hostname) from another device or `curl`. When confirmed, press `Ctrl+C` and proceed to install the service.

---

## Step 7: Install the launchd service

```bash
sudo cloudflared service install
```

This does three things:
1. Copies the cloudflared binary path into a plist
2. Writes the plist to `/Library/LaunchDaemons/com.cloudflare.cloudflared.plist`
3. Loads it with `launchctl load`

The service runs as root (required for LaunchDaemons). It reads `~/.cloudflared/config.yml` — specifically, it reads the config from the home directory of the user who ran `cloudflared tunnel login`, which is `/Users/mbryant/.cloudflared/config.yml`.

> **SIP / permissions note:** `sudo cloudflared service install` writes to `/Library/LaunchDaemons/`, which requires root. This works fine on macOS with SIP enabled — SIP only protects `/System/Library/`, not `/Library/`. No SIP exceptions needed.

---

## Step 8: Start and enable the service

After `service install`, the plist is loaded but you may need to start it explicitly on the first run:

```bash
sudo launchctl start com.cloudflare.cloudflared
```

To verify it started:

```bash
sudo launchctl list | grep cloudflared
# Should show: <PID>  0  com.cloudflare.cloudflared
# A PID means it's running. A "-" means it's not running.
```

The service auto-starts on boot — no further configuration needed.

---

## Step 9: Configure Slack

Take the public URL and append `/slack/events`:

```
https://<tunnel-id>.cfargotunnel.com/slack/events
# or
https://chowder.<your-domain.com>/slack/events
```

Enter this in the Slack app portal under **Event Subscriptions** → **Request URL**.

---

## Verification

See the `## Verification` section at the bottom of this document.

---

## Managing the service

```bash
# Stop the tunnel
sudo launchctl stop com.cloudflare.cloudflared

# Start the tunnel
sudo launchctl start com.cloudflare.cloudflared

# Uninstall the service (removes plist and unloads)
sudo cloudflared service uninstall

# View live tunnel logs
sudo tail -f /var/log/cloudflared.log
# (log path may vary; also check Console.app for com.cloudflare.cloudflared)

# List tunnel connections from the CLI
cloudflared tunnel info chowder-slack
```

---

## Updating cloudflared

```bash
brew upgrade cloudflare/cloudflare/cloudflared
sudo cloudflared service install   # reinstall plist after upgrade
sudo launchctl stop com.cloudflare.cloudflared
sudo launchctl start com.cloudflare.cloudflared
```

---

## Verification

### 1. Check launchd is running the process

```bash
sudo launchctl list | grep cloudflared
```

Expected output:
```
12345   0   com.cloudflare.cloudflared
```

- Column 1 is the PID. A real PID (not `-`) means the process is alive.
- Column 2 is the last exit code. `0` means the last exit was clean (or it has never exited).
- Column 3 is the label.

If column 1 shows `-` and column 2 shows a non-zero exit code, the tunnel is crashing — check the logs (see below).

### 2. Check tunnel connection status

```bash
cloudflared tunnel info chowder-slack
```

Expected output:
```
NAME          : chowder-slack
ID            : <tunnel-id>
CREATED       : 2026-06-20 ...
CONNECTIONS   :
  colo=... id=... ...
  colo=... id=... ...
```

Four connections (`CONNECTIONS` entries) is normal — cloudflared opens 4 connections to different Cloudflare PoPs for redundancy.

### 3. End-to-end HTTP test

```bash
curl -v https://<tunnel-id>.cfargotunnel.com/
```

If the local server is running on port 3000 and responding, you should get its response. If the local server is not yet running, you will get a `502 Bad Gateway` from cloudflared (tunnel is up, but the backend is down) — this is expected and correct. A `502` confirms the tunnel itself is working.

### 4. View logs

```bash
# Real-time:
sudo log stream --predicate 'subsystem == "com.cloudflare.cloudflared"' --level debug

# Or check the system log:
sudo log show --predicate 'subsystem == "com.cloudflare.cloudflared"' --last 1h
```

Healthy log lines look like:
```
INF Registered tunnel connection connIndex=0 connection=... location=...
INF Registered tunnel connection connIndex=1 ...
```

Unhealthy signs:
- `ERR` lines with `failed to sufficiently increase receive buffer size` — ignorable on macOS
- `ERR connection failed` repeatedly — check credentials file path in config.yml
- Process exits with code `1` — config file syntax error; run `cloudflared tunnel ingress validate` to check

### 5. Validate config file syntax

```bash
cloudflared tunnel ingress validate
```

Reads `~/.cloudflared/config.yml` and reports any ingress rule errors. Run this any time you edit the config.
