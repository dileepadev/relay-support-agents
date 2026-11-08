# Relay Support Agents design: Decisions, deliverables and agents

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 25. Architecture decision records

These are the initial decisions, and this table is their record. When a decision changes, or a new one is made (for example, a threshold change), add a short MADR-format file to `docs/adr/`, numbered from 015, using the [template](../adr/000-template.md).

| ADR | Decision | Alternatives | Rationale |
| --- | --- | --- | --- |
| 001 | Policy in a deterministic engine, called by tools | Policy in the prompt | Prompts can be overridden; code can't. Auditable, testable |
| 002 | Identity from signed token; no `customer_id` tool args | Model passes customer ID | Prevents confused-deputy and cross-account attacks |
| 003 | MCP for tools | In-process function calling | Standard protocol, reusable by any client, clean trust boundary, separately deployable |
| 004 | LangGraph supervisor graph with agent subgraphs + Postgres checkpointer | OpenAI Agents SDK, Claude Agent SDK, plain loop | Durable interrupts for approvals; explicit, testable handoffs; compared in E5 |
| 005 | Single-use approval tokens bound to parameters + re-check at execution | Boolean "approved" flag | Prevents approval reuse and time-of-check/time-of-use bugs |
| 006 | Hash-chained, INSERT-only audit log | Plain log table | Tamper evidence; demonstrable integrity |
| 007 | Simulated-user evals with end-state grading | Static transcript tests | Multi-turn behavior; objective pass/fail; pass^k reliability |
| 008 | ECS Fargate + Terraform on AWS | App Runner, Lambda, Cloud Run | Industry-standard container platform; reproducible IaC; SSE-friendly |
| 009 | No web, email or image output from the agent | Richer capabilities | Breaks the lethal trifecta for data exfiltration |
| 010 | Secure inputs (one-time codes) bypass the LLM via an input-required round trip | Customer types the code in the chat | Secrets never enter model context, logs or traces |
| 011 | Cacheable stable prefix + just-in-time policy retrieval + compaction | Full policies and full history in every call | Lower cost and latency; measured in E10 |
| 012 | Four agents with separated privileges (router, support, actions, escalation) | One agent with all tools; many fine-grained specialist agents | No single model both reads untrusted text and moves money; measured against the single agent in E11 |
| 013 | Typed handoff contracts through an in-process orchestrator | Free-text messages between agents; A2A over the network | Validated in code, auditable, no injection path into the Actions agent; A2A only if agents need separate services |
| 014 | Customer confirmation card and receipts rendered from templates | Agents write confirmations and receipts | Exact amounts; the money-moving agent never writes free text to customers |

---

## 26. Deliverables: README, demo video, write-up

### 26.1 README blueprint

1. **Title, one-liner, badges, GIF** of the approval flow.
2. **Live demo** (persona picker + console) and the video.
3. **Results:**
   - pass^1–pass^4 curve;
   - success, routing and escalation table;
   - **red-team table with the "stopping layer" column**;
   - cost per conversation by model;
   - **multi-agent vs single-agent table (E11)**.
4. **How it works:** trust-boundary diagram, agent graph and the per-agent permissions table.
5. **Security design:** principles, threat model summary, OWASP mappings (link to `docs/threat-model.md`).
6. **Evaluation methodology:** scenarios, simulator validation, judge κ, error taxonomy.
7. **Engineering decisions:** ADR links.
8. **Quickstart:** `make setup && make up && make seed && make eval-agent`.
9. **Deploy:** the Terraform summary and the cost table.
10. **Limitations** (synthetic data, simulated payments) and next steps.
11. **How I built this with coding agents** (build log).

### 26.2 Demo video (2.5 minutes)

| Time | Show |
| --- | --- |
| 0:00 | Problem: agents that act are risky; the store and its rules |
| 0:15 | $24 refund → confirmation card → auto-approved → receipt; the trace shows router → support → actions |
| 0:40 | $120 damaged lamp → "specialist reviewing" → console shows policy trace → approve → chat updates live |
| 1:20 | Attack: a poisoned product review tries to trigger a refund → the Support agent has no write scope and the handoff guard rejects the smuggled request; show the stopping layer and the audit entry |
| 1:50 | Kill switch → the Actions agent is disabled; Support keeps answering |
| 2:05 | Eval report: pass^k curve, red-team table, multi vs single agent; Langfuse trace with cost per agent |
| 2:25 | Architecture + repo link |

### 26.3 Write-ups

- "My support agent passed 91% of tests once, and 78% four times in a row. Here's why that gap matters."
- "I attacked my own AI agents 120 times. Here's what got through and what stopped it."
- "Why I split my support agent into four, and what it cost." (E11)

---

## 27. Working with coding agents on this project

Use Claude Code, Codex, Cursor or similar. The rules and workflow live in [AGENTS.md](../../AGENTS.md), which `CLAUDE.md` loads; they aren't repeated here, so there is one source of truth. In short: every task starts from an issue, the agent proposes a plan first, and a human reviews every diff, runs the tests and evals, and owns the scenario, red-team and calibration labels, the thresholds and approval limits, metric interpretation, the review of security-sensitive code and the README claims.

- **CODEOWNERS** (`.github/CODEOWNERS`) marks the security-sensitive paths, so they always get your explicit review, even when an agent wrote the change.
- **Agent Skills (optional):** a `redteam` skill that runs the attack suite and summarizes attack success by category and stopping layer. Keep the security rules in `AGENTS.md`, never only in a skill.
- **Build log (`docs/build-log.md`):** record notable sessions: what you asked, what the agent got wrong, how you caught it (a test? an eval drop?), and what you changed. Good entries for this project include:
  - a time the agent proposed putting a check in the prompt and you moved it to code;
  - a time the agent tried to add a free-text field to a handoff contract and you rejected it;
  - a test that caught an agent-written bug in idempotency.

**Rule:** never commit code you can't explain line by line.
