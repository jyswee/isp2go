# ISP2Go Integration

This project uses ISP2Go for ISP/WISP management — customers, billing, networking, tickets, and RICA/POPIA compliance — via the `isp` CLI. You (the agent) run it yourself. Don't ask a human to do things you can do.

## Setup (one-time)

```bash
npm install -g isp2go
isp login --local --key YOUR_API_KEY
```

Config lives in `.isp2go/config.json` and is auto-loaded. Add `.isp2go/` to your `.gitignore`.

No CLI (Claude Web, Raycast, other web agents)? Use the MCP server instead, same API key:

```json
{ "mcpServers": { "isp2go": { "type": "stdio", "command": "isp", "args": ["mcp-serve"] } } }
```

## Every session: load the schema FIRST

```bash
isp init --agent-schema
```

Returns the authoritative JSON of EVERY command and valid flag. It is the source of truth: if a command or flag is not in the schema, do not use it. Then run `isp status` for tenant state.

## Sandbox first, live later

ISP2Go has a Stripe-style test/live split. A sandbox is an isolated copy seeded with realistic demo data — safe to bill, ticket, and provision against without touching real customers.

```bash
isp signup my-isp --sandbox   # create tenant + seed a full demo ISP
isp mode sandbox              # route reads/writes to the sandbox DB
isp status                    # confirm you're in sandbox before any writes
isp mode live                 # flip to the clean live env when ready
```

Rule of thumb: rehearse every destructive or billing action in `sandbox` mode first. `isp mode` shows which you're in; `isp config` shows which config is active.

## What you can run

- **Customers** — `isp customers` (list/search), view services per customer.
- **Billing** — `isp billing-stats` (MRR/ARPU/overdue), `isp invoices`, `isp billing-run --preview` (dry-run a run before committing).
- **Tickets** — `isp ticket-create "subject" --customer <id> --priority high`, `isp tickets`.
- **Networking** — `isp routers`, `isp monitoring` (latency/uptime dashboard).
- **Leads** — `isp leads`, convert to customers.
- **Services / Tariffs** — `isp services`, `isp tariffs`.
- **Compliance** — `isp rica`, `isp popia`, `isp dsars` (RICA, POPIA/DSAR status).
- **Expert** — `isp ask "question"` queries the ISP2Go AI expert over WebSocket.

## Operating rhythm

- Load `isp init --agent-schema` at the start of every session — never guess a flag.
- Confirm mode with `isp status` before writes; default to `sandbox` for anything you haven't rehearsed.
- Preview billing with `isp billing-run --preview` before a live run.
- File tickets with a real `--customer <id>` (grab one from `isp customers --json`).
- Surface money problems, don't act blindly: agents never touch payment cards — a failed/overdue invoice is something you report, not force.

## Rules

- NEVER invent flags that are not in the `isp init --agent-schema` output.
- NEVER run a live `billing-run` you haven't previewed in sandbox first.
- Keep the API key in `.isp2go/config.json` (gitignored). Don't paste it into code or commits.
