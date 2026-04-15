---
name: slack-triage
type: task
version: 1.1.0
collection: slack-triage
description: Scan Slack for messages awaiting the member's response — unanswered DMs, @mentions, thread replies, and channel questions — then prioritize and deliver a structured summary.
stateful: true
produces_artifacts: false
produces_shared_artifacts: false
dependencies:
  skills: []
  tasks: []
external_dependencies:
  - Slack MCP
reads_from: null
writes_to: null
---

## About This Task

Slack Triage is the core scan task. It identifies messages across the member's Slack workspace that are likely waiting for their response: unanswered direct messages, @mentions without follow-up, threads they're involved in with new replies, and messages in monitored channels that appear directed at them. Each message is evaluated for urgency and the results are delivered as a prioritized summary.

The task focuses narrowly on **response debt** — things that need the member's attention or reply. It does not attempt to summarize channel activity, categorize bot messages, or organize Slack broadly. Those are different problems for different collections.

Detection is driven by three inputs, checked in order of precedence:
1. **Learned rules** — sender and context rules from `slack-triage-train` corrections (highest confidence)
2. **Configuration signals** — VIP senders, monitored channels, ignore patterns from the member's config
3. **Agent judgment** — contextual reasoning for ambiguous messages (is this a question directed at me?)

---

## Configuration

This task reads its configuration from the member's `setup-responses.md` file at runtime. All parameters are set during setup or via the `slack-triage-config` skill.

### Required Parameters

- **`slack_user_id`** — The member's Slack user ID (e.g., `U04ABC123`). Used to identify @mentions, DMs, and thread participation.
- **`slack_display_name`** — The member's Slack display name. Used for detecting casual name-mentions without an @tag.
- **`delivery_method`** — `slack` or `chat`. How the triage summary is delivered.
- **`delivery_channel_id`** — Slack channel or DM ID for summary delivery (required if delivery_method is `slack`).
- **`monitored_channels`** — Array of Slack channel IDs to scan for messages directed at the member beyond DMs and @mentions.
- **`response_threshold_hours`** — Number of hours after which an unanswered message is considered overdue (default: 4).
- **`max_items_per_run`** — Maximum items to surface in a single triage summary (default: 20).

### Optional Parameters

- **`vip_senders`** — Slack user IDs whose messages always flag as high-urgency.
- **`ignore_senders`** — Slack user IDs or bot IDs whose messages are always skipped (e.g., automated bots, CI/CD notifications).
- **`ignore_channels`** — Channel IDs to never scan, even if the member is mentioned there.
- **`urgency_sensitivity`** — `high`, `medium`, or `low` (controls how many urgency criteria must be met).
- **`scan_window_hours`** — How far back to look for messages (default: 24). Messages older than this are not scanned.

---

## Workflow

### Step 1 — Load Configuration

Read the member's `setup-responses.md` to load all parameters: Slack user ID, display name, monitored channels, VIP senders, ignore lists, response threshold, and urgency sensitivity.

If `triage-corrections.json` exists in the member's task directory, load it. Extract `sender_rules` and `context_rules` for quick-match detection and keep `corrections` in context as reference examples for ambiguous cases.

### Step 2 — Scan Direct Messages

Retrieve the member's recent DM conversations from the past `scan_window_hours`.

For each DM conversation:
1. Read the most recent messages in the thread.
2. Check if the last message was sent by someone other than the member.
3. If the last message is from someone else AND the member has not replied after it: flag as **unanswered DM**.
4. Record: sender user ID, sender display name, channel ID, message timestamp, message snippet, and time since the message was sent.

Skip any DM where the sender matches `ignore_senders`.

### Step 3 — Scan @Mentions

Search for messages mentioning the member's user ID (`<@{slack_user_id}>`) within the `scan_window_hours` window, across all accessible channels.

For each @mention:
1. Read the thread (if the mention is in a thread) or the surrounding conversation.
2. Check if the member has posted a reply in the same thread after the mention timestamp.
3. If no reply exists: flag as **unanswered mention**.
4. Record: sender, channel name, channel ID, message timestamp, message snippet, thread context (if applicable), and time since the mention.

Skip mentions in channels listed in `ignore_channels`. Skip mentions from senders in `ignore_senders`.

### Step 4 — Scan Threads the Member Participates In

Search for threads where the member has previously posted a message. For each such thread:

1. Check if new replies have been posted after the member's last message in that thread.
2. If new replies exist and the most recent reply appears to be directed at the member (asks a question, references them, or continues a conversation they were actively part of): flag as **thread follow-up**.
3. If new replies exist but appear to be between other participants and don't require the member's input: skip.
4. Record: thread topic (first message subject/snippet), channel name, channel ID, last reply sender, last reply snippet, number of new replies since the member's last post.

Use agent judgment to determine whether the new replies need the member's response. Err on the side of flagging — a false positive (surfacing a thread that doesn't need response) is preferable to a false negative (missing one that does).

### Step 5 — Scan Monitored Channels for Directed Questions

For each channel in `monitored_channels`:

1. Retrieve messages from the past `scan_window_hours`.
2. Identify messages that appear directed at the member without an explicit @tag. Signals include:
   - The member's display name is mentioned casually in the message text
   - The message asks a question about a topic the member owns or has previously discussed in that channel
   - The message is a direct reply to something the member posted
