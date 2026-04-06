---
name: slack-triage-tutorial-setup
type: setup
version: 1.1.0
collection: slack-triage
description: Setup for the slack-triage-tutorial skill
target: slack-triage-tutorial
target_type: skill
upgrade_compatible: true
---

## Setup Overview

This installs the Slack Triage Tutorial skill. Say '@ai:slack-triage-tutorial' at any time to get a guided walkthrough of how the slack triage system works or to ask specific questions about configuring and running triage.

---

## Pre-Setup Checks

None.

---

## Parameters

No member-configurable parameters.

---

## Setup Completion

1. Write the installed instance to `/members/{member_hash}/skills/slack-triage-tutorial/`
2. Write `manifest.json`
3. Write empty `setup-responses.md`
4. Register entry in `member-index.json` with alias `@ai:slack-triage-tutorial`
5. Confirm to member: "Slack Triage Tutorial is installed. Say '@ai:slack-triage-tutorial' anytime to learn how the triage system works."

---

## Upgrade Behavior

### Preserved Responses
N/A.

### Reset on Upgrade
N/A.

### Requires Member Attention
None. The tutorial content updates automatically with the collection — no member action needed.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
