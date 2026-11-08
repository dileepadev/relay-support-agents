# Relay Support Agents design: Problem and requirements

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 2. Problem statement

### 2.1 Context

Support teams spend most of their time on repetitive requests:

- "Where is my order?"
- returns and refunds
- cancellations
- address changes
- damaged items

FAQ chatbots answer questions but don't **resolve** anything. Agents that take actions can resolve requests, but acting creates new risks.

### 2.2 Who has this problem

E-commerce, fintech, SaaS, travel and telecom companies. A whole product category exists for it (Sierra, Decagon, Intercom Fin, Salesforce Agentforce and others), and companies hire engineers to build and customize these systems.

### 2.3 Why it's hard

| Challenge | What can go wrong |
| --- | --- |
| Policy compliance with edge cases | Refunding a final-sale item or an order outside the return window |
| Identity and authorization | Agent reveals or refunds **another customer's** order (confused deputy) |
| Untrusted input everywhere | A message, product review or order note contains instructions the agent follows (prompt injection) |
| Non-determinism | The agent works once, fails on the second try; customers see the worst case |
| Knowing when to escalate | Giving up too early, or never handing off |
| Auditability | Nobody can explain later why money moved |
| Cost and latency | Multi-turn loops with large prompts get slow and expensive |
| Evaluation | Multi-turn behavior can't be checked with single input/output pairs |
| Multi-agent coordination | Agents pass bad data to each other, bounce a case back and forth, or one tricked agent tricks the next |

---

## 3. Goals, non-goals, success metrics

### 3.1 Goals

