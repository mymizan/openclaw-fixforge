# Multi-user OpenClaw systemd template

Use this template to run one OpenClaw gateway service per Linux user on the same VPS.

## 1) Create systemd template unit

Create `/etc/systemd/system/openclaw@.service`:

```ini
[Unit]
Description=OpenClaw Gateway for user %i
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
RemainAfterExit=yes
User=%i
WorkingDirectory=/home/%i
ExecStart=/usr/bin/openclaw gateway start
ExecStop=/usr/bin/openclaw gateway stop
ExecReload=/usr/bin/openclaw gateway restart
TimeoutStartSec=60
TimeoutStopSec=60

[Install]
WantedBy=multi-user.target
```

## 2) Reload systemd and enable instances

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw@alice
sudo systemctl enable --now openclaw@bob
```

## 3) Manage instances

```bash
sudo systemctl status openclaw@alice
sudo systemctl restart openclaw@alice
sudo systemctl stop openclaw@alice
```

## Notes

- Adjust `WorkingDirectory` if the user home is not `/home/<user>`.
- Adjust the OpenClaw binary path if not at `/usr/bin/openclaw`.
- Each user should keep isolated config, workspace, and secrets.
- Ensure unique ports/webhooks/tokens per instance to avoid conflicts.
