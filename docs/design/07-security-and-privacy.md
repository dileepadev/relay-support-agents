# Relay Support Agents design: Security and privacy

Part of the [Relay Support Agents design](../design.md). Section numbers (§) match the index and the GitHub issues.

---

## 14. Security (the core of this project)

### 14.1 Principles

1. **The LLM is not a security boundary.** Authorization, limits and policy live in deterministic code.
2. **Identity comes from authentication, never from the model.** No tool takes `customer_id`.
3. **Least privilege per agent:** a scoped token for each agent; only the Actions agent holds write scopes; separate customer and admin tool servers; minimal fields; INSERT-only audit role.
   - **Separate reading from acting:** no agent both reads untrusted content and moves money. The agent that acts receives only validated IDs, enums and amounts, confirmed by the customer.
4. **Humans approve high-impact actions**, with single-use approval tokens bound to exact parameters.
5. **Defense in depth:** prompt rules, spotlighting, privilege separation, guards, handoff validation, policy engine, approvals, audit. Measure which layer stopped what.
6. **Break the "lethal trifecta".** An agent is most dangerous when it combines private data, untrusted content and a way to send data out. Relay Support Agents has the first two, so the third is removed: no web access, no arbitrary email, links allowlisted, no images in output.
7. **Assume compromise of the prompt** and limit the blast radius with kill switches, budgets and rate limits.

### 14.2 Threat model (STRIDE summary)

| Threat | Example | Mitigation |
| --- | --- | --- |
| Spoofing | Pretending to be another customer | Session auth; OTP verification for address changes; identity in a signed token |
| Tampering | Editing audit records; altering approval parameters | Hash-chained audit, INSERT-only role; approval token bound to order, items and amount, single use |
| Repudiation | "I never approved that" | Approver identity, reason and timestamp in the audit log |
| Information disclosure | Seeing another customer's order | Ownership checks in every tool; uniform `NOT_FOUND`; minimized fields; output guard |
| Denial of service | Token floods, loops | Rate limits, budgets, max lengths, timeouts |
| Elevation of privilege | An agent calls admin functions, or a read-only agent calls a write tool | Per-agent scopes checked server-side; admin tools on a separate server no customer-facing agent loads |

### 14.3 OWASP Top 10 for LLM Applications (2025)

| Risk | Controls |
| --- | --- |
| LLM01 Prompt Injection | Policy in code; spotlighting; classifier risk flag; approvals; red-team suite |
| LLM02 Sensitive Information Disclosure | Ownership checks; field minimization; output guard; redacted traces |
| LLM03 Supply Chain | Locked dependencies; pinned MCP tool definitions; SBOM; image scanning |
| LLM04 Data and Model Poisoning | Help-center content from a reviewed source; reviews treated as untrusted |
| LLM05 Improper Output Handling | No HTML/Markdown images rendered from model output; link allowlist; sanitized rendering |
| LLM06 Excessive Agency | Per-agent minimal tool sets and scopes; only the Actions agent can write; approvals; kill switches; no admin tools |
| LLM07 System Prompt Leakage | No secrets or authorization logic in prompts |
| LLM08 Vector and Embedding Weaknesses | (If help-center RAG is used) tenant/source filters in SQL, not prompts |
| LLM09 Misinformation | Policy reasons returned verbatim from code; communication judge |
| LLM10 Unbounded Consumption | Per-turn/per-conversation budgets; rate limits; daily cap |

### 14.4 OWASP Top 10 for Agentic Applications (December 2025)

Verify the exact category titles at genai.owasp.org before publishing.

| Category | Controls |
| --- | --- |
| ASI01 Agent Goal Hijack | Spotlighting; goals can't change policy (code); red-team indirect-injection cases |
| ASI02 Tool Misuse | Typed strict schemas; tool guard; state-based allowlists; idempotency |
| ASI03 Identity and Privilege Abuse | Token-derived identity; a scoped token per agent; no customer_id arguments; separate admin server |
| ASI04 Agentic Supply Chain | Only your own allowlisted MCP servers; tool-hash pinning; dependency scanning |
| ASI05 Unexpected Code Execution | No code execution or shell tools at all |
| ASI06 Memory and Context Poisoning | No cross-conversation memory in v1; tool data marked untrusted |
| ASI07 Insecure Inter-Agent Communication | Agents never message each other directly. Handoffs are typed contracts passed through the in-process orchestrator, validated by the handoff guard, recorded in `handoffs` and the audit log. No free text crosses into the Actions agent |
| ASI08 Cascading Failures | Handoff limit per turn; budgets, timeouts, circuit breakers, kill switches; a tricked agent can't pass instructions to the next |
| ASI09 Human-Agent Trust Exploitation | Approvers see the policy trace, raw facts and risk flags, not just an agent's summary; escalation summaries are labeled AI-generated; required reasons |
| ASI10 Rogue Agents | Audit chain with the acting agent recorded, anomaly alerts, kill switches |

### 14.5 MCP-specific controls

- Validate token signature, **audience** and expiry on every request.
- **Never pass a client's token through** to other services.
- Validate the `Origin` header on the HTTP transport, and bind local development servers to `localhost`.
- Rate-limit per token. Audit every call.
- Pin and hash tool definitions, alert on change, and only connect to allowlisted servers.

### 14.6 Infrastructure security

- RDS in private subnets, encrypted at rest, automated backups, no public access.
- Security groups allow ALB → API, API → MCP and services → RDS only.
- TLS everywhere (ACM certificate on the ALB).
- **GitHub Actions deploy via OIDC role assumption, with no long-lived AWS keys.** The trust policy is restricted to `repo:dileepadev/relay-support-agents:ref:refs/heads/main` ([Appendix F](appendices.md#appendix-f-github-oidc-deploy-role-terraform-excerpt)).
- Least-privilege ECS task roles; CloudTrail enabled.
- Secrets in Secrets Manager, injected as task secrets.
- Terraform scanned with Checkov or tfsec; images with Trivy; an SBOM with Syft.

---

## 15. Privacy and compliance

- **Synthetic data only.** Say this clearly in the README. The design treats the data as if it were real.
- **Data classification:**

| Field | Class | Handling |
| --- | --- | --- |
| Name, email, phone, address | Personal data | Masked in UI lists and traces; full value only where needed for an action |
| Payment method | Financial | Only brand + last 4 digits ever stored or shown (PCI DSS principle: the agent never sees card numbers) |
| Conversation transcripts | Personal data | Retained 30 days, then deleted or anonymized |
| Audit log | Business record | Retained 1 year; contains hashes, not full arguments |

- **Data minimization:** tools return only needed fields, and prompts never include more than the current customer's data.
- **Data subject rights:** export and delete endpoints ([§10](04-data-api-stack.md#10-api-design)); deletion anonymizes personal fields while keeping hashed audit integrity.
- **Transparency:** AI disclosure at the start; a human is always available (rule R12).
- **Provider terms:**
  - API data isn't used for training by default with the major providers;
  - know the retention windows;
  - check zero-data-retention eligibility before promising it, since some newer models and features are excluded;
  - consider regional processing options.
- **Regulatory awareness:** GDPR/UK GDPR principles, PCI DSS (why the agent never sees card data), EU AI Act transparency, and your local data-protection law.
- **Document it:** write `docs/privacy.md` and a `docs/system-card.md` (intended use, limits, eval results, known risks).
