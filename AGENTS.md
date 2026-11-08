# Relay Support Agents: agent instructions

Before starting a task, read [`docs/engineering-standards.md`](docs/engineering-standards.md) and the design index [`docs/design.md`](docs/design.md), then the section files in [`docs/design/`](docs/design/) that the issue references (issues cite sections as `docs/design.md §N`; the index maps each § to its file). The work order is in the GitHub issues: phases under #1, `priority: must` first ([design §22](docs/design/09-plan-and-risks.md#22-implementation-plan-phases-and-milestones)).

## Rules

- Python 3.12, uv. Run `make lint test` before every commit; also run `make eval-smoke` when touching `agents/` (including the per-agent prompts), `mcp_server/` or `policies/`.
- SECURITY-SENSITIVE (human review required, see [`.github/CODEOWNERS`](.github/CODEOWNERS)): `domain/policy.py`, `mcp_server/auth.py`, `agents/guards/`, `agents/handoff.py`, `domain/audit.py` (all under `src/relay_support_agents/`) and `infra/terraform/`.
- Never add a tool argument that identifies a customer. Identity comes from the token only. The only exception is the deliberately unsafe E0 baseline in `evals/baselines/e0/`, which never ships.
- Never give the Router, Support or Escalation agent a write scope. Only `agents/actions/` may hold `actions:write`, plus the single-agent E11 baseline in `agents/single/`, which runs only in evals (`AGENT_MODE=single`) and never in the deployed demo.
- Never add a free-text field to `ActionRequest` or pass free text into the Actions agent. Use IDs, enums, amounts and opaque references; update the handoff guard and its tests with any contract change.
- Never move a policy rule into a prompt. Policies live in `policies/*.yaml` + `domain/policy.py`.
- Never edit `evals/scenarios/`, `evals/redteam/` or `evals/judges/calibration/` without being asked.
- Never weaken `evals/thresholds.yaml` without an ADR.
- Every new tool needs: strict schema, description with "use when / don't use when", an owning agent and scope, audit record, contract snapshot update, at least 2 scenarios and 1 attack case.
- All model IDs and prices come from config; never hardcode them.
- Synthetic data only. Never put real personal data in code, fixtures, prompts, scenarios, logs or traces.
- Keep functions small and typed; no global state; no `print()`, use the logger.
- Keep it simple: the scope is 6 intents, 12 tools and 4 agents ([design §24](docs/design/09-plan-and-risks.md#24-risks-and-mitigations)). Don't add services, frameworks or dependencies that the issue doesn't ask for.
- Every Markdown file must pass [markdownlint](https://github.com/DavidAnson/markdownlint) with no warnings, using the repo config `.markdownlint.jsonc` (default rules, with MD013 line length off, as in the editor). Fix the content instead of disabling rules. Write placeholders as `[placeholder]` or in backticks, never as bare `<placeholder>` (MD033).
- The only exception is a one-line file that can't be fixed, such as `CLAUDE.md` (just `@AGENTS.md`): it may use `<!-- markdownlint-disable-next-line <rule> -->` for that one rule. Never disable a rule for a whole file or folder, and don't turn off more rules in `.markdownlint.jsonc`.

## Workflow for every task

1. A GitHub issue with acceptance criteria.
2. Propose a plan first; wait for approval.
3. Implement on a branch.
4. The human reviews the diff, runs tests and evals, then fixes or rejects.
5. Small PRs only, using Conventional Commits. Follow `BRANCH_NAMING_GUIDELINES.md`, `COMMIT_MESSAGE_GUIDELINES.md` and `PULL_REQUEST_GUIDELINES.md`, and fill in the **Eval impact** section of the PR template.

## Commits and pushes

Never commit or push without the maintainer's explicit approval. Before every commit, stage the files, then show:

1. the full commit message;
2. the files being committed (`git diff --staged --stat`);
3. the lines being committed (`git diff --staged`). If the diff is too long to read in chat, show the per-file line counts and say how to view the full diff;
4. whether it will be pushed, and to which remote and branch.

Then wait. Approval covers only that commit and push; ask again for the next one.

## Never delegated to an agent

- scenario, red-team and judge-calibration labels;
- choosing thresholds and approval limits;
- security-sensitive code without line-by-line review;
- interpreting metrics;
- the final README claims.

## CODEOWNERS

Security-sensitive paths always need explicit human review, even when an agent wrote the change.

## Agent Skills (optional)

A `redteam` skill that runs the attack suite and summarizes attack success by category and stopping layer. Keep the security rules above in this file, never only in a skill.

## Build log

Notable sessions go in `docs/build-log.md`:

```markdown
## YYYY-MM-DD: <task>
- Asked: ...
- Agent got wrong: ...
- Caught by: ... (a test? an eval drop?)
- Verified: ...
- Lesson: ...
```

Never commit code that can't be explained line by line.
