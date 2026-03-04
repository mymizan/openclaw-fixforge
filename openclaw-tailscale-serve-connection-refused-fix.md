# OpenClaw Dashboard over Tailscale: `ERR_CONNECTION_REFUSED`

## Problem

Dashboard access from another tailnet device failed with:

- Browser error: `ERR_CONNECTION_REFUSED`
- URL used: `http://alex.tail758f1d.ts.net:18789/`

Ping to hostname worked, but HTTP still failed.

## Environment

- OpenClaw gateway config:
  - `gateway.bind: "loopback"`
  - `gateway.tailscale.mode: "serve"`
- Gateway running locally on `127.0.0.1:18789`
- Tailscale connected, tailnet hostname resolvable

## Root Cause

Two issues combined:

1. **Wrong access URL for Serve mode**
   - In Serve mode, use HTTPS on the MagicDNS hostname **without port**:
   - ✅ `https://alex.tail758f1d.ts.net/`
   - ❌ `http://alex.tail758f1d.ts.net:18789/`

2. **Serve not enabled in Tailscale admin**
   - `tailscale serve status` returned: `No serve config`
   - Attempting to configure serve prompted:
     `Serve is not enabled on your tailnet.`

## Verification Commands

```bash
openclaw config get gateway --json
openclaw gateway status
tailscale status
tailscale serve status
```

## Solution (Preferred: Tailscale Serve + HTTPS)

### 1) Enable Serve in Tailscale admin

Use the enable link shown by Tailscale when running `tailscale serve ...`, e.g.:

```text
https://login.tailscale.com/f/serve?node=<node-id>
```

### 2) Configure/ensure OpenClaw settings

```bash
openclaw config set gateway.bind loopback
openclaw config set gateway.tailscale.mode serve
openclaw gateway restart
```

### 3) Set Serve target (one-time or repair)

```bash
tailscale serve --bg 127.0.0.1:18789
tailscale serve status
```

### 4) Access dashboard from tailnet device

```text
https://alex.tail758f1d.ts.net/
```

## Alternative (No Serve admin enable required)

Bind gateway directly to tailnet IP:

```bash
openclaw config set gateway.bind tailnet
openclaw gateway restart
```

Access via:

```text
http://<tailscale-ip>:18789/
```

## Notes

- `ping` success only proves DNS/connectivity, not that an HTTP service is listening on that endpoint.
- In `bind: loopback` mode, `:18789` is local-only unless proxied (Serve/SSH tunnel/reverse proxy).
- In Serve mode, the expected user URL is HTTPS on MagicDNS hostname (typically port 443).

## Tags
- #openclaw #tailscale #serve #dashboard #connection-refused #networking
