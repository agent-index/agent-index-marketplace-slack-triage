---
name: slack-triage-train
type: skill
version: 1.1.0
collection: slack-triage
description: Review triage decisions from recent runs and record corrections that improve future accuracy.
stateful: true
always_on_eligible: false
dependencies:
  skills: []
  tasks:
    - slack-triage
external_dependencies: []
---

## About This Skill

Slack Triage Train is the iterative learning loop for the slack-triage system. After a triage run, members invoke this skill to review what was flagged, correct mistakes, and build a persistent correction history that future triage runs read to improve accuracy.

The skill reads `triage-run-log.json` (written by the `slack-triage` task after each run) and writes to `triage-corrections.json` (read by the `slack-triage` task at the start of each run). Over time, repeated corrections for the same sender or context get promoted to high-confidence rules that act as automatic quick-matches.

---

## Directives

### Invocation

When the member invokes this skill:

1. **Check for run log.** Read `triage-run-log.json` from the member's `slack-triage` task directory. If no log exists, inform the member: "No triage run log found. Run `slack-triage` first, then come back to review the results."

2. **Load existing corrections.** Read `triage-corrections.json` if it exists.

3. **Summarize the run.** Show a brief overview:
   - Run date and scan window
   - Total messages scanned (by source: DMs, mentions, threads, channel)
   - Number flagged
   - Number marked urgent
   - Number skipped (ignored senders)

4. **Ask the member how they want to review.** Offer two modes:
   - **By source** (recommended) — walk through each scan source (DMs, mentions, threads, channel questions) as a group
   - **By individual message** — walk through every decision one by one

### Review Flow: By Source

For each scan source that had flagged messages:

1. Show the source type and count: "I flagged {N} messages from **{source}**."
2. List each flagged message: sender, channel, message snippet, urgency status, and one-line reasoning.
3. Ask: "Are all of these correct, or did I flag anything that didn't actually need your response?"
4. If the member identifies false positives:
   - For each: ask "Should this sender/context always be ignored, or was it just this time?"
   - Record the correction.
5. If all correct, move to the next source.

After reviewing flagged messages, ask about misses:

1. "Were there any messages in the last {SCAN_WINDOW} hours that needed your response but I didn't flag?"
2. If yes: ask the member to describe the message (sender, channel, topic). Record as a missed-detection correction.
3. For missed messages: ask "Should messages like this always be flagged? What pattern should I look for?"

### Review Flow: By Individual Message

Walk through each decision in the run log sequentially:

1. Show: sender, channel, message snippet, scan source, urgency status, and reasoning.
2. Ask: "Correct?" with options:
   - **Yes** — move on
   - **Shouldn't have been flagged** — record as false positive
   - **Should have been urgent** / **Should not have been urgent** — record urgency correction
3. After every 10 messages, offer to stop: "Want to keep reviewing, or is this enough for now?"

### Recording Corrections

Each correction is appended to the `corrections` array in `triage-corrections.json`:

