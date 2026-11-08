# Relay Support Agents design: Implementation plan and risks

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 22. Implementation plan (phases and milestones)

The plan lives in GitHub issues: [#1](https://github.com/dileepadev/relay-support-agents/issues/1) lists the phases, and each phase issue lists its tasks as sub-issues with acceptance criteria. Work through the phases in order; inside a phase, do `priority: must` issues first, then `should`, then `could` (MoSCoW: M = must, S = should, C = could).

| Phase | Goal | Milestone and release |
| --- | --- | --- |
| [0 · Prep](https://github.com/dileepadev/relay-support-agents/issues/2) | Rules, permissions, data model and the first scenarios designed before any agent; repo and tooling ready | — |
| [1 · Domain, tools, baseline](https://github.com/dileepadev/relay-support-agents/issues/9) | Policy engine, MCP server, eval harness and a naive baseline (E0) that the harness measures | M1: baseline numbers · `v0.1.0` |
| [2 · Agents, reliability and safety](https://github.com/dileepadev/relay-support-agents/issues/20) | Four agents with typed handoffs, approvals, the single-agent baseline, 60 scenarios, 60 attacks and the CI eval gate | M2: safe agents · `v0.5.0` |
| [3 · Production](https://github.com/dileepadev/relay-support-agents/issues/43) | Tracing, kill switches, privacy endpoints, AWS via Terraform, security docs and a results-first README | M3: deployed v1.0 · `v1.0.0` |
| [4 · Polish and publish](https://github.com/dileepadev/relay-support-agents/issues/57) | The remaining ablations and the public write-ups | — |
| [Backlog](https://github.com/dileepadev/relay-support-agents/issues/61) | Stretch goals ([§23](#23-stretch-goals)), only after v1.0 | — |

A milestone is reached when its phase's `must` tasks are done and the real numbers are reported, even if some targets are missed. Tasks that touch security-sensitive paths (listed in `AGENTS.md`) need a line-by-line human review.

### Phase 0: prep

| Task | Done when |
| --- | --- |
| [M] Commit the design docs and agent instructions | Docs on `main`; `CLAUDE.md` loads `AGENTS.md` |
| [M] Commit the community standards and configure repository governance (CODEOWNERS, branch protection, Dependabot, CodeQL, labels) | Direct pushes to `main` are blocked; a test PR shows the template |
| [M] Scaffold the repository and tooling (uv, ruff, mypy, pytest, pre-commit, Makefile, Postgres in compose, CI) | `make setup && make test` passes on a clean clone; CI is green |
| [M] Write `policies/POLICY.md` (R1–R12), the 12-tool catalog and the per-agent permissions table (agent → tools → scopes). **Design the rules and permissions before the agents.** | Reviewed by a human; limits and windows in `policies/2026-11-01.yaml` |
| [M] Draft the Postgres schema and seed 5 hand-made customers with edge cases | The 5 customers cover the R1–R10 boundaries |
| [M] Write the first 5 scenarios, with expected agent paths, **before** writing any agent | 5 human-written scenario files in `evals/scenarios/` |

### Phase 1: domain, tools, baseline (milestone **M1: baseline numbers**)

| Task | Done when |
| --- | --- |
| [M] Model-agnostic LLM client: model IDs and prices from config, cost from `usage`, retries, fallback model, `stop_reason` checks, recorded responses for tests | Unit tests pass without network access |
| [M] Migrations, database roles and a seed generator with edge-case fixtures | `make seed` builds the same database every time |
| [M] Policy engine + unit and property tests | 100% branch coverage |
| [M] Hash-chained audit log with an INSERT-only role | `make verify-audit` reports the first tampered row |
| [M] MCP token identity, scope checks, rate limits and transport checks | A call with a wrong audience, an expired token or a missing scope is rejected (tested) |
| [M] MCP read tools (7) and the help-center articles | Tool contract snapshot committed; tested with MCP Inspector and a local MCP client |
| [M] MCP write tools (5) with idempotency keys, approval tokens and the secure-input round trip | The same key returns the same result (property test); contract snapshot updated |
| [M] Scenario format, user simulator, runner and grading (end state, trajectory, pass^k) | `make eval-agent` runs a scenario end to end and reports pass^k |
| [M] Scenario suite v0: 20 scenarios with a datasheet | Committed by a human |
| [M] Naive baseline E0 (single LLM loop, `customer_id` arguments, policy in the prompt only) and its report | `make eval-agent` prints success, pass^1, cost; tag `v0.1.0` |

### Phase 2: agents, reliability and safety (milestone **M2: safe agents**)

| Task | Done when |
| --- | --- |
| [M] Orchestrator graph: `CaseState`, per-agent context views, Router, Postgres checkpointer, budgets, handoff limit | Router accuracy measured on labeled turns; state survives an API restart |
| [M] Per-agent MCP tokens and scopes; per-agent tool loading and pinning | A write call with a read-only token is rejected (tested) |
| [M] Tool guard and spotlighting | Blocked calls return a structured reason (tested) |
| [M] Input and output guards | Guard unit tests pass; a high injection score forces approval |
| [M] Support agent (read-only) and Escalation agent | Lookup and escalation scenarios pass |
| [M] Handoff contracts, handoff guard and confirmation card | Invalid `ActionRequest`s never reach the Actions agent (tested) |
| [M] Actions agent (write scopes) with approval interrupts and templated receipts | An approval survives an API restart (tested) |
| [M] Agent API with SSE streaming ([§10](04-data-api-stack.md#10-api-design)) | OpenAPI contract test passes |
| [M] Single-agent baseline with the same tools and guards | Runs on the same suite |
| [M] Customer chat UI with the confirmation card | A refund under $50 works end to end |
| [M] Approvals console with staff login | The US3 flow works end to end (Playwright) |
| [M] Scenarios → 60 with expected agent paths and a dev/test split; simulator validation | Suite v1 and its datasheet committed by a human; simulator error rate reported |
| [M] Red-team suite → 60 hand-written attacks, including cross-agent attacks | The attack runner reports ASR per category and the stopping layer |
| [M] Communication judge, calibrated against 50 human labels | Cohen's κ reported |
| [M] CI eval gate: PR smoke scenarios + critical attacks with a sticky PR comment | A PR with a critical attack success fails CI |
| [M] k = 4 runs, error taxonomy and fixes | pass^1 ≥ 0.80, critical ASR = 0 (or gaps documented); `docs/error-analysis.md` committed |
| [M] E11: multi-agent vs single-agent on the same scenarios and attacks | Comparison table committed; tag `v0.5.0` |
| [S] Safety ablations E1–E4 | ASR per added layer in the report |
| [S] promptfoo red-team generation (+60 attacks) | Report merged |
| [S] Context engineering: cached prefixes, operator notices, compaction ([§8.10](03-components.md#810-context-engineering)) | Cache-hit rate and tokens per turn in traces |
| [S] Scheduled full runs twice a week (k = 4, full red-team, both designs) | Trend chart in `evals/reports/` |
| [C] A `redteam` agent skill ([§27](10-decisions-and-deliverables.md#27-working-with-coding-agents-on-this-project)) | Summarizes ASR by category and stopping layer |

### Phase 3: production (milestone **M3: deployed v1.0**)

| Task | Done when |
| --- | --- |
| [M] OpenTelemetry → Langfuse with redaction | Trace screenshot with every agent, handoff and tool span |
| [M] Kill switches, daily cap, demo reset, ops dashboard and audit view | Kill switch demo; the verify-chain button works |
| [M] Privacy endpoints (export, delete) and transcript retention | Deletion keeps the audit chain valid (tested) |
| [M] Terraform AWS (network, ECS, RDS, ALB, secrets, OIDC, alarms, budgets) | `terraform plan` is clean; Checkov passes |
| [M] Terraform and deploy workflows with OIDC; post-deploy smoke test and rollback | `terraform apply` from CI; live HTTPS URL |
| [M] Security scans, SBOM and IaC checks | Scans run in CI; the SBOM is attached to releases |
| [M] Threat model and privacy docs | `docs/threat-model.md` and `docs/privacy.md` merged after a human security review |
| [M] Red-team report | `docs/redteam-report.md` merged |
| [M] Runbook and system card | `docs/runbook.md` and `docs/system-card.md` merged |
| [S] Scenarios → v2 (80–100) from error analysis | Suite v2 and its datasheet committed by a human |
| [S] k6 load test with mocked LLM | Results in README |
| [C] Chaos tests: kill the MCP task mid-conversation; restart the API during a pending approval | Resume verified |
| [M] README, demo video, eval report | Tag `v1.0.0` |

### Phase 4: polish and publish

| Task | Done when |
| --- | --- |
| [S] Ablations E5–E8 (framework vs plain loop; model and cost per agent; caching; effort) | Write-up merged |
| [S] Ablations E9, E10 and E12 (simulator swap; context engineering; Actions agent vs deterministic executor) | Write-up merged |
| [M] Blog posts ([§26.3](10-decisions-and-deliverables.md#263-write-ups)) + LinkedIn post | Published and linked from the README |

**Day-1 checklist:**

1. Create `relay-support-agents`.
2. Write `POLICY.md` and the per-agent permissions table (agent → tools → scopes).
3. Draft the Postgres schema.
4. Seed 5 hand-made customers with edge cases.
5. Write the first 5 scenarios, with expected agent paths, **before** writing any agent.

---

## 23. Stretch goals

1. **Framework comparison write-up:**
   - LangGraph vs a plain provider-SDK loop with approval hooks (e.g. Anthropic's Tool Runner), the OpenAI Agents SDK (handoffs) or the Claude Agent SDK (subagents);
   - compare lines of code, latency, reliability (pass^k) and debugging experience.
2. **External benchmark:** run your agent design on τ²-bench's retail domain for a comparable number.
3. **Full MCP authorization (OAuth 2.1)**, so Claude Desktop, IDEs or other MCP clients can safely connect to your public MCP endpoint. Demo it.
4. **Slack approvals:** approve or reject from a Slack message with interactive buttons.
5. **Multi-tenancy:** a second fictional store with different policies, `tenant_id` + row-level security.
6. **Email channel:** inbound email → agent → templated reply, with the same guards.
7. **MCP Tasks extension:** expose pending approvals as tasks, so external MCP clients can poll for the result instead of blocking.
8. **MCP Apps:** ship an interactive refund-confirmation card that MCP hosts render in a sandboxed frame. Very current, and strong in a demo.
9. **Kubernetes:** a Helm chart and a local `kind` cluster deployment, if your target jobs list Kubernetes.
10. **Agents as separate services (A2A protocol):** run the Actions agent as its own service with its own credentials, talking to the orchestrator over A2A. Only worth it if it needs independent deploys or ownership; measure the latency and failure-handling cost.

---

## 24. Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| Too many tools/intents → never finish | Fix scope to 6 intents, 12 tools and 4 agents; MoSCoW |
| Multi-agent complexity without benefit | Keep the single-agent baseline; typed contracts; if E11 shows no gain, say so and explain the trade-off honestly |
| Flaky evals | Fresh seeds; k trials; simulator validation; report invalid runs separately |
| AWS bill surprise | Budgets + alerts; destroy when idle; skip NAT |
| Security mistakes | Threat model review; red-team in CI; fix in code before prompts |
| Framework/SDK API changes | Pin versions; isolate framework code in `agents/`; contract tests |
| LLM cost during iteration | Cheaper agent model while iterating; smoke subsets; spend limits |
| Scope too large | Phase 1–2 deliverables (safe agent + eval report) are already strong without AWS |
