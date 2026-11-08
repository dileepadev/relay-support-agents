# Relay Support Agents: A Multi-Agent Customer-Support System That Takes Real Actions, Safely

> **A small team of AI support agents that can actually do things, like look up orders, issue refunds and cancel orders, without being tricked into doing the wrong things.** Each agent gets only the permissions it needs: the agents that read customer messages and untrusted content can never move money, and the agent that moves money never reads untrusted content. Built on its own MCP tool server, with human approval for risky actions, rules enforced in code rather than in prompts, and an evaluation suite that tests reliability, handoffs and attacks against a single-agent baseline.

| | |
| --- | --- |
| **Repository** | `relay-support-agents` |
| **GitHub description** | Multi-agent AI customer-support system that takes real actions safely: router, support, actions and escalation agents with separated privileges and typed handoffs, an MCP tool server with per-agent scopes, LangGraph orchestration, human-in-the-loop approvals, a deterministic policy engine, τ-bench-style simulated-user evals and red-teaming |
| **GitHub topics** | `ai-agents` `multi-agent-systems` `agentic-ai` `mcp` `model-context-protocol` `langgraph` `human-in-the-loop` `guardrails` `prompt-injection` `llm-security` `red-teaming` `evals` `customer-support` `fastapi` `nextjs` `aws` `terraform` `opentelemetry` `python` |

---

## Contents

