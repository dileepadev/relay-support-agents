# Relay Support Agents design: Data model, API and tech stack

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 9. Data model

```sql
CREATE TABLE customers (
  id            TEXT PRIMARY KEY,               -- "C-104"
  name          TEXT NOT NULL,
  email         TEXT NOT NULL UNIQUE,
  phone         TEXT,
  fraud_flag    BOOLEAN NOT NULL DEFAULT false,
  created_at    TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE orders (
  id            TEXT PRIMARY KEY,               -- "HP-48213"
  customer_id   TEXT NOT NULL REFERENCES customers(id),
  status        TEXT NOT NULL CHECK (status IN ('placed','paid','fulfilled','shipped','delivered','cancelled')),
  total         NUMERIC(10,2) NOT NULL,
  currency      CHAR(3) NOT NULL DEFAULT 'USD',
  placed_at     TIMESTAMPTZ NOT NULL,
  delivered_at  TIMESTAMPTZ,
  notes         TEXT                           -- untrusted free text
);

CREATE TABLE order_items (
  id              TEXT PRIMARY KEY,
  order_id        TEXT NOT NULL REFERENCES orders(id),
  product_id      TEXT NOT NULL,
  quantity        INT NOT NULL CHECK (quantity > 0),
  paid_amount     NUMERIC(10,2) NOT NULL,
  refunded_amount NUMERIC(10,2) NOT NULL DEFAULT 0,
  final_sale      BOOLEAN NOT NULL DEFAULT false
);

CREATE TABLE approvals (
  id              TEXT PRIMARY KEY,             -- "AP-3310"
  conversation_id TEXT NOT NULL,
  action          TEXT NOT NULL,                -- "refund"
  payload         JSONB NOT NULL,               -- order, items, amount (bound to the token)
  policy_trace    JSONB NOT NULL,
  status          TEXT NOT NULL CHECK (status IN ('pending','approved','rejected','expired','used')),
  decided_by      TEXT,
  reason          TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
  decided_at      TIMESTAMPTZ
);

CREATE TABLE refunds (
  id              TEXT PRIMARY KEY,             -- "RF-1042"
  order_id        TEXT NOT NULL REFERENCES orders(id),
  amount          NUMERIC(10,2) NOT NULL CHECK (amount > 0),
  status          TEXT NOT NULL CHECK (status IN ('completed','pending_approval','rejected')),
  approval_id     TEXT REFERENCES approvals(id),
  idempotency_key TEXT NOT NULL UNIQUE,         -- retries can never double-refund
  policy_version  TEXT NOT NULL,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE audit_log (
  id              BIGSERIAL PRIMARY KEY,
  ts              TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_type      TEXT NOT NULL CHECK (actor_type IN ('agent','staff','system')),
  actor_id        TEXT NOT NULL,                -- e.g. 'agent:actions', 'staff:S-12'
  conversation_id TEXT,
  action          TEXT NOT NULL,                -- tool or decision name
  args_hash       CHAR(64) NOT NULL,
  result_code     TEXT NOT NULL,
  policy_version  TEXT,
  prev_hash       CHAR(64) NOT NULL,
  row_hash        CHAR(64) NOT NULL
);
CREATE TABLE handoffs (
  id              BIGSERIAL PRIMARY KEY,
  conversation_id TEXT NOT NULL,
  turn            INT NOT NULL,
  from_agent      TEXT NOT NULL,                -- router, support, actions, escalation, code
  to_agent        TEXT NOT NULL,
  contract        TEXT NOT NULL,                -- RouteDecision, ActionRequest, ActionResult, EscalationRequest
  payload         JSONB NOT NULL,               -- validated payload; no free text
  result          TEXT NOT NULL CHECK (result IN ('accepted','rejected')),
  reject_reason   TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
-- Plus: addresses, address_drafts, products, payments (including chargeback status), shipments,
-- store_credits, tickets, reviews, help_articles, conversations, feature_flags (kill switches).
-- LangGraph's Postgres checkpointer creates its own tables.
```

**Roles:**

