# Slack Triage — Changelog

All notable changes documented here.

Format: [MAJOR.MINOR.PATCH] — YYYY-MM-DD

---

## [1.0.0] — 2026-03-26

### Added
- Initial release of the slack-triage collection
- `slack-triage` task — scan DMs, mentions, threads, and monitored channels for messages awaiting response; prioritize and deliver summary
- `slack-triage-config` skill — interactive management of monitored channels, VIP senders, response-time thresholds, and ignore patterns
- `slack-triage-train` skill — post-run review and iterative correction learning for triage accuracy
- Three-tier configuration: org-mandated, role-suggested, and member-defined parameters
- Built-in response detection: tracks whether the member has replied in DMs, threads, and channel conversations
