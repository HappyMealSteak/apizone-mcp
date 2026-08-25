# APIzone — MCP server for API status

**Is that API down?** [APIzone](https://apizone.io) is independent, real-time
status monitoring for **200+ popular APIs** — Stripe, OpenAI, AWS, GitHub,
Twilio and more. It runs its **own** probes from independent infrastructure, so
status reflects what's actually observed, not what a vendor's status page
reports.

This repository is the public home of APIzone's **Model Context Protocol (MCP)
server**, so any AI agent — Claude, Cursor, or your own — can natively answer
*"is X down right now?"* mid-task.

🔗 **[apizone.io](https://apizone.io)** ·
[Status board](https://apizone.io) ·
[Incidents](https://apizone.io/incidents) ·
[API docs](https://apizone.io/api-docs) ·
[MCP guide](https://apizone.io/mcp)

Live examples (these badges update automatically):

[![Stripe API status](https://apizone.io/badge/stripe.svg)](https://apizone.io/status/stripe)
[![OpenAI API status](https://apizone.io/badge/openai.svg)](https://apizone.io/status/openai)
[![GitHub API status](https://apizone.io/badge/github.svg)](https://apizone.io/status/github)

## Connect the MCP server

The server is hosted and stateless — no install, no build, no API key:

```bash
claude mcp add --transport http apizone https://apizone.io/api/mcp
```

Or point any MCP client at the Streamable-HTTP endpoint directly:

```
https://apizone.io/api/mcp
```

The [`server.json`](./server.json) in this repo is the Model Context Protocol
registry manifest.

### Tools

All tools are **read-only** and require **no authentication**:

| Tool | What it answers |
| --- | --- |
| `list_apis` | Which APIs are monitored, and their current status (optionally by category) |
| `get_api_status` | Is a specific API up, degraded, or down right now? |
| `check_apis` | Status of several APIs at once — find which dependency is the culprit |
| `get_api_uptime` | Uptime % and latency (p50/p95) over 24h / 7d / 30d / 90d |
| `list_recent_incidents` | Recent outages and degradations, newest first |

Agent discovery is also published at
[`/llms.txt`](https://apizone.io/llms.txt).

## Embed a status widget

Drop a live status widget into any page:

```html
<div data-apizone="stripe,openai,aws-s3"></div>
<script src="https://apizone.io/widget.js" async></script>
```

Or a single badge: `https://apizone.io/badge/<slug>.svg` — generator at
[/badges](https://apizone.io/badges), multi-API widget at
[/widget](https://apizone.io/widget).

## Public JSON API

No key required; 60 req/min per IP:

- `GET /api/v1/status` — all services with current status + latency
- `GET /api/v1/status/<slug>` — one API: 24h checks, 90-day uptime, incidents
- `GET /api/v1/uptime/<slug>` — uptime % and latency over 24h / 7d / 30d / 90d

Spec at [`/openapi.json`](https://apizone.io/openapi.json); full reference at
[/api-docs](https://apizone.io/api-docs).

## How status is derived

- **Synthetic checks.** APIzone probes a public, unauthenticated endpoint of
  every monitored service. Any HTTP response under 500 counts as healthy (a
  `401` from `api.stripe.com` still proves the API is up); timeouts, network
  errors, and 5xx are failures.
- **Status.** *Down* = recent checks all failing. *Degraded* = some recent
  failures, or latency well above the service's own baseline.
- **False-positive protection.** Before opening an incident, the checker
  re-probes once; a transient blip that recovers is suppressed, so it never
  alerts subscribers.
- **Independent.** Status reflects APIzone's own probes and may differ from a
  vendor's official status page.

## About this repository

This is the public MCP + product home for APIzone. The monitoring service
itself is hosted at [apizone.io](https://apizone.io); the application source is
maintained privately. Issues and suggestions for the MCP server (missing APIs,
tool ideas) are welcome here.