- `relay_support_agents_mcp`: read/write on domain tables.
- `relay_support_agents_api`: checkpoints, approvals, conversations and feature flags; read-only on domain tables.
- Both app roles have INSERT and SELECT only on `audit_log`, never UPDATE or DELETE ([§8.8](03-components.md#88-tamper-evident-audit-log)).
- Migrations run under the owner role. Use Alembic.

---

## 10. API design

| Method and path | Purpose | Auth |
| --- | --- | --- |
| `POST /v1/conversations` | Start a conversation (demo persona or real auth) → `conversation_id` | Customer session |
| `POST /v1/conversations/{id}/messages` | Send a message; SSE stream of the reply | Customer session |
| `GET /v1/conversations/{id}` | Transcript and status | Customer (own) / staff |
| `POST /v1/conversations/{id}/handoff` | Ask for a human | Customer session |
| `POST /v1/conversations/{id}/confirmations/{action_id}` | `{confirmed}`; confirms or declines the pending action card | Customer session |
| `POST /v1/conversations/{id}/secure-input/{request_id}` | `{code}`; sends a one-time code straight to the MCP server ([§8.3](03-components.md#83-mcp-server-relay-support-agents-mcp)); never stored, logged or shown to a model | Customer session |
| `GET /v1/approvals?status=pending` | Approval queue | Staff |
| `POST /v1/approvals/{id}/decision` | `{approved, reason}`; resumes the graph | Staff |
| `GET /v1/audit?conversation_id=…` | Audit entries | Staff |
| `POST /v1/admin/kill-switch` | `{write_tools: false}` or `{agent: false}` | Staff (lead) |
| `POST /v1/admin/reset-demo` | Re-seed the demo database | Staff |
| `GET /v1/privacy/customers/{id}/export` | Data export (DSAR) | Staff |
| `DELETE /v1/privacy/customers/{id}` | Delete or anonymize a customer | Staff |
| `POST /mcp` (MCP service) | MCP Streamable HTTP endpoint | Service token |

**SSE events:**

- `token`: reply text delta.
- `tool_activity`: sanitized, with the agent name, e.g. "Support: checking your order…".
- `confirmation_required`: the action card (order, items, amount, action), rendered by code.
- `receipt`: an action completed, with a reference.
- `approval_pending`: includes the approval ID and expected wait.
- `approval_resolved`: the specialist's decision.
- `secure_input_required`: show the secure code field, with the request ID only.
- `handoff`, `done`, `error`.

---

## 11. Tech stack

| Layer | Choice | Why | Alternatives (ADRs) |
| --- | --- | --- | --- |
| Language | Python 3.12+ (backend), TypeScript (web) | Market standard | — |
| Agent orchestration | LangGraph supervisor graph with agent subgraphs + Postgres checkpointer | Durable execution, first-class interrupts for human-in-the-loop, explicit and testable handoffs | OpenAI Agents SDK (handoffs), Claude Agent SDK (subagents), Pydantic AI, A2A protocol (agents as separate services), plain loop |
| Tool protocol | MCP (official Python SDK), Streamable HTTP | Industry standard; any MCP client can reuse the tools | In-process function calling |
| LLMs | Config-driven per agent; defaults: Router `claude-haiku-5-5`, Support `claude-opus-5-5`, Actions and Escalation `claude-sonnet-5-5`; compare with one OpenAI model | Eval-driven choice per agent; a second provider for comparison and fallback | LiteLLM router |
| User simulator | A different model family from the agents (for example OpenAI when the agents run on Claude) | Avoids the agent and simulator "agreeing" with each other | — |
| Guardrails | Deterministic guards + optional injection classifier | Predictable, cheap, testable | NeMo Guardrails, Guardrails AI, Llama Guard family |
| Red-teaming | Custom attack suite + promptfoo red-team plugins | Targeted plus broad automated coverage | garak, PyRIT |
| API | FastAPI, SSE, Pydantic v2 | Typed, async | — |
| Web | Next.js, Tailwind, shadcn/ui | Standard | — |
| Database | PostgreSQL 16 (RDS) | Transactions, constraints, checkpoints | — |
| Infra | AWS ECS Fargate, ALB, RDS, Secrets Manager, CloudWatch, ECR; Terraform; GitHub Actions OIDC | Industry-standard container platform; reproducible infrastructure as code | App Runner, Cloud Run, Fly.io (budget path) |
| Observability | OpenTelemetry → Langfuse; CloudWatch | Vendor-neutral | Arize Phoenix, LangSmith, Datadog |
| Testing | pytest, hypothesis, Playwright, k6 | — | — |
| Security tooling | CodeQL, pip-audit, Trivy, gitleaks, Checkov or tfsec, Syft (SBOM) | Supply-chain and IaC scanning | — |
