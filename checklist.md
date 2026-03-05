# OpenClaw Multi-Agent Setup Checklist

Use this checklist to stand up a complete, production-ready multi-agent OpenClaw setup.

## 1) Base install + services

- [ ] Install/update OpenClaw to latest stable
- [ ] Verify install: `openclaw status`
- [ ] Ensure gateway service is running:
  - [ ] `openclaw gateway status`
  - [ ] `openclaw gateway restart` (after config changes)
- [ ] Confirm channel connectivity (Telegram/Discord/etc.) in `openclaw status`

## 2) Define agents

In `~/.openclaw/openclaw.json`:

- [ ] Add all agents under `agents.list` (example: `main`, `ops`, `crm`, `email`)
- [ ] Set per-agent workspace and skills where needed
- [ ] Keep a clear role split (coordinator vs specialists)

Suggested role model:

- `main`: user-facing coordinator
- `ops`: infrastructure, health, security, updates
- `crm`: lead/contact and workflow automations
- `email`: mailbox operations (read/reply/send/organize) via Himalaya

## 3) Subagent allowlist (important)

Configure which agents each agent can spawn.

For `main`, add:

```json
{
  "id": "main",
  "subagents": {
    "allowAgents": ["main", "ops", "crm", "email"]
  }
}
```

Checklist:

- [ ] Add `subagents.allowAgents` under `main`
- [ ] Include `main` itself if self-spawn is desired
- [ ] Include specialist targets (`ops`, `crm`, `email`)
- [ ] Restart gateway: `openclaw gateway restart`
- [ ] Verify from chat with `agents_list`/“List my agents”

## 4) Tooling and safety policy

- [ ] Set appropriate global tool profile (`tools.profile`)
- [ ] Restrict dangerous node/device commands via `gateway.nodes.denyCommands`
- [ ] Keep `auth.mode: token` and protect token material
- [ ] Run validation: `openclaw config validate`
- [ ] Run security audit: `openclaw security audit --deep`

## 5) Channel routing + messaging

- [ ] Configure channel(s) under `channels.*` (Telegram already configured)
- [ ] Verify DM/group policies (`dmPolicy`, `groupPolicy`, allowlists)
- [ ] Test inbound and outbound message flow per channel
- [ ] Confirm reply tag behavior (`[[reply_to_current]]`) if used

## 6) Memory and context hygiene

- [ ] Keep `memory/YYYY-MM-DD.md` updated for daily continuity
- [ ] Curate long-term memory in `MEMORY.md` (main session only)
- [ ] Avoid sensitive leakage in shared/group contexts

## 7) Automation and reliability

- [ ] Add cron jobs for repetitive tasks (`openclaw cron add ...`)
- [ ] Use isolated sessions for background jobs where appropriate
- [ ] Add failure alerts/cooldowns for critical cron jobs
- [ ] Keep heartbeats lightweight and useful (no noisy loops)

## 8) Operational runbook

- [ ] Document recovery commands:
  - [ ] `openclaw status`
  - [ ] `openclaw logs --follow`
  - [ ] `openclaw gateway restart`
- [ ] Document “known-good” config snapshot strategy
- [ ] Keep backup of `openclaw.json` before large edits

## 9) Acceptance tests (done = ready)

- [ ] `main` can spawn `ops`
- [ ] `main` can spawn `crm`
- [ ] `main` can spawn `email`
- [ ] Subagent completion announcements return to requester
- [ ] Tool permissions behave as intended (no unexpected elevated actions)
- [ ] Security audit has no critical findings
- [ ] End-to-end channel test passes (send + receive)

## 10) Optional hardening (recommended)

- [ ] Restrict Control UI exposure (`gateway.bind`, `trustedProxies`, origins)
- [ ] Limit ACP allowlist separately (if using ACP runtime)
- [ ] Pin explicit defaults that changed across versions (e.g., Telegram streaming, ACP dispatch)
- [ ] Re-run `openclaw config validate` after every config change

## 11) Email agent setup (Gmail + Himalaya, no `pass`)

### A) Add `email` agent to config

- [ ] Add agent entry in `~/.openclaw/openclaw.json`:

```json
{
  "id": "email",
  "name": "email",
  "workspace": "/home/yakub/.openclaw/workspace/email",
  "agentDir": "/home/yakub/.openclaw/agents/email/agent",
  "model": "openai-codex/gpt-5.3-codex",
  "skills": ["himalaya", "session-logs", "model-usage"]
}
```

- [ ] Ensure `main.subagents.allowAgents` contains `email`
- [ ] Restart gateway: `openclaw gateway restart`
- [ ] Verify in chat: `List my agents` shows `email`

### B) Gmail prerequisites

- [ ] In Gmail, enable IMAP (Settings → Forwarding and POP/IMAP)
- [ ] In Google account, enable 2FA
- [ ] Generate Google App Password (16 chars)

### C) Himalaya config (direct password storage)

- [ ] Create `~/.config/himalaya/config.toml`
- [ ] Use app password in both IMAP and SMTP blocks
- [ ] Lock config permissions: `chmod 600 ~/.config/himalaya/config.toml`

Template:

```toml
downloads-dir = "~/Downloads"

[accounts.gmail]
email = "YOUR_GMAIL@gmail.com"
display-name = "YOUR NAME"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "YOUR_GMAIL@gmail.com"
backend.auth.type = "password"
backend.auth.password = "YOUR_16_CHAR_APP_PASSWORD"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "YOUR_GMAIL@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.password = "YOUR_16_CHAR_APP_PASSWORD"
```

### D) PATH troubleshooting (`himalaya: command not found`)

- [ ] Check binary location: `command -v himalaya`
- [ ] If installed under Linuxbrew, add to shell PATH:

```bash
echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

- [ ] Validate: `himalaya --version`

### E) Connectivity tests

- [ ] `himalaya folder list --account gmail`
- [ ] `himalaya envelope list --account gmail --page-size 10`
- [ ] Send test email to self via `himalaya template send --account gmail`

---

## Quick apply snippet (main allowlist)

Edit `~/.openclaw/openclaw.json` and ensure:

```json
"agents": {
  "list": [
    {
      "id": "main",
      "subagents": {
        "allowAgents": ["main", "ops", "crm", "email"]
      }
    }
  ]
}
```

Then:

```bash
openclaw gateway restart
```