**False positive (flagged but shouldn't have been):**
```json
{
  "date": "{DATE}",
  "message_ts": "1711468800.123456",
  "channel_id": "C01DEF456",
  "sender_id": "B07PQR678",
  "sender_name": "GitHub Bot",
  "channel_name": "engineering",
  "message_snippet": "PR #342 merged to main",
  "scan_source": "channel_question",
  "correction_type": "false_positive",
  "should_ignore_sender": true,
  "member_note": "Bot messages from GitHub never need my response"
}
```

**Missed detection (should have been flagged but wasn't):**
```json
{
  "date": "{DATE}",
  "message_ts": "1711468800.789012",
  "channel_id": "C02GHI789",
  "sender_id": "U09VWX234",
  "sender_name": "Jordan Lee",
  "channel_name": "design-reviews",
  "message_snippet": "Can someone review the new nav mockups?",
  "scan_source": "channel_question",
  "correction_type": "missed_detection",
  "should_always_flag": false,
  "member_note": "Questions in design-reviews about mockups are usually for me"
}
```

**Urgency correction:**
```json
{
  "date": "{DATE}",
  "message_ts": "1711468800.345678",
  "channel_id": "D03ABC123",
  "sender_id": "U05JKL012",
  "sender_name": "Sarah Kim",
  "channel_name": "DM",
  "message_snippet": "Hey, quick question about the API spec",
  "scan_source": "dm",
  "correction_type": "urgency_override",
  "correct_urgency": true,
  "member_note": "Anything from Sarah is always urgent"
}
```

The `member_note` field is optional. If the member volunteers reasoning, capture it — it becomes valuable context for the agent on future runs.

### Promoting Sender Rules

After recording corrections, check whether any sender now has 3 or more corrections pointing to the same action. If so, promote to a `sender_rule`:

**Ignore rule:**
```json
{
  "sender_id": "B07PQR678",
  "sender_name": "GitHub Bot",
  "action": "always_ignore",
  "confidence": "high",
  "learned_from": 3,
  "last_updated": "{DATE}"
}
```

**Always-flag rule:**
```json
{
  "sender_id": "U09VWX234",
  "sender_name": "Jordan Lee",
  "action": "always_flag",
  "confidence": "high",
  "learned_from": 3,
  "last_updated": "{DATE}"
}
```

**Urgency rule:**
```json
{
  "sender_id": "U05JKL012",
  "sender_name": "Sarah Kim",
  "action": "always_urgent",
  "confidence": "high",
  "learned_from": 3,
  "last_updated": "{DATE}"
}
```

Inform the member: "I've noticed you've corrected messages from {sender} {N} times. I've added a rule so future messages from them will be automatically {action}."

### Promoting Context Rules

Check for patterns beyond senders. If the same channel + scan_source combination has 3+ corrections of the same type, promote to a `context_rule`:

```json
{
  "channel_id": "C02GHI789",
  "channel_name": "design-reviews",
  "pattern": "Questions about mockups or reviews are directed at this member",
  "action": "always_flag",
  "confidence": "medium",
  "learned_from": 3,
  "last_updated": "{DATE}"
}
```

Context rules have `medium` confidence because they're broader and more likely to produce false positives. The triage task uses them as a signal, not a hard rule.

### Suggesting Configuration Changes

At the end of a training session, if the skill has identified patterns that would be better served as permanent configuration, suggest them:

- "You've ignored messages from GitHub Bot 5 times now. Want me to add it to your ignore list in config?"
- "You always mark messages from Sarah Kim as urgent. Want me to add her to your VIP senders?"
- "You keep flagging questions in #design-reviews. Want me to add it to your monitored channels?"

These are suggestions only — the member must confirm. If confirmed, update `setup-responses.md` directly.

### The Corrections File

The full `triage-corrections.json` schema:

```json
{
  "version": "1.0.0",
  "last_trained": "{ISO_TIMESTAMP}",
  "stats": {
    "total_corrections": 0,
    "total_confirmations": 0,
    "sessions_completed": 0
  },
  "sender_rules": [],
  "context_rules": [],
  "corrections": []
}
```

- `sender_rules` — promoted sender-to-action mappings (read by slack-triage Steps 2-5)
- `context_rules` — promoted channel/context patterns (read by slack-triage Step 5)
- `corrections` — raw correction history (read by slack-triage as reference examples)
- `stats` — aggregate counts for the member's training activity

### Session Wrap-Up

At the end of a training session:

1. Summarize what was reviewed and corrected: "{N} corrections recorded, {M} rules promoted."
2. Update `stats` in the corrections file.
3. If any configuration suggestions were accepted, confirm the changes.
4. Remind the member: "These corrections will take effect on your next triage run."

### Guardrails

- Never modify `triage-run-log.json` — it is read-only input from the triage task.
- Never delete or truncate `corrections` history — only append.
- Never auto-apply configuration changes without member confirmation.
- If `triage-run-log.json` is stale (more than 7 days old), warn the member and suggest running a fresh triage first.
- Limit `corrections` array to the most recent 500 entries. If exceeded, archive older corrections to `triage-corrections-archive.json` and note the archive date in the main file.
