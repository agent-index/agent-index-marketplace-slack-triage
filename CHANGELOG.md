# Slack Triage — Changelog

All notable changes documented here.

Format: [MAJOR.MINOR.PATCH] — YYYY-MM-DD

---

## [1.1.1] — 2026-04-19

### Added
- **Natural language trigger phrases in `collection.json`.** API entries now include trigger arrays that map conversational phrases to capabilities, powering the routing layer introduced in agent-index-core 3.0.5. Members can say things like "triage my Slack" or "what's important in Slack" instead of using `@ai:` alias syntax. Triggers are customizable per-member via `routing.json`.

## [1.1.0] — 2026-04-02

### Added
- `slack-triage-tutorial` skill — interactive guided tour and targeted Q&A for member onboarding covering collection concepts, workflows, response debt, detection, training, and configuration

## [1.0.0] — 2026-03-26

### Added
- Initial release of the slack-triage collection
- `slack-triage` task — scan DMs, mentions, threads, and monitored channels for messages awaiting response; prioritize and deliver summary
- `slack-triage-config` skill — interactive management of monitored channels, VIP senders, response-time thresholds, and ignore patterns
- `slack-triage-train` skill — post-run review and iterative correction learning for triage accuracy
- Three-tier configuration: org-mandated, role-suggested, and member-defined parameters
- Built-in response detection: tracks whether the member has replied in DMs, threads, and channel conversations
