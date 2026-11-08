# Relay Support Agents design: Architecture

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 6. Solution overview

**Core principle: the LLMs decide what to try; code decides what's allowed. And no single LLM both reads untrusted text and moves money.**

In 2026 terms, Relay Support Agents is an **agent harness**: the loop, tools, guards, budgets, context policy, handoffs and human checkpoints around the models. The harness's quality, not clever prompting, is what makes agents reliable.

1. **Domain services and policy engine.** Plain Python. They own the data, the rules and the audit log. No LLM inside.
2. **MCP server.** The *only* way any agent can read or change data. Typed tools, **per-agent scopes**, identity from a signed token, idempotency, rate limits.
3. **Multi-agent runtime.** A LangGraph supervisor graph. The orchestrator is code; each agent has its own prompt, model, tools and token:

   | Agent | Job | Tools (MCP scopes) | Reads untrusted content? | Can move money? |
   | --- | --- | --- | --- | --- |
   | **Router** | Classify each turn and pick the next agent | none | Customer message only | No |
   | **Support** | Talk to the customer, look things up, propose actions | Read-only (`orders:read`, `help:read`, `eligibility:read`) | Yes | No |
   | **Actions** | Execute a validated, customer-confirmed action request | Write (`actions:write`) + `eligibility:read` | **No** | Yes, through the policy engine |
   | **Escalation** | Hand the case to a human with a summary | `tickets:write`, `orders:read` | Yes (spotlighted) | No |

   - Guards run before and after every step, plus a **handoff guard** between agents.
   - Agents never message each other directly: they return typed outputs and the orchestrator decides the next step.
   - The customer confirms every action on a card rendered by code. Action outcomes reach the customer as **templated receipts**, so the Actions agent never writes free text to the customer.
   - Budgets on steps, handoffs, tokens and cost; Postgres checkpoints, so a conversation can pause for hours and resume.
4. **Single-agent baseline.** One agent with all customer tools, the same guards and the same MCP server. It exists to measure what the multi-agent design buys (E11).
5. **Apps.** A customer chat, and a staff console for approvals, audit and metrics.
6. **Evaluation and red-team harness.**
   - Seeds a fresh database.
   - Plays the customer with a separate LLM.
   - Checks the database end state, the tool path and the agent path.
   - Repeats each scenario k times to measure reliability.
   - Runs a library of attacks, including cross-agent ones.
   - Runs both designs on the same suite.
7. **Operations.** Tracing, metrics, alerts, kill switches, Terraform on AWS.

---

## 7. Architecture

### 7.1 System context

```mermaid
flowchart LR
    cust["Customer<br/>chat widget"]
    staff["Support specialist<br/>approvals console"]
    web["Next.js web app"]
    api["Agent API<br/>FastAPI + LangGraph<br/>router, support, actions, escalation"]
    mcp["Relay Support Agents MCP server<br/>typed tools"]
    domain["Domain services<br/>policy engine + audit"]
    db[("PostgreSQL<br/>orders, refunds, audit,<br/>agent checkpoints")]
    llm["LLM providers"]
    obs["Langfuse<br/>OpenTelemetry"]
    cust --> web
    staff --> web
    web --> api
    api --> llm
    api -->|MCP calls with per-agent scoped tokens| mcp
    mcp --> domain --> db
    api --> db
    api --> obs
    mcp --> obs
```

### 7.2 Trust boundaries

```mermaid
flowchart TB
    subgraph U["Untrusted input"]
        M["Customer messages"]
        RV["Product reviews, order notes,<br/>address fields"]
    end
    subgraph S["Semi-trusted: LLM agents"]
        RT["Router<br/>no tools"]
        SUP["Support agent<br/>read-only tools"]
        ESC["Escalation agent<br/>ticket tool"]
        ACT["Actions agent<br/>write tools, no untrusted input"]
    end
    subgraph T["Trusted: deterministic code"]
        HG["Handoff guard<br/>schema, enums, ownership, confirmation"]
        TG["Tool guard + MCP scopes<br/>per-agent allowlist, budgets"]
        AUTH["Authorization<br/>identity from token, ownership checks"]
        POL["Policy engine<br/>eligibility, limits, routing"]
        AUD["Audit log<br/>append-only, hash-chained"]
    end
    M --> RT
    M --> SUP
    RV -->|inside tool results| SUP
    RV -->|inside tool results| ESC
    SUP -->|ActionRequest| HG
    HG -->|validated and confirmed| ACT
    SUP --> TG
    ESC --> TG
    ACT --> TG
    TG --> AUTH --> POL --> AUD
```

**Reading the diagram:** anything in the top box can be malicious. The Router, Support and Escalation agents can be tricked, but they hold no write scopes, so the MCP server refuses any money movement they attempt. The Actions agent holds write scopes, but it only ever receives a request built from IDs, enums and amounts that code has validated and the customer has confirmed. It never sees free text. Everything in the bottom box is plain code that no LLM can talk its way past.