3. Check whether the member has responded.
4. If no response and the message appears to need one: flag as **channel question**.
5. Record: sender, channel name, channel ID, message snippet, reason for flagging.

**This step requires the most agent judgment.** When in doubt about whether a message is directed at the member, check learned `context_rules` from `triage-corrections.json` for patterns in that channel. If still uncertain, flag it — the training skill will let the member correct false positives.

### Step 6 — Deduplicate

A single message might be caught by multiple scan steps (e.g., an @mention in a monitored channel could appear in both Step 3 and Step 5). Deduplicate by message timestamp + channel ID. Keep the entry from the most specific scan step (DM > mention > thread > channel question).

### Step 7 — Evaluate Urgency

For each flagged message, assess urgency. The number of criteria required depends on `urgency_sensitivity`:
- `high` sensitivity: one or more criteria met (default)
- `medium` sensitivity: two or more criteria met
- `low` sensitivity: all three criteria met

**A. Time Overdue**
The message has been waiting longer than `response_threshold_hours`. Messages more than 2x the threshold are marked critical.

**B. Important Sender**
The sender appears in `vip_senders`, or agent judgment infers importance from: the sender's role (manager, director, executive based on Slack profile/title), the sender being external (guest account), or a pattern of high-priority corrections for this sender in `triage-corrections.json`.

**C. Content Urgency**
The message content implies a timely response is expected: contains urgency keywords (urgent, ASAP, blocker, deadline, EOD, waiting on you, need your input), is a decision request, is a customer or client escalation, or references a time-sensitive event.

**Non-urgency signals (down-rank):**
- Messages in large channels with no direct address
- Messages that are purely informational (FYI, sharing a link)
- Messages from known low-priority bots not yet in the ignore list

### Step 8 — Write Run Log

Write a `triage-run-log.json` file to the member's task directory. This log is consumed by `slack-triage-train` for the review flow.

```json
{
  "run_date": "{ISO_TIMESTAMP}",
  "scan_window_hours": 24,
  "totals": {
    "scanned_dms": 0,
    "scanned_mentions": 0,
    "scanned_threads": 0,
    "scanned_channel_messages": 0,
    "flagged": 0,
    "urgent": 0,
    "skipped_ignored": 0
  },
  "decisions": [
    {
      "message_ts": "",
      "channel_id": "",
      "channel_name": "",
      "sender_id": "",
      "sender_name": "",
      "message_snippet": "",
      "scan_source": "dm|mention|thread|channel_question",
      "flagged": true,
      "urgent": false,
      "urgency_reasons": [],
      "time_waiting_hours": 0,
      "reasoning": "",
      "detection_source": "sender_rule|context_rule|config_signal|agent_judgment"
    }
  ]
}
```

### Step 9 — Compose and Deliver Summary

Compose the summary with urgent items first, then non-urgent items, grouped by source type.

**If delivery_method is `slack`:**
Send a single formatted Slack message to `delivery_channel_id`:

```
:speech_balloon: *Slack Triage — {DATE}*

Scanned the last {SCAN_WINDOW} hours. Found {TOTAL} message(s) that may need your response.

{If urgent items exist:}
:rotating_light: *{URGENT_COUNT} urgent:*

---

*1. {CHANNEL_NAME}* — {SENDER_NAME}
> {MESSAGE_SNIPPET}
:clock: Waiting {TIME} hours | {URGENCY_REASON}

---

{If non-urgent items exist:}
:inbox_tray: *{NON_URGENT_COUNT} other:*

---

*1. {CHANNEL_NAME}* — {SENDER_NAME}
> {MESSAGE_SNIPPET}
:clock: {TIME} hours ago

---
```

If no items found:
```
:speech_balloon: *Slack Triage — {DATE}*

:white_check_mark: No messages waiting for your response in the last {SCAN_WINDOW} hours.
```

**If delivery_method is `chat`:**
Output the same formatted summary directly in the conversation.

---

## Directives

- **Only read messages. Never send, edit, delete, or react to any message.**
- **Never mark any message as read** or change any Slack state.
- **Do not access private channels** the member is not a participant in.
- **Only surface messages that appear to need the member's response.** Do not surface FYI messages, bot notifications (unless from VIP senders), or messages clearly directed at someone else.
- **When in doubt, flag it.** A false positive is preferable to missing something that needs a response.
- **Always write the run log.** The training skill depends on it.
- **Respect the scan window.** Do not look at messages older than `scan_window_hours`.
- **Limit results to `max_items_per_run`.** If more items qualify, surface the most urgent based on time waiting and sender importance.

---

## Error Handling

- If Slack access fails, deliver an error message: "Slack Triage failed: could not access Slack. Please check your Slack MCP connector."
- If a specific channel is inaccessible (permissions, archived), note the error and continue scanning other channels.
- If Slack delivery fails and delivery_method is `slack`, fall back to outputting the summary in chat.
- If the workspace has 0 flagged messages, deliver the "no messages waiting" message.
- If `setup-responses.md` is missing or incomplete, halt and instruct the member to run setup.
