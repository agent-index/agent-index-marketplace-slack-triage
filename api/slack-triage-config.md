---
name: slack-triage-config
type: skill
version: 1.1.0
collection: slack-triage
description: Interactively manage Slack triage settings — monitored channels, VIP senders, ignore lists, response thresholds, and urgency sensitivity.
stateful: true
always_on_eligible: false
dependencies:
  skills: []
  tasks: []
external_dependencies: []
---

## About This Skill

Slack Triage Config is the interactive management interface for all slack-triage settings. Members invoke this skill to add or remove monitored channels, manage their VIP sender and ignore lists, adjust response-time thresholds, and tune urgency sensitivity.

This skill reads and writes to the member's `setup-responses.md` file for the `slack-triage` task. Changes take effect on the next triage run.

---

## Directives

### Invocation

When the member invokes this skill, begin by reading their current `setup-responses.md` for `slack-triage`. Display a summary of their current configuration:
- Number of monitored channels with names
- Number of VIP senders
- Number of ignored senders/bots
- Current response threshold (hours)
- Current urgency sensitivity setting
- Current delivery method
- Current scan window

Then ask what they'd like to do.

### Supported Operations

**Add a monitored channel**
Ask for the channel name. Look up the channel ID via Slack MCP. Confirm with the member: "Monitor #{channel-name} for messages directed at you?" On confirmation, add the channel ID to `monitored_channels`.

**Remove a monitored channel**
Show the member their current monitored channels. Let them select one to remove. Confirm before removing. Note that DMs and @mentions in that channel will still be caught — only the "directed question" scan (Step 5 of the triage task) is affected.

**Manage VIP senders**
Show the current VIP list with display names. The member can add or remove Slack users by name or user ID. VIP senders always flag as urgent in triage runs.

**Manage ignore list**
Show the current ignore list with display names. The member can add or remove Slack users or bot IDs. Ignored senders are completely skipped during triage — their messages never appear in the summary, even @mentions or DMs. Warn the member of this before adding someone.

**Adjust response threshold**
Explain what the threshold does: "Messages waiting longer than this are flagged as overdue. Your current threshold is {N} hours." Let the member change it. Suggest reasonable ranges: 2-4 hours for responsive roles, 8-12 hours for roles that batch-process messages, 24+ hours for low-urgency monitoring.

**Adjust urgency sensitivity**
Explain the three levels:
- `high` — flag as urgent if any one urgency criterion is met (most items flagged urgent)
- `medium` — flag as urgent if two or more criteria are met (balanced)
- `low` — flag as urgent only if all three criteria are met (fewest urgent flags)
Let the member choose. Default is `high`.

**Adjust scan window**
Explain: "The scan window controls how far back the triage task looks for messages. Current window: {N} hours." Let the member change it. Warn that very large windows (48+ hours) will produce more results and take longer to scan.

**Import from training data**
If `triage-corrections.json` exists, offer to review learned `sender_rules` and promote them to permanent VIP or ignore list entries. This closes the loop between training and configuration.

### Writing Changes

After any modification, update the member's `setup-responses.md` with the new configuration. The configuration section uses this format:

```yaml
## Slack Triage Configuration

slack_user_id: "U04ABC123"
slack_display_name: "Bill"
delivery_method: chat
delivery_channel_id: null

monitored_channels:
  - id: "C01DEF456"
    name: "engineering"
  - id: "C02GHI789"
    name: "design-reviews"

vip_senders:
  - id: "U05JKL012"
    name: "Sarah Kim"
  - id: "U06MNO345"
    name: "Alex Chen"

ignore_senders:
  - id: "B07PQR678"
    name: "GitHub Bot"
  - id: "B08STU901"
    name: "CI Pipeline"

response_threshold_hours: 4
urgency_sensitivity: high
scan_window_hours: 24
max_items_per_run: 20
```

### Validation

Before saving any change:
- Channel IDs must be valid (resolvable via Slack MCP)
- User IDs must be valid Slack user or bot IDs
- `response_threshold_hours` must be a positive number
- `scan_window_hours` must be a positive number, minimum 1
- `max_items_per_run` must be a positive integer, minimum 1

### Guardrails

- Never modify the `slack_user_id` or `slack_display_name` — those are set during setup
- Never delete the member's `triage-corrections.json` or `triage-run-log.json`
- Always confirm destructive operations (clearing VIP list, removing all monitored channels) before executing
- Warn before adding a real person (not a bot) to the ignore list — their DMs and @mentions will be silently skipped
