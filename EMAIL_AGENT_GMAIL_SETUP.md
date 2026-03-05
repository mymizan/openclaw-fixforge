# OpenClaw Email Agent + Gmail Setup (Himalaya)

This guide documents how to add and use an `email` agent in OpenClaw for Gmail operations (read, send, reply, organize inbox).

---

## 1) Add the `email` agent to OpenClaw config

Edit:

- `~/.openclaw/openclaw.json`

Add this object under `agents.list`:

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

Create workspace folder (if missing):

```bash
mkdir -p /home/yakub/.openclaw/workspace/email
```

---

## 2) Allow `main` to delegate to `email`

In the `main` agent entry, ensure:

```json
"subagents": {
  "allowAgents": ["main", "ops", "crm", "email"]
}
```

Apply config changes:

```bash
openclaw gateway restart
```

Verify in chat (or tool call):

- `List my agents`

You should see: `main`, `ops`, `crm`, `email`.

---

## 3) Install/verify Himalaya CLI

Check:

```bash
himalaya --version
```

If shell says `himalaya: command not found` but installed via Linuxbrew, fix PATH:

```bash
echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH"' >> ~/.bashrc
source ~/.bashrc
himalaya --version
```

(You can also run directly via `/home/linuxbrew/.linuxbrew/bin/himalaya`.)

---

## 4) Prepare Gmail

1. Enable IMAP in Gmail:
   - Gmail → Settings → See all settings → Forwarding and POP/IMAP → Enable IMAP
2. Enable Google 2-Step Verification
3. Generate Google **App Password** (16 chars)

Use the App Password for both IMAP and SMTP auth.

---

## 5) Configure Himalaya (direct password storage, no `pass`)

Create:

- `~/.config/himalaya/config.toml`

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

Lock permissions:

```bash
chmod 600 ~/.config/himalaya/config.toml
```

---

## 6) Test email connectivity

List folders:

```bash
himalaya folder list --account gmail
```

List inbox messages:

```bash
himalaya envelope list --account gmail --page-size 10
```

Send test mail to yourself:

```bash
cat << 'EOF' | himalaya template send --account gmail
From: YOUR_GMAIL@gmail.com
To: YOUR_GMAIL@gmail.com
Subject: OpenClaw email agent test

If you got this, Himalaya + Gmail is working.
EOF
```

---

## 7) Security notes

- Direct password in config is convenient but less secure than secret managers.
- Keep strict file permissions (`chmod 600`).
- Prefer App Passwords over primary Google account password.
- Revoke and rotate App Password if leaked.

---

## 8) Operational usage examples

Once configured, the `email` agent can be delegated tasks such as:

- “List unread emails from last 24h”
- “Draft reply to this sender”
- “Move invoices to folder X”
- “Summarize my inbox by priority”

The agent can execute those via Himalaya CLI in its own workspace/session.
