# Relay Support Agents design: Operations, scaling and cost

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 13. Observability and operations

| Signal | Details |
| --- | --- |
| Traces | Conversation turn → router decision, agent runs, handoffs (payload, result), LLM calls (tokens, model), tool calls (name, result code, latency; args redacted), guard decisions, approval waits |
| Product metrics | Containment rate (resolved without a human), escalation rate, approval rate, approval queue time, CSAT proxy (thumbs up/down) |
| Engineering metrics | Tool error rate, guard trigger counts by type, steps per turn, handoffs per turn, rejected handoffs, tokens and cost per agent and per conversation, p50/p95 turn latency |
| Security metrics | Injection-classifier flags, blocked tool calls, output-guard rewrites, cross-account attempts |
| Logs | Structured JSON in CloudWatch with conversation and trace IDs; PII-redacted |

- **SLOs (demo):** 99% availability; p95 agent turn ≤ 8 s; 95% of approvals decided within 30 minutes (staffed hours).
- **Alerts:**
  - any security event in production;
  - approval queue older than 30 minutes;
  - tool error rate > 5% for 10 minutes;
  - daily cost at 80% of the cap.
- **Kill switches (feature flags in the database, cached for 10 s):** `WRITE_TOOLS_ENABLED` (read-only mode: the Actions agent is disabled; Support can still look things up and Escalation can hand off) and `AGENT_ENABLED` (all agents off; chats go straight to human handoff).
- **Runbook (`docs/runbook.md`):**
  - **Suspected abuse:** flip write tools off → freeze the customer → `make verify-audit` → review traces → fix → add an attack case.
  - **Provider outage:** fallback model → if both are down, the handoff message plus a ticket.
  - **Bad deploy:** ECS rollback to the previous task definition.

---

## 16. Scaling and performance

### 16.1 Today

A demo with a handful of concurrent conversations, one region, one database.

### 16.2 Capacity math

What would 1,000 concurrent chats need? Assume each turn makes about 2 LLM calls of about 6,000 input tokens, of which about 3,500 is the stable prefix (system prompt + tool definitions + policy summary). Assume a turn every 30 s per conversation.

- 1,000 concurrent chats ≈ 33 turns/s ≈ 67 LLM calls/s ≈ **4M input tokens/minute** and about 0.3M output tokens/minute.
- This hits **provider rate limits** before it hits your servers. So:
  - request higher tiers;
  - spread load across a primary and a fallback provider;
  - cache the prefix (cached reads are billed at a fraction of normal input);
  - use small models where evals allow (the Router always; often Actions and Escalation).

### 16.3 Architecture changes by tier

**T1, demo:** as designed.

**T2, 1k concurrent chats:**

- Horizontally scale the stateless API on ECS (state lives in the Postgres checkpointer).
- The MCP service is stateless under the 2026-07-28 spec, so load balancing is simple.
- RDS Proxy for connection pooling.
- Async approval results reach the customer's open SSE stream via Postgres `LISTEN/NOTIFY` or Redis pub/sub, because the approval may resume on a different instance.
- Prompt caching per agent; small models for the Router and other agents where evals allow.

**T3, multi-tenant platform:**

- `tenant_id` everywhere + Postgres row-level security.
- Per-tenant policy files, models, budgets and tool scopes.
- A queue for long-running actions (or the MCP Tasks extension for external clients).
- Separate read and write tool servers.
- Regional deployments for data residency; per-tenant dashboards and audit exports.

### 16.4 Latency levers

Streaming; parallel read-only tool calls; prompt caching of the stable prefix (also lowers TTFT); trimming long tool outputs; summarizing (compacting) long conversations; lower effort settings where evals show no quality loss.

---

## 17. Cost model

Anthropic API prices (cached October 2026, per million tokens):

| Model | Input | Output | Cached input read |
| --- | --- | --- | --- |
| Opus 5.5 (`claude-opus-5-5`) | $4.00 | $20.00 | $0.20 |
| Sonnet 5.5 (`claude-sonnet-5-5`) | $2.00 | $10.00 | $0.20 |
| Haiku 5.5 (`claude-haiku-5-5`) | $0.10 | $0.50 | — |

Re-check current pricing and cache-write prices before publishing numbers.