### 7.3 Agent graph (LangGraph)

```mermaid
flowchart TD
    S0(["New customer message"]) --> IG["Input guard<br/>length, injection score, PII tags"]
    IG --> RT["Router<br/>structured RouteDecision"]
    RT -->|question, lookup or new request| SUP["Support agent<br/>read-only tools"]
    RT -->|human requested, high risk, stuck| ESC["Escalation agent"]
    SUP -->|reply| OG["Output guard"]
    SUP -->|ActionRequest| HG["Handoff guard<br/>schema, ownership, enums, limits"]
    HG -->|rejected with reason| SUP
    HG -->|valid| CONF["Confirmation card<br/>rendered by code"]
    CONF -->|customer confirms| ACT["Actions agent<br/>write tools"]
    CONF -->|customer declines or edits| SUP
    ACT -->|tool calls| TG["Tool guard + MCP scopes"]
    TG --> EXEC["Execute via MCP"]
    EXEC -->|needs approval| HITL["Human approval<br/>interrupt + checkpoint"]
    HITL -->|approved| EXEC2["Execute approved action"]
    HITL -->|rejected| SUP
    EXEC --> RES["ActionResult<br/>templated receipt"]
    EXEC2 --> RES
    RES --> REPLY
    SUP -->|budget exhausted or ESCALATE| ESC
    ACT -->|ESCALATE or failure| ESC
    ESC --> OG
    OG --> REPLY(["Reply to customer"])
```

- The Router never reaches the Actions agent directly. The **only** path to a write tool is Support → handoff guard → customer confirmation → Actions.
- A per-turn handoff limit (default 4) stops agents bouncing a case back and forth; when it's hit, the case goes to the Escalation agent.

### 7.4 Approval flow

```mermaid
sequenceDiagram
    autonumber
    participant C as Customer
    participant S as Support agent
    participant A as Actions agent
    participant M as MCP server
    participant P as Policy engine
    participant H as Specialist
    C->>S: My lamp arrived broken, I want my 120 dollars back
    S->>M: get_order and check_refund_eligibility (read scopes)
    M->>P: Evaluate rules R1 to R6 and R9
    P-->>M: Eligible, route = approval, rule R5
    M-->>S: Eligible, route = approval
    S-->>C: Confirmation card - refund 120 dollars for the lamp on HP-48213?
    C->>S: Confirms
    Note over S,A: Handoff guard validates the ActionRequest
    S->>A: ActionRequest - refund, HP-48213, lamp, 120.00, reason damaged
    A->>M: request_refund with idempotency key (write scope)
    M->>P: Evaluate again
    P-->>M: route = approval
    M-->>A: NEEDS_APPROVAL with policy trace
    A-->>C: Templated - a specialist will review this, usually within 30 minutes
    A->>A: Interrupt and checkpoint state in Postgres
    H->>A: Approve, reason - photo confirms damage
    A->>M: request_refund with idempotency key and approval token
    M->>P: Re-check eligibility and verify approval token
    P-->>M: OK
    M-->>A: Refund RF-1042 created
    A-->>C: Templated receipt - refund of 120 dollars, reference RF-1042
```

The policy is checked **twice**: once to route the request, and again at execution time. Something may have changed while it waited, for example another refund went through. The approval token is single-use and bound to the exact order, items and amount.

### 7.5 Evaluation harness

```mermaid
flowchart LR
    SC["Scenario YAML<br/>persona, hidden goal,<br/>expected end state"] --> SEED["Seed a fresh database"]
    SEED --> SIM["User simulator<br/>separate LLM"]
    SIM <-->|conversation| AGT["Relay Support Agents<br/>multi-agent or single-agent baseline"]
    AGT --> DBS[("Database end state")]
    DBS --> CHECK["Assertions<br/>end state, tool path,<br/>agent path, routing"]
    CHECK --> RES["Results<br/>pass^k, violations, cost, turns"]
    RT["Red-team attacks"] --> SEED
```

### 7.6 Deployment on AWS

```mermaid
flowchart LR
    subgraph AWS["AWS account"]
        ALB["Application Load Balancer<br/>HTTPS"]
        subgraph ECS["ECS Fargate cluster"]
            APIS["agent-api service"]
            MCPS["mcp-server service<br/>internal only"]
        end
        RDS[("RDS PostgreSQL<br/>encrypted, automated backups")]
        SM["Secrets Manager"]
        CW["CloudWatch<br/>logs and alarms"]
        ECR["ECR<br/>container images"]
    end
    VER["Vercel<br/>Next.js web"] --> ALB --> APIS
    APIS --> MCPS
    APIS --> RDS
    MCPS --> RDS
    SM -.-> APIS
    SM -.-> MCPS
    APIS --> CW
    MCPS --> CW
    GH["GitHub Actions<br/>OIDC deploy role"] --> ECR
    GH --> ECS
```
