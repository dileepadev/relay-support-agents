# Security Policy

The [Relay Support Agents](https://github.com/dileepadev/relay-support-agents) project team takes security issues very seriously. We appreciate your efforts to responsibly disclose any security vulnerabilities you find.

## Supported Versions

Relay Support Agents is in early development. Security fixes are made on `main` and included in the next release. Only the latest release and the live demo receive fixes.

## Reporting a Vulnerability

If you discover a security vulnerability, please send an email to **<contact@dileepa.dev>**. Please do not create a public issue for the vulnerability.

Your email should include the following information:

- A description of the vulnerability
- Steps to reproduce the vulnerability (for prompt-injection or agent issues, the demo persona, the exact messages and, if available, the conversation ID or trace ID)
- Possible impact of the vulnerability
- Any suggested mitigation or remediation steps

We will respond to your email as soon as possible and work with you to address any security issues.

## Scope

Relay Support Agents is a multi-agent support system for a fictional store. It runs on **synthetic data**, and refunds, cancellations and store credits are **simulated records**: no real money or real customer data is involved. The design still treats them as real. The security principles, threat model and OWASP mappings are in [docs/design.md §14](docs/design/07-security-and-privacy.md#14-security-the-core-of-this-project); the full threat model will be in `docs/threat-model.md` (planned in [#50](https://github.com/dileepadev/relay-support-agents/issues/50)).

**In scope:**

- Unauthorized money movement: a refund, cancellation, address change or store credit that breaks the store policy (R1–R12) or skips a required approval
- Cross-account access (BOLA / confused deputy): seeing or acting on another customer's orders or data
- Privilege escalation between agents: a read-only agent (Router, Support, Escalation) getting a write tool to run, or free text reaching the Actions agent past the handoff guard
- Approval bypass or reuse: forging, replaying or widening an approval token, or executing an action after its approval was rejected or expired
- Secrets reaching the model: one-time codes, tokens or keys appearing in model context, logs or traces
- Data exfiltration through agent output (Markdown images, links outside the allowlist) or through escalation tickets
- Tampering with the audit log without `make verify-audit` detecting it
- Bypassing the kill switches, budgets or rate limits (unbounded consumption)
- MCP server weaknesses: token audience or expiry not checked, token pass-through, missing scope checks, tool-definition changes that pinning doesn't catch
- Unsafe rendering of model output in the chat or the approvals console (e.g., XSS)
- Exposed secrets, CI/CD, OIDC, Terraform or supply-chain weaknesses (dependencies, container images, GitHub Actions)

**Out of scope (please use a public issue instead):**

- Wrong answers, misstated policies or bad handoffs with no security impact: use the **Agent Behavior** issue template
- Jailbreaks that only make an agent produce off-topic text, with no action, data exposure or policy bypass
- Extracting the system prompts: they contain no secrets or authorization logic by design
- Findings that need denial-of-service or load testing against the live demo

Please don't run automated scanners or load tests against the live demo, and don't include real personal data in your reports.

## Bug Bounty Program

At this time, we do not offer a bug bounty program. If we decide to introduce one in the future, details will be shared here.

## Maintainer Responsibilities

Maintainers are responsible for the security of the project. This includes:

- Responding to security reports in a timely manner
- Investigating reported vulnerabilities
- Developing and releasing patches for confirmed vulnerabilities, fixing them in code first and in prompts last
- Adding a regression case to the red-team suite for every confirmed agent or prompt-injection report
- Communicating with the reporting party as the issue is addressed
- Regularly reviewing and auditing the security of the project

## Disclosures

- We will acknowledge receipt of your report within one business day as soon as possible.
- We will confirm the vulnerability and determine its impact.
- We will release a fix as soon as possible, depending on the complexity of the issue.
- We will communicate the vulnerability and any patches or workarounds to our users.

Thank you for your help in making the [Relay Support Agents](https://github.com/dileepadev/relay-support-agents) project a more secure place.
