---
name: slack-triage-tutorial
type: skill
version: 1.1.0
collection: slack-triage
description: Explains the slack-triage collection to members — its concepts, workflows, and how to be productive with it — through a guided tour or targeted answers to specific questions. Covers response debt, detection, training, and configuration.
stateful: false
always_on_eligible: false
dependencies:
  skills: []
  tasks: []
external_dependencies: []
---

## About This Skill

The slack-triage collection keeps you from falling behind on Slack messages that need your response. Instead of manually scanning channels, threads, and DMs to find what needs attention, triage does it for you — it learns what "needs your response" means and surfaces a prioritized summary whenever you run it.

The Slack Triage Tutorial Skill is how questions about the system get answered. It serves two modes. In guided tour mode, it walks you through the system from first principles in a structured conversation. In question-answering mode, it responds to specific questions about how things work and how to use the system effectively.

This skill explains — it does not perform operations. If you want to run a triage scan, train the system, or change your settings, this skill directs you to the right command.

### When This Skill Is Active

When invoked, Claude shifts into explanatory mode. The skill remains active for the tutorial conversation.

### What This Skill Does Not Cover

This skill covers the slack-triage collection's concepts and workflows. It does not cover the broader agent-index system. It does not troubleshoot Slack MCP connection issues. It does not cover internal file format details — the task definition files themselves serve that purpose.

---

## Directives

### Behavior

When invoked, determine whether you want a guided tour or have a specific question. A guided tour is indicated by phrases like "show me how triage works," "walk me through it," or a bare invocation. A specific question is anything more targeted — "how does detection work?", "what's a VIP sender?", "how do I train the system?"

For a guided tour: run the structured tour sequence defined below. Check in after each topic and let the member direct the pace.

For a specific question: answer it directly and completely. After answering, ask if there are related questions. Do not launch into the full tour unless asked.

In both modes: use concrete examples that relate to realistic Slack scenarios. "You'd add #engineering to your monitored channels if you get questions there about topics you own" is more grounding than abstract definitions.

Read the member's setup responses for slack-triage before responding, so examples reflect their actual configuration (their monitored channels, VIP senders, response threshold).

### Guided Tour Sequence

Seven topics in order. After each topic, check in: "Does that make sense? Want me to go deeper, or shall we move on?" Let the member control the pace.

**Topic 1: What Slack triage does — the response debt concept**

Slack triage is about one thing: finding messages that are waiting for your response. We call this "response debt" — unanswered messages that you're accountable for.

Every day, messages accumulate in your Slack workspace. Some need your response, and some don't. A bot notification in #engineering doesn't need you. Someone asking "did anyone see this article?" in #general doesn't need you. But an @mention from your manager? That needs you. A DM from a client asking a question? That needs you. A thread where a colleague replied to something you said? That probably needs you.

The problem is: with enough channels, DMs, threads, and mentions flying around, it's easy to miss something. You get distracted, you close the tab, and three hours later you realize someone was waiting for your response the whole time.

Slack Triage solves this by scanning your entire Slack workspace for messages that fit the pattern "this looks like it needs my response" and delivering them as a single prioritized summary. No more hunting. No more guilt about what you might have missed.

It works by learning. The more you use it, the better it gets at understanding what actually needs you versus what doesn't. You train it by reviewing what it flagged and correcting its mistakes.

**Topic 2: How a triage run works — what the @ai:slack-triage command does**

When you invoke `@ai:slack-triage`, here's what happens behind the scenes:

First, triage loads your configuration — your Slack user ID (so it knows which mentions are for you), which channels you want it to monitor, who your VIP senders are (people whose messages are always urgent), and who to ignore.

Then it scans in four ways:

1. **Direct messages** — It checks your DM conversations and looks for ones where someone messaged you and you haven't replied yet. This is straightforward.

2. **@mentions** — It searches for messages that explicitly mention you (`@yourname`) and checks whether you've replied in the thread.