- **G1:** Resolve 6 request types end to end ([§4.2](#42-supported-intents)).
- **G2:** Never execute an action that violates policy or authorization, whatever the input.
- **G3:** Humans approve high-impact actions. The agent pauses and resumes durably.
- **G4:** Prove reliability and safety with simulated-user evals and red-teaming that run in CI.
- **G5:** Production deployment with tracing, audit, budgets and kill switches.
- **G6:** Separate privileges across agents so that no single model both reads untrusted content and moves money, and prove the design against a single-agent baseline.

### 3.2 Non-goals

- Real payments (refunds are simulated records).
- Real customer data (synthetic only).
- Voice and email channels (stretch).
- Multilingual support (stretch).
- Fine-tuning.

### 3.3 Success metrics (v1 targets)

| Metric | Definition | Target |
| --- | --- | --- |
| Task success (pass^1) | Database end state matches the scenario's expected state | ≥ 0.85 |
| Reliability (pass^4) | Probability that 4 independent trials all succeed (τ-bench estimator, [§12.4](05-evaluation.md#124-reliability-passk)) | ≥ 0.70 |
| Critical policy violations | Any forbidden money movement or cross-account data exposure, in any eval | **0** |
| Critical attack success rate | Red-team attacks that cause unauthorized money movement or data exposure | **0%** |
| Overall attack success rate | Across all red-team categories | ≤ 5% |
| Routing correctness | Action routed correctly: auto / approval / escalate / deny | ≥ 0.95 |
| Escalation accuracy | Escalates when it should, and only then | ≥ 0.90 |
| Handoff accuracy | Router sends the turn to the right agent (on labeled turns) | ≥ 0.95 |
| Invalid handoffs executed | Action requests that failed validation but still reached the Actions agent | **0** |
| Multi-agent vs single-agent | Same scenarios and attacks for both designs (E11) | Report both |
| Communication quality | Judge rubric pass: AI disclosure, accurate policy statements, clear next step | ≥ 0.90 |
| Cost per resolved conversation | From API `usage` | Report per model; see [§17](06-operations-and-cost.md#17-cost-model) |
| Agent turn latency p95 | From customer message to the full reply | ≤ 8 s (excluding human approval wait) |

---

## 4. Users, use cases and the store policy

### 4.1 Personas

| Persona | Needs |
| --- | --- |
| **Customer** | Fast resolution, clear status, the option of a human |
| **Support specialist** (approver) | Enough context to approve or reject in under 30 seconds; a clear audit trail |
| **Support lead** | Containment and escalation rates, approval queue time, cost; policy changes |
| **Developer (you)** | Safe iteration: evals and red-team in CI, kill switches, traces |

### 4.2 Supported intents

1. Order status ("Where is my order?")
2. Refund or return request
3. Cancel order
4. Change shipping address
5. Damaged or missing item claim
6. Policy questions (returns, shipping), answered from the help center
7. Escalation or handoff to a human, available at any time

### 4.3 User stories and acceptance criteria

| ID | Story | Acceptance criteria |
| --- | --- | --- |
| US1 | As a customer, I ask where my order is | The agent finds the order (only among *my* orders) and gives carrier status and ETA, with no other customer's data in any tool call or reply |
| US2 | As a customer, I want a refund of $24 for an eligible item | The agent confirms details, the refund executes automatically, and I get a reference number |
| US3 | As a customer, I want a $120 refund for a damaged lamp | The agent tells me a specialist will review it. When the specialist approves, the refund executes and I'm notified in the same chat |
| US4 | As a customer, I try to cancel a shipped order | The agent explains it has shipped and offers a return instead; no cancellation happens |
| US5 | As a specialist, I approve or reject a pending refund | One screen shows the transcript summary, order, policy decision trace and risk signals; a reason is required; the decision is audited |
| US6 | As a lead, I flip a kill switch | All write tools are disabled within 10 seconds (the Actions agent stops); the Support agent can still look things up and the Escalation agent can hand off |
| US7 | As a developer, I open a PR | CI runs smoke scenarios plus critical attacks and posts results; any critical attack success fails the build |
| US8 | As a developer, I inspect a conversation trace | Each turn shows which agent handled it, every handoff with its validated payload, and why the router chose that agent |
| US9 | As a customer, I confirm an action | Before any refund, cancellation, address change or credit, I see a confirmation card with the exact order, items and amount, rendered by code, and nothing happens until I confirm |

### 4.4 Store policy

These rules (Harbor & Pine, `policy_version: 2026-11-01`) are enforced by the **policy engine in code**. The LLM sees a summary only so it can explain them.

| Rule | Detail |
| --- | --- |
| R1 Return window | Refunds only within 30 days of delivery |
| R2 Final sale | Items marked final sale can't be refunded |
| R3 Amount | Refund ≤ amount paid − previous refunds, to the original payment method only |
| R4 Auto-approve | Allowed if the customer's refunds in the last 24 hours (including this one) total ≤ $50 |
| R5 Specialist approval | Required if that 24-hour total is > $50 and ≤ $500 |
| R6 Escalate | > $500, fraud-flagged accounts, or open chargebacks go to a human, never the agent |
| R7 Cancellation | Only before shipment (status `placed` or `paid`) |
| R8 Address change | Only before fulfillment, and only after one-time-code verification |
| R9 Damaged / missing items | Must be reported within 14 days of delivery; refund or replacement; requires photo evidence (simulated flag) |
| R10 Goodwill credit | Store credit ≤ $15, at most once per 90 days per customer |
| R11 Identity | Customers can only see and act on their own orders and data |
| R12 Disclosure | The agent says it's an AI assistant and offers a human at any time |

---

## 5. Requirements

### 5.1 Functional

| ID | Requirement |
| --- | --- |
| FR1 | Commerce system of record with seeded synthetic data and edge-case fixtures |
| FR2 | Deterministic, versioned policy engine (R1–R12) with a full decision trace |
| FR3 | MCP server exposing typed tools ([§8.3](03-components.md#83-mcp-server-relay-support-agents-mcp)) with identity derived from a signed token |
| FR4 | Multi-agent runtime (LangGraph): router, support, actions and escalation agents with input, tool, handoff and output guards, budgets and durable checkpoints |
| FR5 | Human approval flow with pause/resume (interrupts), timeouts and notifications |
| FR6 | Customer chat UI with streaming replies, action receipts and human handoff |
| FR7 | Approvals console and ops dashboard |
| FR8 | Append-only, tamper-evident audit log of every tool call and decision |
| FR9 | Kill switches: write tools off (Actions agent disabled); agents off (human-only mode) |
| FR10 | Scenario-based evaluation with a user simulator; red-team suite; CI integration |
| FR11 | Privacy endpoints: export and delete a customer's data (staff only) |
| FR12 | Typed handoff contracts between agents, validated in code and audited |
| FR13 | Per-agent MCP tokens and scopes; per-agent tool pinning |
| FR14 | A single-agent baseline with the same tools, guards and MCP server, for comparison |

### 5.2 Non-functional

| Area | Requirement |
| --- | --- |
| Safety | Zero critical violations in eval and red-team; authorization enforced outside the LLM |
| Latency | p95 agent turn ≤ 8 s; streaming first token ≤ 2 s |
| Durability | A pending approval survives restarts and deploys (Postgres checkpoints) |
| Idempotency | Every write tool takes an idempotency key; retries never duplicate refunds |
| Observability | Every conversation is one trace; agent runs, handoffs, tool calls, guard decisions and approvals are spans |
| Cost | Per-conversation budget (tokens and $); daily cap; caching on the stable prompt prefix |
| Security | Least privilege everywhere; OIDC deploys; secrets in Secrets Manager; encrypted data at rest |
| Reproducibility | Deterministic seeds; versioned scenarios; model IDs and policy version recorded per run |
