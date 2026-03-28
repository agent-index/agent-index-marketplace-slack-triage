---
name: slack-triage-config-setup
type: setup
version: 1.0.0
collection: slack-triage
description: Setup for the slack-triage-config skill — installs the interactive settings management interface.
target: slack-triage-config
target_type: skill
upgrade_compatible: true
---

## Setup Overview

This installs the Slack Triage Config skill, which provides interactive management of monitored channels, VIP senders, ignore lists, response thresholds, and urgency settings. This skill operates on the same `setup-responses.md` file used by the `slack-triage` task.

---

## Pre-Setup Checks

- `slack-triage` task is installed in the member's workspace → if not: "Install the slack-triage task first — this skill manages its configuration."

---

## Parameters

No additional member-configurable parameters. This skill reads and writes to the slack-triage task's `setup-responses.md`.

---

## Setup Completion

1. Write the skill instance to the member's skills directory
2. Write `manifest.json`
3. Write empty `setup-responses.md` (stub — the skill operates on the slack-triage task's config)
4. Register entry in `member-index.json` with alias `@ai:slack-triage-config`
5. Confirm to member: "Slack Triage Config is installed. Say '@ai:slack-triage-config' to manage your channels, VIP senders, and thresholds."

---

## Upgrade Behavior

### Preserved Responses
N/A — this skill has no parameters of its own.

### Reset on Upgrade
N/A.

### Requires Member Attention
None expected.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
