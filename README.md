# Slack Triage

Scan Slack for messages that need your response and deliver a prioritized summary. Tracks unanswered DMs, @mentions, thread replies, and questions directed at you across configured channels.

## What It Does

Slack Triage scans your Slack workspace on each run and identifies messages that are waiting for your response. It checks direct messages you haven't replied to, @mentions you haven't acknowledged, threads you're involved in that have new replies, and messages in monitored channels that are directed at you — even without an explicit tag. Everything is prioritized by urgency and delivered as a structured summary.

Over time, a built-in training loop lets you correct what the system flags (or misses) so it learns what "needs my response" actually means for you.

## Included Skills and Tasks

- **slack-triage** (task) — The core scan. Checks DMs, mentions, threads, and monitored channels for messages awaiting your response. Evaluates urgency and delivers a prioritized summary.
- **slack-triage-config** (skill) — Interactive configuration management. Add or remove monitored channels, manage VIP senders, adjust response-time thresholds, and configure ignore patterns.
- **slack-triage-train** (skill) — Review and correct triage decisions from recent runs. Corrections accumulate into sender rules and context patterns that improve future accuracy.

## Prerequisites

- A Slack MCP server connected to the agent's session with channel history and user lookup access
- The member's Slack user ID (collected during setup)

## Workflow

1. **Install** — Admin installs the collection and configures org-level defaults (delivery method, default monitored channels, response-time thresholds).
2. **Member setup** — Each member provides their Slack user ID, selects which channels to monitor, configures VIP senders, and sets personal thresholds.
3. **Run** — Member invokes `slack-triage` (manually or on a schedule). Workspace is scanned and a prioritized summary is delivered.
4. **Train** — Member invokes `slack-triage-train` to review recent triage decisions and correct mistakes. Corrections feed back into future runs.
5. **Tune** — Member uses `slack-triage-config` to adjust monitored channels, VIP lists, and thresholds as their Slack usage changes.

## Version History

See [CHANGELOG.md](CHANGELOG.md).