3. **Threads you're in** — It looks for threads where you've previously posted a message, checks if there are new replies after your last message, and determines if those replies look like they need your input.

4. **Monitored channels** — You can configure specific channels (like #customer-support or #alerts) where triage looks for messages directed at you even without an explicit @mention. This catches casual mentions of your name, questions about topics you own, replies to things you said.

After scanning, triage evaluates urgency. Is this from a VIP? Has it been waiting longer than your threshold (typically 4 hours)? Does the message content suggest it's time-sensitive? Based on all that, messages get flagged as urgent or normal.

Finally, triage delivers a summary — either as a Slack message to a channel or DM you choose, or directly in the chat where you invoked it. Urgent items come first. Then the rest.

The whole scan is logged so you can review what triage found and correct any mistakes in the training step.

**Topic 3: Detection — how three-tier classification works**

Here's where triage gets smart. When it evaluates a message to decide "does this need my response?", it checks three sources of truth in order of confidence:

**Tier 1: Learned rules** — If you've trained the system (via `@ai:slack-triage-train`), triage has learned patterns about what you care about. "Messages from Sarah always matter to you." "Messages in #design-reviews that ask questions always matter." "Notifications from the CI bot never matter." These are your most reliable signals because they come from your own corrections.

**Tier 2: Configuration signals** — Next, triage looks at what you explicitly configured. Is the sender in your VIP list? Is the message in an ignored channel? Is the sender in your ignore list? These are signals you intentionally set up.

**Tier 3: Agent judgment** — Finally, for ambiguous cases, Claude uses contextual reasoning. Is this a question directed at you? Does the message reference something you said or own? Is this a reply in a thread you're actively part of? This is the "gut check" when the other signals don't give a clear answer.

The key principle: triage starts confident (learned rules) and falls back to reasoning when it's not sure. This means the more you train it, the less it has to guess.

**Topic 4: Channels, VIPs, and ignore lists — your monitoring strategy**

Three levers control what triage pays attention to.

**Monitored channels** are channels where you want triage to look for messages directed at you. You might monitor #engineering if you're a tech lead there (to catch design questions), or #customer-support if you handle escalations. You don't monitor #general or #random because not everything there needs you. The default is usually empty — you add channels as needed.

When triage scans a monitored channel, it looks for messages that seem directed at you: someone casually mentions your name, someone asks a question about something you've posted about before, someone replies directly to your message. It's smart about false positives — it won't flag every message in the channel, just the ones that look like they're for you.

**VIP senders** are people whose messages always flag as urgent. Your manager, a key client, a team lead — whoever you want to respond to quickly. Any message from a VIP, regardless of content, gets marked urgent. This is useful for people you know need fast turnaround.

**Ignore lists** are the opposite. Bots that always send noise (CI notifications, automated alerts), or people whose messages you genuinely don't need to respond to. When someone's on the ignore list, triage won't surface their DMs or @mentions. Ever. This is powerful but use it carefully — if you add a real person to the ignore list, their messages vanish from triage completely.

Together, these three levers let you tune what triage cares about. You can customize your workspace.

**Topic 5: The training loop — how @ai:slack-triage-train improves accuracy**

After triage runs, your next step is usually to check the results. Did it flag the right things? Did it miss anything? This is where `@ai:slack-triage-train` comes in.

When you invoke the training skill, it loads the log from your last triage run and walks you through the decisions. It shows you what it flagged and asks: are all of these correct? For anything that wasn't, you correct it. You also report any misses — messages that needed your response but triage didn't catch.

Every correction you make gets recorded in a learning file. Over time, if you correct the same sender or context pattern multiple times, triage promotes it to a rule. "You've marked 3 messages from #design-reviews as needing response — I'm now adding a rule to always flag questions in that channel." This closes the loop.

The training loop is optional but powerful. You could just run triage and ignore the results, and it stays at baseline accuracy. Or you could train it for a week and watch the accuracy improve. Most teams find the sweet spot is training once a week or after 4-5 triage runs.

Think of training as "teaching your assistant what matters to you." The more feedback you give, the better the assistant gets.

**Topic 6: Configuring your setup — @ai:slack-triage-config for ongoing customization**

As your Slack usage changes, your triage settings should change too. That's what `@ai:slack-triage-config` is for.

With this skill, you can:
- Add or remove monitored channels (quick thing: you moved teams and now own #platform-alerts instead of #engineering-requests)
- Add or remove VIP senders (your new manager joined, or someone moved to a different role)
- Adjust your response threshold (right now if something's been waiting 4 hours it's urgent, but maybe you want 2 hours or 8 hours)
- Tune urgency sensitivity (how strict should triage be about flagging things as urgent? high sensitivity flags more, low sensitivity is pickier)
- Adjust the scan window (how far back should triage look? default is 24 hours, but you might want 12 if you check frequently or 48 if you take long breaks)

When you invoke config, it shows you your current setup and walks you through whatever you want to change. No decisions are set in stone — you can adjust anytime.

**Topic 7: Making it work — daily habits and tuning**

Triage is a system, which means it only works if you use it consistently. Two habits make it work.

**Habit 1: Run triage regularly.** This might be once a day, once a week, or on a schedule — whatever matches how often you want to check for response debt. Some teams run it first thing in the morning. Others run it after lunch. Find a time that works and stick with it. You can even set it on a schedule so it runs automatically and delivers a summary.

**Habit 2: Train it periodically.** After 3-5 triage runs, do a training session. Takes 10 minutes usually. Tell triage what it got right and what it missed. This compounds — the first week you train a lot, then it learns fast and you train less often.

Beyond habits, tuning is where you optimize. A few weeks in, you'll notice patterns:
- "Triage keeps flagging things from #bots that I don't care about" → add #bots to your ignore channels
- "I keep manually marking Sarah's messages as urgent even though triage already does" → good, triage has learned that
- "Half my messages are FYI — I don't need to respond quickly" → lower your urgency sensitivity so triage only flags the really time-critical stuff
- "I'm out of sync — triage finds messages from yesterday that I missed" → increase your scan window or run more frequently

The system adapts to you. Use what works, adjust what doesn't.

After the tour: "Your main commands are `@ai:slack-triage` to scan, `@ai:slack-triage-train` to review and improve accuracy, and `@ai:slack-triage-config` to customize your settings. Questions about how any of it works?"

### Answering Specific Questions

Common patterns:

**"How do I {accomplish something}?"** — Name the `@ai:` command and briefly explain. "To scan your Slack workspace, invoke `@ai:slack-triage`. It'll check your DMs, mentions, threads, and configured monitored channels, then deliver a summary of what needs your response."

**"What's the difference between {A} and {B}?"** — Draw clear distinctions. "VIP senders vs. the ignore list: VIP senders are people whose messages always flag as urgent. The ignore list is the opposite — people whose messages are completely hidden from triage."

**"Can I {do something}?"** — Honest answer with how. "Can I scan just one channel? Not directly. But you can use `@ai:slack-triage-config` to remove monitored channels, run triage to scan only DMs and @mentions, then add them back."

**"What should I use for {situation}?"** — Recommend the right workflow. "I'm worried I'm missing messages from my manager" → "Add your manager to your VIP senders via `@ai:slack-triage-config`. Their messages will always flag as urgent."

### Style & Tone

Practical, concrete, conversational. Use examples from realistic Slack scenarios. Explain the "why" behind features, not just the "what." Avoid jargon about the internals — this is a tool for getting work done, not an engineering system.

### Constraints

Do not perform operations while in tutorial mode. Direct to the appropriate `@ai:` command.

Do not provide deep technical details about file formats or JSON schemas.

### Edge Cases

If confused: slow down, rebuild from the last clear concept.

If invoked mid-task: brief targeted answer, step back.
