# Engineering standards

These standards apply to everything in this repository. The project-specific design is in [design.md](design.md).

## 1. Repository hygiene

- [ ] README follows the blueprint in [design §26](design/10-decisions-and-deliverables.md#26-deliverables-readme-demo-video-write-up). **Results come before setup instructions.**
- [ ] `LICENSE` (MIT), `.env.example`, `Makefile` with standard targets (`setup`, `up`, `lint`, `test`, `seed`, `eval-smoke`, `eval-agent`, `verify-audit`, `run`).
- [ ] `docker compose up` works on a fresh machine (test it in a clean clone).
- [ ] `pre-commit` with ruff, mypy, gitleaks and markdownlint.
- [ ] **Conventional Commits** ([COMMIT_MESSAGE_GUIDELINES.md](../COMMIT_MESSAGE_GUIDELINES.md)); small, squash-merged PRs ([PULL_REQUEST_GUIDELINES.md](../PULL_REQUEST_GUIDELINES.md)).
- [ ] **Semantic versioning tags** + GitHub Releases whose notes include eval numbers (`v0.1.0` baseline → `v1.0.0` launch; [VERSIONING.md](../VERSIONING.md)).
- [ ] `CHANGELOG.md`; PR template with an **"Eval impact"** section; issue templates.
- [ ] Branch protection on `main` (CI must pass). Dependabot and CodeQL enabled. `.github/CODEOWNERS` marks the security-sensitive paths.
- [ ] Work is tracked in GitHub issues: each phase is a sub-issue of [#1](https://github.com/dileepadev/relay-support-agents/issues/1), with its tasks as sub-issues ([design §22](design/09-plan-and-risks.md#22-implementation-plan-phases-and-milestones)).

## 2. Code quality

- Typed Python (mypy strict on core packages), ruff, pytest. Coverage ≥ 80% on core logic; 100% branch coverage on the policy engine.
- 12-factor configuration (env vars and config files, no hardcoded secrets or model IDs).
- Structured logging with request/trace IDs; no `print`.
- No global mutable state; small functions; explicit error handling.

## 3. LLM engineering

- A **model-agnostic LLM interface**; model IDs and prices in config; cost computed from each response's `usage`.
- Retries with exponential backoff and jitter (SDK built-ins), timeouts, fallback models, and checks on `stop_reason` (`refusal`, `max_tokens`).
- Prompts are versioned files (prompts as code), reviewed in PRs like code.
- Prompt caching for stable prefixes; budgets per request/conversation; daily caps; kill switches.
- **API drift note (2026):**
  - current Claude API versions reject `temperature`/`top_p`/`top_k`;
  - the newest Claude models reject forced tool choice and assistant prefill;
  - thinking tokens are billed as output.
  - Design for these, and measure variance with repeated runs.

## 4. Evaluation

- Scenario and red-team suites are **versioned**, each with a **datasheet** (sources, labeling rules, counts, known biases).
- Dev/test split. **Never tune on test.** Headline numbers come from test.
- `evals/thresholds.yaml`, a PR smoke suite with a sticky PR comment, and scheduled full runs.
- LLM judges **calibrated against your own labels** (report Cohen's κ).
- Report confidence intervals or repeated-trial variance (pass^k).
- `docs/error-analysis.md` with a failure taxonomy and counts over time.

## 5. Observability

OpenTelemetry with the GenAI semantic conventions → Langfuse; one trace per conversation turn; cost per request; dashboards; an external uptime check on the live demo.

## 6. Security and privacy

- Map the system to the OWASP Top 10 for LLM Applications (2025) and the OWASP Top 10 for Agentic Applications.
- Secrets in a secret manager. CI deploys via OIDC (no long-lived cloud keys). Least-privilege roles.
- SBOM, dependency scanning and image scanning.
- `docs/threat-model.md` (threat model and OWASP mappings) and `docs/privacy.md` (data inventory, retention, provider terms).

## 7. Documentation as code

- The design is the architecture document: [design.md](design.md), with Mermaid diagrams in [§7](design/02-architecture.md#7-architecture).
- `docs/adr/` in MADR format: one short file per new or changed decision. The initial decisions are in [design §25](design/10-decisions-and-deliverables.md#25-architecture-decision-records).
- `docs/runbook.md`, `docs/system-card.md` (intended use, limits, evaluation, risks).
- `docs/build-log.md` (see `AGENTS.md`).

## 8. Cost controls

- Set spend limits in each model provider's console.
- Set an AWS Budget alert (for example at $30 and $60). Run `terraform destroy` when the stack is idle.
- Iterate with cheaper models, then run the final reported evals with the model you chose as default.
- Use the Batch API (about 50% off) for offline, single-turn jobs such as judging. Cache eval generations.
- Re-check model prices before publishing any cost numbers.

## 9. Definition of done

- [ ] Live demo URL (or a recorded demo + IaC to recreate it) and a 2.5-minute video.
- [ ] README with **real** results on the test split.
- [ ] CI green; eval gate active; release tagged.
- [ ] Security and privacy docs; red-team report; ADRs; runbook; system card.
- [ ] Write-up published ([design §26](design/10-decisions-and-deliverables.md#26-deliverables-readme-demo-video-write-up)).
