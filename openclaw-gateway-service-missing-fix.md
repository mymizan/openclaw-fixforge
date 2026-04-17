# OpenClaw Gateway Error: `systemctl is-enabled unavailable`

**Update: This happened because I logged in as root and then used `su` to switch to the target user, which ran OpenClaw in an incomplete session. The fix was to start a proper login shell—either with `su - target_user` or by logging in directly as the target user.**

## Problem

Running:

```bash
openclaw gateway start
```

returned:

```text
Gateway service check failed:
Error: systemctl is-enabled unavailable:
Command failed: systemctl --user is-enabled openclaw-gateway.service
```

## Environment Findings

- `systemctl --user` was available and working.
- User systemd session was running.
- `loginctl show-user "$USER" | grep Linger` showed `Linger=yes`.
- Root cause: `openclaw-gateway.service` unit file was missing (not installed/generated).

## Root Cause

OpenClaw expected a user-level systemd unit named:

`openclaw-gateway.service`

but the unit did not exist, so `systemctl --user is-enabled ...` failed.

## Solution

### 1) (Optional but recommended) Reinstall OpenClaw

```bash
npm i -g openclaw@latest
hash -r
which openclaw
openclaw --version
```

### 2) Try automatic service setup

```bash
openclaw gateway start
```

### 3) If still missing, create unit manually

```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/openclaw-gateway.service <<'UNIT'
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/env openclaw gateway start
ExecStop=/usr/bin/env openclaw gateway stop
Restart=on-failure
RestartSec=3
Environment=PATH=%h/.nvm/versions/node/v24.14.0/bin:/usr/local/bin:/usr/bin:/bin

[Install]
WantedBy=default.target
UNIT
```

### 4) Reload + enable + start

```bash
systemctl --user daemon-reload
systemctl --user enable --now openclaw-gateway.service
systemctl --user status openclaw-gateway.service --no-pager
openclaw gateway status
```

## Verification Checklist

- `systemctl --user status openclaw-gateway.service` shows **active (running)**.
- `openclaw gateway status` reports gateway is running.
- Reboot test (optional): service should auto-start for user session (with linger enabled).

## Notes for Future AI Troubleshooting

When seeing `systemctl --user is-enabled ... unavailable`:
1. Verify user systemd bus exists.
2. Check linger status.
3. Confirm unit presence with:
   ```bash
   systemctl --user list-unit-files | grep openclaw-gateway
   ```
4. If missing, create/install unit and reload daemon.
