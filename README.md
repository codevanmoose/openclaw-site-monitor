# OpenClaw Site Monitor

Monitor your websites. Get email alerts when they go down. Powered by [Truncus](https://truncus.co).

An [OpenClaw](https://openclaw.ai) agent template that watches your sites and emails you on downtime and recovery — no code, no webhook infrastructure, no cron scripts.

## How it Works

The agent checks your sites on a heartbeat (every 5 minutes by default) and tracks state between runs:

```
Heartbeat: checking 3 sites...
  truncus.co        200 OK    (142ms)
  vanmoose.cc       200 OK     (89ms)
  myapp.com         TIMEOUT   (10000ms)

State change detected: myapp.com UP -> DOWN
Sending alert to ops@example.com via Truncus...
  Alert sent: "[DOWN] myapp.com is unreachable"

2 up, 0 slow, 1 down. 1 alert sent.
```

When `myapp.com` recovers, you get a recovery email with the downtime duration. No repeated alerts while a site stays down.

## Quick Start

```bash
git clone https://github.com/codevanmoose/openclaw-site-monitor
cd openclaw-site-monitor
```

**1. Add your sites** — edit `config/sites.json`:

```json
{
  "sites": [
    {
      "url": "https://yoursite.com",
      "name": "My Site",
      "alert_email": "you@example.com"
    },
    {
      "url": "https://api.yoursite.com/health",
      "name": "My API",
      "alert_email": "ops@example.com"
    }
  ],
  "defaults": {
    "check_interval_minutes": 5,
    "timeout_seconds": 10,
    "expected_status": 200,
    "slow_threshold_ms": 5000
  }
}
```

**2. Add your Truncus API key** — copy `.env.example` to `.env`:

```bash
cp .env.example .env
# Edit .env and add your TRUNCUS_API_KEY
```

**3. Start the agent:**

```bash
openclaw
```

The agent reads `AGENTS.md` for instructions, `HEARTBEAT.md` for the monitoring schedule, and `config/sites.json` for your sites. It runs the checks every 5 minutes via the heartbeat system.

## Get a Truncus API Key

This agent sends email alerts via [Truncus](https://truncus.co), a transactional email API.

1. Sign up at [truncus.co](https://truncus.co) (GitHub or Google login)
2. Go to Dashboard > API Keys
3. Create a key with the `send` scope
4. Add it to your `.env` file

**Free tier: 3,000 emails/month. No credit card required.**

For a site monitor checking 5 sites every 5 minutes, you'd only send emails on actual incidents — the free tier handles thousands of alerts.

## Without Truncus

If `TRUNCUS_API_KEY` is not set, the agent still monitors your sites and logs results to `memory/` daily files. You'll see the output in your terminal — it just can't send email alerts. This lets you try the template before signing up.

## Configuration

### Site Options

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `url` | string | required | URL to check |
| `name` | string | required | Display name for alerts |
| `alert_email` | string | required | Where to send alerts |
| `expected_status` | number | `200` | Expected HTTP status code |
| `timeout_seconds` | number | `10` | Request timeout |
| `check_interval_minutes` | number | `5` | Check frequency |

### Heartbeat Interval

The default heartbeat is 5 minutes, configured in `openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "5m"
      }
    }
  }
}
```

Change `"5m"` to `"1m"`, `"10m"`, `"30m"`, etc.

### From Address

The agent sends from `noreply@mail.truncus.co` by default. To use your own verified domain, update the from address in `AGENTS.md` under the Alerting section.

## What You Get

- Uptime checks on a configurable schedule
- Email alerts on downtime via Truncus
- Recovery notifications with downtime duration
- Slow response warnings (> 5s threshold)
- State tracking between runs — only alerts on changes, no spam
- Daily logs in `memory/` for history

## Project Structure

```
openclaw-site-monitor/
  AGENTS.md              # Agent instructions (the brain)
  HEARTBEAT.md           # Periodic check schedule
  MEMORY.md              # Site state between runs
  openclaw.json          # Heartbeat interval config
  config/
    sites.json           # Your sites to monitor
  skills/
    truncus-email/
      SKILL.md           # Truncus email skill (pre-installed)
  examples/
    multi-site-config.json  # Example with 4 sites
  memory/                # Daily logs (auto-populated)
```

## Extending

**Add more alert channels.** The agent instructions in `AGENTS.md` are plain Markdown. Add a section for Slack webhooks, Telegram bots, or any other notification channel alongside the Truncus email alerts.

**Custom health checks.** Change `expected_status` to `301` for redirect monitoring. Point `url` at `/health` or `/api/status` endpoints for deeper checks.

**Multiple recipients.** Use the `cc` field in the Truncus skill to alert multiple people on the same incident.

## Built With

- [OpenClaw](https://openclaw.ai) — AI agent runtime
- [Truncus](https://truncus.co) — Transactional email API

## License

MIT
