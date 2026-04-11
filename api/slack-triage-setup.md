---
name: slack-triage-setup
type: setup
version: 1.1.0
collection: slack-triage
description: Setup for the slack-triage task — configures Slack identity, monitored channels, response thresholds, and delivery preferences.
target: slack-triage
target_type: task
upgrade_compatible: true
---

## Setup Overview

This setup configures the core Slack triage task. It collects the member's Slack identity, delivery preferences, and initial channel monitoring settings. The org admin may have pre-configured defaults for monitored channels, response thresholds, and delivery method — those are used as starting points but member-overridable settings can be customized.

---

## Pre-Setup Checks

- Slack MCP server is connected and responding (test with a channel list query) → if not: "Please connect a Slack MCP server before setting up Slack triage."
- If delivery_method is `slack`: validate the member can receive DMs or the delivery channel is accessible → if not: "The configured delivery channel isn't accessible. Switch to chat delivery or check channel permissions."

---

## Parameters

### `delivery_method` [org-mandated]
How the triage summary is delivered after each run.
- Options: `slack`, `chat`
- Default: inherited from collection setup
- If `slack`: the member must also provide a `delivery_channel_id`
- Ask: "How should your triage summary be delivered? 'slack' sends a message to a channel or DM after each run. 'chat' outputs directly in the conversation."

### `max_items_per_run` [org-mandated]
Maximum number of items to surface per triage run.
- Default: 20

### `urgency_sensitivity` [role-suggested]
How aggressively to mark items as urgent.
- Options: `high` (1+ criteria), `medium` (2+ criteria), `low` (all 3 criteria)
- Default: `high`
- Ask: "How sensitive should urgency flagging be? 'high' flags more items as urgent (less likely to miss something), 'medium' is balanced, 'low' only flags when multiple urgency signals overlap."

### `slack_user_id` [member-defined]
The member's Slack user ID (e.g., `U04ABC123`). Used to identify @mentions, DMs, and thread participation.
- Ask: "What's your Slack user ID? You can find it in your Slack profile → three dots → 'Copy member ID.'"

### `slack_display_name` [member-defined]
The member's Slack display name. Used for detecting casual mentions without an @tag.
- Ask: "What display name do you use in Slack? This helps me catch messages that mention you by name without an @tag."

### `delivery_channel_id` [member-defined]
Slack channel or DM ID where the triage summary is sent. Required if delivery_method is `slack`.
- Ask: "Where should I send your triage summary? This can be a DM to yourself or a private channel."
- Suggest: the member's own DM (slackbot channel) for privacy

### `monitored_channels` [member-overridable]
Slack channel IDs to scan for messages directed at the member beyond DMs and @mentions. Initialized from the org's `default_monitored_channels` at setup time.
- Present the org defaults and ask: "These are the default channels your org has configured for monitoring. Would you like to use them as-is, add more, or remove any?"
- For each additional channel, look up the channel ID via Slack MCP and confirm.

### `response_threshold_hours` [member-overridable]
Number of hours after which an unanswered message is considered overdue.
- Default: inherited from org's `default_response_threshold_hours`
- Ask: "How many hours should a message wait before I flag it as overdue? Your org default is {N} hours."
- Suggest ranges: 2-4 for responsive roles, 8-12 for batch processors, 24+ for low-urgency

### `scan_window_hours` [member-defined]
How far back to look for messages on each run.
- Default: 24
- Ask: "How far back should I scan for messages? Default is 24 hours. If you run triage multiple times a day, a shorter window (8-12 hours) may be better."

### `vip_senders` [member-defined]
Slack user IDs whose messages always flag as urgent.
- Default: empty array
- Ask: "Are there any people whose messages should always be flagged as urgent? (e.g., your manager, key stakeholders)"
- For each person named, look up their Slack user ID and confirm.

### `ignore_senders` [member-defined]
Slack user IDs or bot IDs whose messages are always skipped.
- Default: empty array
- Ask: "Are there any Slack bots or users whose messages you always want to skip? (e.g., CI bots, notification bots)"
- Warn: "Messages from ignored senders are completely skipped — including DMs and @mentions."

### `ignore_channels` [member-defined]
Channel IDs to never scan, even if the member is mentioned there.
- Default: empty array
- Ask: "Are there any channels where you want to ignore all mentions? (e.g., high-traffic channels where you're tagged by convention but don't need to respond)"

---

## Setup Completion

1. Write `setup-responses.md` to the member's task directory with all configured parameters in YAML format
2. Write `manifest.json` to the member's task directory
3. Create empty `triage-corrections.json` with the base schema
4. Create empty `triage-run-log.json` placeholder
5. Validate Slack MCP access by testing a channel list query
6. Register entry in `member-index.json` with alias `@ai:slack-triage`
7. Confirm to member: "Slack Triage is set up monitoring {N} channels with a {M}-hour response threshold. Say '@ai:slack-triage' to run it, or '@ai:slack-triage-config' to adjust your settings."

---

## Upgrade Behavior

### Preserved Responses
- `slack_user_id`
- `slack_display_name`
- `delivery_method`
- `delivery_channel_id`
- `monitored_channels` (including member customizations)
- `response_threshold_hours`
- `scan_window_hours`
- `vip_senders`
- `ignore_senders`
- `ignore_channels`
- `urgency_sensitivity`
- `triage-corrections.json` (all training data preserved)

### Reset on Upgrade
- `triage-run-log.json` (ephemeral, overwritten each run)

### Requires Member Attention
- If new org-mandated parameters are added, the member may be prompted to review them
- If the scan logic changes significantly, members may want to re-run training

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
