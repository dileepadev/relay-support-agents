---
name: 🤖 Agent Behavior
about: Report a wrong action, a misstated policy, a bad handoff or a wrong escalation
title: "🤖 Agent: Enter a short description of the problem"
labels: "type: eval"
assignees: "dileepadev"
---

## 🤖 Agent Behavior Report

<!-- If an agent moved money it shouldn't have, showed another customer's data or leaked a one-time code, follow SECURITY.md and don't open a public issue. Relay Support Agents runs on synthetic data: never include real personal information. -->

### 👤 Persona and Request

<!-- [REQUIRED] The demo persona you used (e.g., C-104) and what you asked for (e.g., a refund for a damaged lamp on HP-48213). -->

### 💬 What the Agents Did

<!-- [REQUIRED] What happened: the replies, any confirmation card or receipt, and the agent path if you saw it (e.g., router → support → escalation). Paste or summarize. -->

### ✅ Expected Behavior

<!-- [REQUIRED] What should have happened, and the policy rule that applies if you know it (R1–R12, see docs/design/01-problem-and-requirements.md §4.4). -->

### 🏷️ What Went Wrong

<!-- [REQUIRED] Tick the closest match (codes from the error taxonomy in design §12.8). -->

- [ ] Used the wrong tool or looked up the wrong thing (T1)
- [ ] Wrong order, item or amount in a request or a confirmation card (T2)
- [ ] Acted before I confirmed (T3)
- [ ] Gave up or didn't act when it should have (T4)
- [ ] Misstated a policy or promised an exception (T5)
- [ ] Didn't escalate to a human when it should have (T6)
- [ ] Escalated when it didn't need to (T7)
- [ ] Looped, repeated itself or ran out of budget (T8)
- [ ] Sent my message to the wrong agent (H1)
- [ ] An action request kept getting rejected (H2)
- [ ] Bounced between agents (H3)
- [ ] Didn't say it's an AI assistant, or didn't offer a human (R12)
- [ ] Not sure

### ✨ Details

<!-- [OPTIONAL] Help us find the trace. Remove if not needed. -->

<!-- example:
- 🔎 Conversation ID / Trace ID: [shown in the chat or the transcript download]
- 🕒 Date and time: [e.g. 2026-11-09 14:30 UTC]
- 🖥️ Where: [live demo / local / eval run]
- 🤝 Agent mode: [multi / single]
- 📦 Version: [e.g. v1.0.0 or commit SHA]
-->

### 📝 Additional Information

<!-- [OPTIONAL] Any additional context. Remove if not needed. -->

<!-- Maintainer note: confirmed reports become scenario or red-team candidates after manual labeling. -->