Each section lives in its own file under [design/](design/). Section numbers (§) never change, so references like [`docs/design.md §8.3`](design/03-components.md#83-mcp-server-relay-support-agents-mcp) in issues and docs always point to the same section.

| § | Section | File |
| --- | --- | --- |
| 0 | [TL;DR](#0-tldr) | [design.md](design.md) |
| 1 | [Scope at a glance](#1-scope-at-a-glance) | [design.md](design.md) |
| 2 | [Problem statement](design/01-problem-and-requirements.md#2-problem-statement) | [01-problem-and-requirements.md](design/01-problem-and-requirements.md) |
| 3 | [Goals, non-goals, success metrics](design/01-problem-and-requirements.md#3-goals-non-goals-success-metrics) | [01-problem-and-requirements.md](design/01-problem-and-requirements.md) |
| 4 | [Users, use cases and the store policy](design/01-problem-and-requirements.md#4-users-use-cases-and-the-store-policy) | [01-problem-and-requirements.md](design/01-problem-and-requirements.md) |
| 5 | [Requirements](design/01-problem-and-requirements.md#5-requirements) | [01-problem-and-requirements.md](design/01-problem-and-requirements.md) |
| 6 | [Solution overview](design/02-architecture.md#6-solution-overview) | [02-architecture.md](design/02-architecture.md) |
| 7 | [Architecture](design/02-architecture.md#7-architecture) | [02-architecture.md](design/02-architecture.md) |
| 8 | [Component design](design/03-components.md#8-component-design) | [03-components.md](design/03-components.md) |
| 9 | [Data model](design/04-data-api-stack.md#9-data-model) | [04-data-api-stack.md](design/04-data-api-stack.md) |
| 10 | [API design](design/04-data-api-stack.md#10-api-design) | [04-data-api-stack.md](design/04-data-api-stack.md) |
| 11 | [Tech stack](design/04-data-api-stack.md#11-tech-stack) | [04-data-api-stack.md](design/04-data-api-stack.md) |
| 12 | [Evaluation plan](design/05-evaluation.md#12-evaluation-plan) | [05-evaluation.md](design/05-evaluation.md) |
| 13 | [Observability and operations](design/06-operations-and-cost.md#13-observability-and-operations) | [06-operations-and-cost.md](design/06-operations-and-cost.md) |
| 14 | [Security (the core of this project)](design/07-security-and-privacy.md#14-security-the-core-of-this-project) | [07-security-and-privacy.md](design/07-security-and-privacy.md) |
| 15 | [Privacy and compliance](design/07-security-and-privacy.md#15-privacy-and-compliance) | [07-security-and-privacy.md](design/07-security-and-privacy.md) |
| 16 | [Scaling and performance](design/06-operations-and-cost.md#16-scaling-and-performance) | [06-operations-and-cost.md](design/06-operations-and-cost.md) |
| 17 | [Cost model](design/06-operations-and-cost.md#17-cost-model) | [06-operations-and-cost.md](design/06-operations-and-cost.md) |
| 18 | [Reliability and failure modes](design/06-operations-and-cost.md#18-reliability-and-failure-modes) | [06-operations-and-cost.md](design/06-operations-and-cost.md) |
| 19 | [Testing strategy](design/08-testing-repo-ci.md#19-testing-strategy) | [08-testing-repo-ci.md](design/08-testing-repo-ci.md) |
| 20 | [Repository structure](design/08-testing-repo-ci.md#20-repository-structure) | [08-testing-repo-ci.md](design/08-testing-repo-ci.md) |
| 21 | [CI/CD and infrastructure as code](design/08-testing-repo-ci.md#21-cicd-and-infrastructure-as-code) | [08-testing-repo-ci.md](design/08-testing-repo-ci.md) |
| 22 | [Implementation plan (phases and milestones)](design/09-plan-and-risks.md#22-implementation-plan-phases-and-milestones) | [09-plan-and-risks.md](design/09-plan-and-risks.md) |
| 23 | [Stretch goals](design/09-plan-and-risks.md#23-stretch-goals) | [09-plan-and-risks.md](design/09-plan-and-risks.md) |
| 24 | [Risks and mitigations](design/09-plan-and-risks.md#24-risks-and-mitigations) | [09-plan-and-risks.md](design/09-plan-and-risks.md) |
| 25 | [Architecture decision records](design/10-decisions-and-deliverables.md#25-architecture-decision-records) | [10-decisions-and-deliverables.md](design/10-decisions-and-deliverables.md) |
| 26 | [Deliverables: README, demo video, write-up](design/10-decisions-and-deliverables.md#26-deliverables-readme-demo-video-write-up) | [10-decisions-and-deliverables.md](design/10-decisions-and-deliverables.md) |
| 27 | [Working with coding agents on this project](design/10-decisions-and-deliverables.md#27-working-with-coding-agents-on-this-project) | [10-decisions-and-deliverables.md](design/10-decisions-and-deliverables.md) |
| 28 | [Appendices](design/appendices.md#28-appendices) | [appendices.md](design/appendices.md) |

---

## 0. TL;DR

**What you build:** a multi-agent support system for a fictional online store ("Harbor & Pine", home goods) that resolves real requests end to end: order status, refunds, cancellations, address changes, damaged-item claims and policy questions.

- **Four agents with separated privileges** ([§6](design/02-architecture.md#6-solution-overview)):
  - **Router:** no tools. Decides which agent handles each customer turn.
  - **Support agent:** talks to the customer and looks things up with **read-only** tools. It can *propose* an action, never execute one.
  - **Actions agent:** the only agent with **write** tools. It works from a validated, customer-confirmed action request (IDs, enums, amounts), never from raw chat or untrusted text.
  - **Escalation agent:** hands complex or risky cases to a human with a clean case summary.
- Agents hand off through **typed contracts checked in code**, never through free-text messages ([§8.4](design/03-components.md#84-multi-agent-runtime-langgraph)).
- Agents can touch data **only** through an MCP server you write, and each agent's token scopes decide which tools it may call ([§8.3](design/03-components.md#83-mcp-server-relay-support-agents-mcp)).
- Identity, ownership checks and refund limits are enforced in **deterministic code**, never trusted to an LLM ([§8.2](design/03-components.md#82-policy-engine)).
- Refunds above a limit **pause for a human** in an approvals console ([§7.4](design/02-architecture.md#74-approval-flow)).
- A **single-agent version** with the same tools and guards is kept as the baseline, so you can prove what the multi-agent design buys (E11 in [§12.6](design/05-evaluation.md#126-ablations)).
- You prove it works with **simulated customers** and you **attack it** with prompt injection, social engineering, cross-account and cross-agent tricks ([§12](design/05-evaluation.md#12-evaluation-plan)).
- It runs on AWS, built with Terraform, with full tracing ([§7.6](design/02-architecture.md#76-deployment-on-aws), [§13](design/06-operations-and-cost.md#13-observability-and-operations)).

---

## 1. Scope at a glance

| Area | What Relay Support Agents covers |
| --- | --- |
| Agents | Four agents with separated privileges (router, support, actions, escalation) and typed handoffs checked by a handoff guard in code ([§8.4](design/03-components.md#84-multi-agent-runtime-langgraph)) |
| Tools | Its own MCP server with 12 typed tools, per-agent scopes, identity from a signed token, idempotency and audit ([§8.3](design/03-components.md#83-mcp-server-relay-support-agents-mcp)) |
| Rules | A deterministic, versioned policy engine for the store policy R1–R12, with a decision trace ([§4.4](design/01-problem-and-requirements.md#44-store-policy), [§8.2](design/03-components.md#82-policy-engine)) |
| Humans in the loop | Confirmation cards rendered by code, specialist approvals that pause and resume durably, and a human available at any time ([§7.4](design/02-architecture.md#74-approval-flow), [§8.6](design/03-components.md#86-approvals-console-and-ops-dashboard-nextjs)) |
| Evaluation | Simulated-customer scenarios graded on the database end state, pass^k reliability, a calibrated communication judge, ablations E0–E12 and a CI gate ([§12](design/05-evaluation.md#12-evaluation-plan)) |
| Security | A red-team suite with a "which layer stopped it" table, OWASP LLM and Agentic Top 10 mappings and a hash-chained audit log ([§14](design/07-security-and-privacy.md#14-security-the-core-of-this-project)) |
| Context engineering | A cacheable prefix per agent, just-in-time policy retrieval, compaction and per-agent context views ([§8.10](design/03-components.md#810-context-engineering)) |
| Production | FastAPI and Next.js on AWS ECS Fargate and RDS via Terraform, GitHub OIDC deploys, OpenTelemetry tracing, budgets and kill switches ([§7.6](design/02-architecture.md#76-deployment-on-aws), [§13](design/06-operations-and-cost.md#13-observability-and-operations)) |
| Languages | Python backend and a TypeScript/Next.js frontend |

**Kept small on purpose:** 6 intents, 12 tools and 4 agents; one MCP server, one database (PostgreSQL), one agent API and one web app; synthetic data and simulated payments only. Anything beyond that is a stretch goal ([§23](design/09-plan-and-risks.md#23-stretch-goals)).

**Cost:** about $55 a month while the AWS stack is up (run `terraform destroy` when it's idle) and an LLM API budget of about $100–150 for the whole project. The details are in [§17](design/06-operations-and-cost.md#17-cost-model).
