# Slack Triage — Roadmap

Current version: 1.1.0
Last updated: 2026-04-05

---

## Current State

v1.1 focuses narrowly on **response debt** — finding messages that are waiting for the member's reply and surfacing them with urgency context. The collection covers DMs, @mentions, thread follow-ups, and directed questions in monitored channels. It includes the standard training loop for iterative accuracy improvement and a guided tutorial for member onboarding.

### Known Limitations

- **Casual mention detection is imprecise.** Step 5 of the triage task (scanning monitored channels for directed questions without @tags) relies heavily on agent judgment. It will produce false positives, especially early on before training data accumulates. This is by design — false positives are correctable, false negatives are invisible.
- **No cross-collection awareness.** The triage task doesn't know whether a message has already been handled by another collection (e.g., a message in a project channel that the Projects collection's channel-monitor already surfaced). Future versions should integrate with the Projects collection to avoid duplicate surfacing.
- **Thread scanning is expensive.** Checking every thread the member has participated in can be slow in busy workspaces. May need pagination, caching, or a "recently active threads only" heuristic.
- **No reaction-based response detection.** The task checks whether the member has *posted a reply*, but many people acknowledge messages with emoji reactions. A reaction should optionally count as "responded."
- **Scan window is a blunt instrument.** A fixed hour window means messages right at the boundary may be missed on one run and stale on the next. A cursor-based "since last run" approach would be more robust.

### Known Bugs

None yet (initial release).

---

## Wishlist

### v1.1 — Quality of Life (shipped 2026-04-02)

- ~~**Tutorial skill.**~~ Done. `slack-triage-tutorial` provides interactive guided tour and targeted Q&A for member onboarding.

### v1.2 — Quality of Life (continued)

- **Reaction-as-acknowledgment.** Add a config option `count_reactions_as_response` (default: false). When enabled, if the member has reacted to a message with any emoji, treat it as acknowledged and don't flag it.
- **Since-last-run cursor.** Replace the fixed `scan_window_hours` with a saved cursor that tracks the last completed scan timestamp. Fall back to `scan_window_hours` on first run or if the cursor is missing.
- **Scheduled run support.** Document and test running `slack-triage` on a cron schedule (e.g., every morning at 9am, after lunch at 1pm). The task already works for this but the setup doesn't surface it.

### v1.3 — Cross-Collection Integration

- **Projects collection awareness.** If the Projects collection is installed, check whether a flagged message is in a project channel that already has channel-monitor active. If so, note it in the summary: "This channel is also monitored by your Projects collection." Avoid double-surfacing.
- **Email Triage parity.** Add a `slack-digest` task (similar to `email-digest`) that summarizes activity in specific channels as a structured briefing. This extends the collection from "what needs my response" to "what happened while I was away."

### v2.0 — Intelligent Response Assistance

- **Research and suggest responses.** When a flagged message is a question the agent can help answer, offer to research it: check email threads, project docs, other Slack conversations, and any other agent-index collection data to draft a suggested response. The member reviews and sends it themselves. This is the long-term vision — the triage task becomes the front door to a broader "response assistant" that leverages everything the member's agent knows.
- **Response queue.** Instead of a one-time summary, maintain a persistent queue of items awaiting response. Members can mark items as "done," "snoozed," or "delegated." The queue syncs across runs and provides a running view of response debt.
- **Priority learning beyond senders.** Learn urgency patterns from message content, channel context, time of day, and conversation velocity — not just sender rules. This requires a richer correction model.

---

## Design Notes

This collection deliberately starts narrow. The temptation with Slack automation is to try to organize everything — summarize channels, categorize messages, manage notifications. Those are different problems:

- **Channel summarization** → better served by a dedicated digest collection or the Projects collection's channel-monitor
- **Bot notification routing** → better served by domain-specific collections (budget bots → finance collection, outage bots → ops/monitoring collection)
- **Notification settings optimization** → interesting but a one-time task, not an ongoing triage workflow

The response-debt focus keeps the collection useful for everyone with Slack, regardless of what other collections they have installed. It's the universal Slack problem.