### 17.1 Per conversation

Assumptions for the Support agent: 4 customer turns × 2 LLM calls = 8 calls. Each call has 6,000 input tokens (3,500 cacheable prefix) and about 400 output tokens, including thinking tokens because those bill as output. Cache-write premiums are ignored for simplicity. **Measure your own from `usage`.**

| Support agent model | Without caching | With prefix caching |
| --- | --- | --- |
| Opus 5.5 | 8 × (6,000 × $4/M + 400 × $20/M) ≈ **$0.26** | 8 × (3,500 × $0.20/M + 2,500 × $4/M + 400 × $20/M) ≈ **$0.15** |
| Sonnet 5.5 | ≈ **$0.13** | ≈ **$0.08** |
| Haiku 5.5 | ≈ **$0.006** | lower still |

**Multi-agent overhead:** add a Router call per turn on a small model (well under a cent per conversation) and one or two Actions/Escalation runs on cheaper models when an action happens. Measure the real overhead against the single-agent baseline in E11; small models for the Router, Actions and Escalation agents can make the multi-agent version cheaper than one large-model agent.

**Takeaway:** model choice changes cost about 25×. Caching saves about 40% on the expensive models. Routing simple intents to a small model saves more. Prove each with ablations E6–E8 before choosing.

### 17.2 Evaluation runs

- **Full suite:** 80 scenarios × 4 trials = 320 conversations. With a Sonnet 5.5 Support agent ≈ $25, plus the other agents ≈ $3–5, the simulator (cheap model) ≈ $1–3 and the judge ≈ $3, so **≈ $35 per full run**. With an Opus 5.5 Support agent, roughly double. Running the single-agent baseline as well doubles the cost of a comparison run.
- **PR smoke** (20 scenarios + 20 attacks): ≈ $2–5.
- **Plan:** iterate with a cheaper agent model and run final, reported numbers with your chosen default. **Project API budget ≈ $100–150.** Set provider spend limits.

### 17.3 Hosting on AWS

Approximate monthly prices in us-east-1; verify them with the AWS Pricing Calculator.

| Item | Approx. |
| --- | --- |
| 2 Fargate tasks (0.25 vCPU / 0.5 GB each) | ~$18 |
| Application Load Balancer | ~$20 |
| RDS PostgreSQL db.t4g.micro + 20 GB | ~$14 |
| NAT gateway (avoid for demos: public subnets with strict security groups, or VPC endpoints) | ~$33 |
| Secrets Manager, CloudWatch, ECR | ~$3–5 |
| **Total** | **~$55/mo without NAT, ~$90 with** |

- **Practical approach:** `terraform apply` for the deployment milestone, demo recordings and live demos; `terraform destroy` otherwise. Use AWS free-tier credits for new accounts if available (check current terms).
- **Budget alternative** for an always-on demo: the same containers on Cloud Run, Fly.io or Render with managed Postgres.
- Set an **AWS Budget alert** at $30 and $60.

---

## 18. Reliability and failure modes

| Failure | Handling |
| --- | --- |
| LLM provider errors / 429 | Retries with backoff (SDK built-ins); fallback model; if all fail, polite handoff + ticket |
| MCP server unavailable | The agent replies "I can't access orders right now", offers a human, creates a ticket |
| Database unavailable | 503 with status message; alarms; RDS automated backups/snapshots for recovery |
| Approval never decided | 30-minute SLA → auto-escalate to a lead; the customer is told and gets a ticket number; after 24 h the approval expires and the action is not executed |
| Duplicate submissions / retries | Idempotency keys (UNIQUE constraint) return the original result |
| Lost response after a write | Retrying with the same idempotency key is safe and returns the same refund |
| Policy changed while waiting | Re-check at execution; if no longer eligible, the agent explains and escalates |
| Budget exhaustion / loops | Escalate with a summary; never retry silently |
| Handoff loop | Handoff limit per turn → Escalation agent |
| `ActionRequest` rejected repeatedly | Support asks the customer to clarify; after two rejections, escalate |
| One agent's model unavailable | Per-agent fallback model; if the Actions agent can't run, the action stays pending and the customer is told |
| Simulator flakiness (evals) | Mark the run invalid and report the simulator error rate separately |
