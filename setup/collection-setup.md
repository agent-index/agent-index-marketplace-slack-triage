---
name: slack-triage-collection-setup
type: collection-setup
version: 1.0.0
collection: slack-triage
description: Org-admin setup for the slack-triage collection — configures delivery method, default monitored channels, and response thresholds for all members.
upgrade_compatible: true
---

## Collection Setup Overview

This setup is run by an org admin when installing the slack-triage collection. It configures org-wide defaults that flow into every member's setup as starting values. Members can customize their monitored channels and response thresholds, but the delivery method and max items per run are org-mandated.

---

## Prerequisites

- Slack MCP server available in the org's agent environment
- Admin has identified which Slack channels are broadly relevant for monitoring (team channels, project channels, announcement channels)

---

## Org-Level Parameters

### `delivery_method` [org-mandated]
How triage summaries are delivered to members.
- Options: `slack`, `chat`
- Ask: "How should triage summaries be delivered to your team? 'slack' sends a Slack message after each run. 'chat' outputs directly in the conversation."

### `max_items_per_run` [org-mandated]
Maximum number of items surfaced per triage run. Prevents information overload.
- Default: 20
- Ask: "What's the maximum number of items to surface per triage run? Default is 20."

### `default_response_threshold_hours` [org-mandated]
Default response threshold for all members. Members can override this during their setup.
- Default: 4
- Ask: "How many hours should a message wait before it's flagged as overdue? This is the org default — members can customize. Suggest 4 hours for responsive orgs, 8-12 for async-heavy teams."

### `default_monitored_channels` [org-mandated]
Starter set of channels every member monitors. Members can customize after setup via `slack-triage-config`. These should be channels where cross-team questions commonly occur.

Ask: "Which Slack channels should all members monitor by default? These are channels where questions and requests commonly go unanswered. Members can add or remove channels from their personal list later."

For each channel the admin provides, look up the channel ID via Slack MCP and confirm.

---

## Setup Completion

Collection setup is complete when:

1. `collection-setup-responses.md` is written to the collection's `/setup/` directory on the remote filesystem with all org-level parameters in YAML format
2. All parameter values are valid (delivery_method is `slack` or `chat`, thresholds are positive numbers, channel IDs resolve)
3. Admin has confirmed the configuration

The responses file uses this format:

```yaml
## Slack Triage Collection Configuration

delivery_method: chat
max_items_per_run: 20
default_response_threshold_hours: 4

default_monitored_channels:
  - id: "C01DEF456"
    name: "engineering"
  - id: "C02GHI789"
    name: "general"
```

---

## Upgrade Behavior

### Preserved Responses
- `delivery_method`
- `max_items_per_run`
- `default_response_threshold_hours`
- `default_monitored_channels`

### Reset on Upgrade
N/A.

### Requires Admin Attention
- If new org-mandated parameters are added in a future version, the admin will be prompted to configure them.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
