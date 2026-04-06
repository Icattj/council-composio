# ---

> OpenClaw AI Agent Skill

---
name: Council Composio
description: Connect Council Room agents to 500+ external apps (Gmail, Slack, GitHub, Notion, Jira, Linear, Google Workspace, etc.) via Composio's auth layer. Handles OAuth properly so tokens are managed by Composio, not stored in agent context. Use when agents need to send emails, create tasks, post messages, or interact with external SaaS tools.
---

# Council Composio

Connects any agent to 500+ apps through Composio's managed auth.

## Setup (One-Time)

1. Sign up at https://platform.composio.dev
2. Copy your API key
3. Set it: `export COMPOSIO_API_KEY="your_key_here"`
4. Connect apps via OAuth: call `COMPOSIO_MANAGE_CONNECTIONS`

## Core Flow

Every tool call follows: Search → Connect → Execute

```bash
BASE="https://backend.composio.dev/api/v3"
KEY="$COMPOSIO_API_KEY"

# 1. Find the right tool
curl -X POST "$BASE/tools/execute/COMPOSIO_SEARCH_TOOLS" \
  -H "x-api-key: $KEY" -H "Content-Type: application/json" \
  -d '{"arguments":{"queries":[{"use_case":"send an email via gmail"}],"session":{"generate_id":true}}}'

# 2. Connect app if needed (returns OAuth URL)
curl -X POST "$BASE/tools/execute/COMPOSIO_MANAGE_CONNECTIONS" \
  -H "x-api-key: $KEY" -H "Content-Type: application/json" \
  -d '{"arguments":{"toolkits":["gmail"],"session_id":"SESSION_ID"}}'

# 3. Execute
curl -X POST "$BASE/tools/execute/COMPOSIO_MULTI_EXECUTE_TOOL" \
  -H "x-api-key: $KEY" -H "Content-Type: application/json" \
  -d '{"arguments":{"tools":[{"tool_slug":"GMAIL_SEND_EMAIL","arguments":{"to":"user@example.com","subject":"Hi","body":"Message"}}],"sync_response_to_workbench":false,"session_id":"SESSION_ID"}}'
```

## Useful Apps for DataFlow

| App | What agents can do |
|---|---|
| Gmail | Raguel sends follow-up emails to bookkeepers |
| Google Sheets | Azrael reads/writes financial tracking sheets |
| Notion | Sarael saves meeting notes to workspace |
| Slack | Team notifications when milestones hit |
| GitHub | Uriel creates issues, checks PRs |
| Google Calendar | Rafael schedules bookkeeper test sessions |
| LinkedIn | Raguel researches bookkeeper profiles |

## Security Note

OAuth tokens are stored and managed by Composio's servers — NOT in agent context or config files. Agents only hold the `COMPOSIO_API_KEY` and session IDs.

## AgentExec Integration

After API key is set, agents can use Composio via [EXEC:] tags:
```
[EXEC: curl -s -X POST https://backend.composio.dev/api/v3/tools/execute/GMAIL_SEND_EMAIL -H "x-api-key: $COMPOSIO_API_KEY" ...]
```

Add curl to AgentExec allowlist with composio base URL pattern.

## Rate Limits

- 429 = rate limited → retry with exponential backoff
- Each tool call = 1 API credit (check your plan)

## Installation

```bash
cp -r council-composio/ ~/.openclaw/workspace/skills/council-composio/
```

## License

MIT © [Sentra Technology](https://github.com/Icattj)
