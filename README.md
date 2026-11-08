# Relay Support Agents

> A multi-agent customer-support system that can actually do things, like look up orders, issue refunds and cancel orders, without being tricked into doing the wrong things. A router, a support agent, an actions agent and an escalation agent each get only the permissions they need: the agents that read untrusted text can't move money, and the agent that moves money never reads untrusted text. Built on its own MCP tool server, with human approval for risky actions, rules enforced in code, and evals that test reliability, handoffs and attacks against a single-agent baseline.

**Status:** in development.

**Stack:** Python · FastAPI · LangGraph · MCP · PostgreSQL · Next.js · Claude API · AWS (ECS Fargate, RDS) · Terraform · OpenTelemetry · Langfuse. Synthetic data and simulated payments only.

## Docs

| File | What it's for |
| --- | --- |
| [docs/design.md](docs/design.md) | Index of the full design, with each section in its own file under [docs/design/](docs/design/): problem, architecture, components, data model, evaluation, security, privacy, costs, implementation phases ([§22](docs/design/09-plan-and-risks.md#22-implementation-plan-phases-and-milestones)), stretch goals ([§23](docs/design/09-plan-and-risks.md#23-stretch-goals)) |
| [docs/engineering-standards.md](docs/engineering-standards.md) | Repository, code, LLM, eval, observability and security standards, plus the definition of done |
| [AGENTS.md](AGENTS.md) | Rules and workflow for coding agents (also loaded by `CLAUDE.md`) |

## Contributing

Contributions are welcome. Start with [CONTRIBUTING.md](CONTRIBUTING.md), and follow the [branch](BRANCH_NAMING_GUIDELINES.md), [commit](COMMIT_MESSAGE_GUIDELINES.md) and [pull request](PULL_REQUEST_GUIDELINES.md) guidelines. Report a wrong action, a misstated policy or a bad handoff with the **Agent Behavior** issue template, and security issues privately as described in [SECURITY.md](SECURITY.md). Releases follow [VERSIONING.md](VERSIONING.md) and are recorded in [CHANGELOG.md](CHANGELOG.md). This project follows the [Code of Conduct](CODE_OF_CONDUCT.md) and is released under the [MIT License](LICENSE).

## First steps

The plan lives in GitHub issues: start with [#1](https://github.com/dileepadev/relay-support-agents/issues/1) and work through [Phase 0](https://github.com/dileepadev/relay-support-agents/issues/2) in order. Design the rules and permissions before the agents:

1. Write `policies/POLICY.md` (R1–R12), list the 12 tools, and fill in the per-agent permissions table: agent → tools → scopes ([design §4.4](docs/design/01-problem-and-requirements.md#44-store-policy), [§6](docs/design/02-architecture.md#6-solution-overview), [§8.3](docs/design/03-components.md#83-mcp-server-relay-support-agents-mcp); [#6](https://github.com/dileepadev/relay-support-agents/issues/6)).
2. Draft the Postgres schema and seed 5 hand-made customers with edge cases ([design §8.1](docs/design/03-components.md#81-commerce-domain-and-seed-data), [§9](docs/design/04-data-api-stack.md#9-data-model); [#7](https://github.com/dileepadev/relay-support-agents/issues/7)).
3. Write the first 5 scenarios, with expected agent paths, **before** writing any agent ([design §12.1](docs/design/05-evaluation.md#121-scenario-suite), [Appendix A](docs/design/appendices.md#appendix-a-scenario-yaml); [#8](https://github.com/dileepadev/relay-support-agents/issues/8)).

> When the project ships, replace this README with the blueprint in [design §26](docs/design/10-decisions-and-deliverables.md#26-deliverables-readme-demo-video-write-up): results first, then how it works, then setup.
