---
name: slack-triage-train-setup
type: setup
version: 1.0.0
collection: slack-triage
description: Setup for the slack-triage-train skill — installs the triage review and correction learning interface.
target: slack-triage-train
target_type: skill
upgrade_compatible: true
---

## Setup Overview

This installs the Slack Triage Train skill, which provides the iterative learning loop for improving triage accuracy. After each triage run, members can invoke this skill to review which messages were flagged, correct mistakes, and build a persistent correction history that future runs use.

---

## Pre-Setup Checks

- `slack-triage` task is installed in the member's workspace → if not: "Install the slack-triage task first — this skill reviews its triage decisions."

---

## Parameters

### `review_mode_default` [member-defined]
Default review mode when the skill is invoked.
- Options: `by-source` (recommended — bulk review grouped by scan source), `by-message` (one-at-a-time sequential review)
- Default: `by-source`

### `stale_log_warning_days` [member-defined]
Number of days after which the run log is considered stale and a warning is shown.
- Default: 7

---

## Setup Completion

1. Write the skill instance to the member's skills directory
2. Write `manifest.json`
3. Write `setup-responses.md` with configured parameters
4. Ensure `triage-corrections.json` exists in the slack-triage task directory (created during slack-triage setup, but verify)
5. Register entry in `member-index.json` with alias `@ai:slack-triage-train`
6. Confirm to member: "Slack Triage Train is installed. After running slack-triage, say '@ai:slack-triage-train' to review and improve the results."

---

## Upgrade Behavior

### Preserved Responses
- `review_mode_default`
- `stale_log_warning_days`
- All data in `triage-corrections.json` is preserved (managed by the slack-triage task, not this setup)

### Reset on Upgrade
N/A.

### Requires Member Attention
None expected.

### Migration Notes
- v1.0 → future versions: migration notes will be added here as new versions are published.
