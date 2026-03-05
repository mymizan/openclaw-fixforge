# Setup Guide: HubSpot MCP for OpenClaw `crm` Agent

## Goal
Connect an OpenClaw CRM agent to HubSpot via MCP so the agent can perform CRM operations (contacts, companies, deals, associations, notes/tasks).

## Prerequisites

- OpenClaw running on server
- `crm` agent already created (or create it first)
- Node/npm available
- `mcporter` available
- HubSpot account with permission to create Private Apps

---

## 1) Create HubSpot Private App + Token

In HubSpot:

1. Go to **Settings → Integrations → Private Apps**
2. Click **Create private app**
3. Name it (e.g., `openclaw-crm-agent`)
4. Grant required scopes (start minimal; read-only first if possible)
5. Create app and copy the token

> Start with minimum required scopes; expand later only if needed.

---

## 2) Persist token as environment variable

Use a shell profile (example: bash):

```bash
echo 'export HUBSPOT_PRIVATE_APP_ACCESS_TOKEN="PASTE_TOKEN_HERE"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
echo "$HUBSPOT_PRIVATE_APP_ACCESS_TOKEN" | wc -c
```

(Should return a value > 1)

---

## 3) Configure HubSpot MCP server in CRM workspace

```bash
cd /home/yakub/.openclaw/workspace/crm

mcporter config add hubspot \
  --command npx \
  --arg -y \
  --arg @hubspot/mcp-server \
  --env PRIVATE_APP_ACCESS_TOKEN=$HUBSPOT_PRIVATE_APP_ACCESS_TOKEN
```

This configures a stdio MCP server that runs HubSpot's official package via `npx`.

---

## 4) Validate configuration

```bash
cd /home/yakub/.openclaw/workspace/crm
mcporter config list
mcporter list hubspot --schema
```

If tools/schema appear, MCP wiring is successful.

---

## 5) Smoke test (safe)

```bash
cd /home/yakub/.openclaw/workspace/crm
mcporter call hubspot.hubspot-get-user-details
```

Expected: user/account/scope details returned from HubSpot.

---

## 6) Recommended first operational tests

1. Read-only contact search
2. Read-only company/deal lookup
3. Single-field update on a test contact (non-critical field)
4. Confirm changes in HubSpot UI

---

## Troubleshooting

### `mcporter list hubspot --schema` shows nothing or fails
- Confirm env var is set in current shell:
  ```bash
  echo "$HUBSPOT_PRIVATE_APP_ACCESS_TOKEN"
  ```
- Re-open shell / source profile
- Re-add config entry if needed:
  ```bash
  cd /home/yakub/.openclaw/workspace/crm
  mcporter config remove hubspot
  mcporter config add hubspot --command npx --arg -y --arg @hubspot/mcp-server --env PRIVATE_APP_ACCESS_TOKEN=$HUBSPOT_PRIVATE_APP_ACCESS_TOKEN
  ```

### Auth errors from HubSpot
- Token may be invalid/expired/revoked
- Missing required scopes for requested tool action
- Regenerate token and update env var

### `npx`/package execution issues
- Ensure Node/npm installed and working:
  ```bash
  node -v
  npm -v
  ```
- Run once manually to warm cache:
  ```bash
  npx -y @hubspot/mcp-server --help
  ```

---

## Security Notes

- Never paste token into chat logs or commit it to Git.
- Prefer env variables over hardcoding secrets in config files.
- Keep scopes minimal; use write scopes only when required.
- Rotate token if exposed.

---

## Quick Re-setup Checklist

```bash
# 1) Export token
export HUBSPOT_PRIVATE_APP_ACCESS_TOKEN='...'

# 2) Configure MCP
cd /home/yakub/.openclaw/workspace/crm
mcporter config add hubspot --command npx --arg -y --arg @hubspot/mcp-server --env PRIVATE_APP_ACCESS_TOKEN=$HUBSPOT_PRIVATE_APP_ACCESS_TOKEN

# 3) Verify
mcporter list hubspot --schema
mcporter call hubspot.hubspot-get-user-details
```
