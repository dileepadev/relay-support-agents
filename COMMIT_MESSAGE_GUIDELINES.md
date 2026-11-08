# Commit Message Guidelines

A good commit message should be descriptive and provide context about the changes made. This helps improve code review, debugging, and long-term maintainability. Relay Support Agents uses [Conventional Commits](https://www.conventionalcommits.org) with the conventions below.

Table of Contents:

- [Commit Message Guidelines](#commit-message-guidelines)
  - [Commit Message Format](#commit-message-format)
    - [`<type>`](#type)
    - [`<scope>`](#scope)
    - [`<short message>`](#short-message)
    - [`<optional issue references>`](#optional-issue-references)
    - [`<optional PR reference>`](#optional-pr-reference)
    - [`<optional longer description>`](#optional-longer-description)
  - [Issue and PR References](#issue-and-pr-references)
    - [Best Practices](#best-practices)
    - [Examples for Issue and PR References](#examples-for-issue-and-pr-references)
    - [Summary Table for Issue and PR Linking (with Scope)](#summary-table-for-issue-and-pr-linking-with-scope)
  - [Squash Merges](#squash-merges)
  - [✅ Summary Examples with All Fields](#-summary-examples-with-all-fields)

## Commit Message Format

Use the following format for commit messages:

```md
<type>(<scope>): <short message> (<optional issue references>) (<optional PR reference>)

<optional longer description>
```

### `<type>`

The `<type>` field indicates the nature of the changes made in the commit. Use one of the following values:

| Type | Description |
| --- | --- |
| feat | A new feature or agent behavior (domain, policy engine, MCP tools, agents, guards, API, web app). |
| fix | A bug fix or correction of an issue. |
| eval | Scenario runner, simulator, grading, judge prompts, red-team runner, thresholds, ablations or eval reports. |
| docs | Documentation updates (e.g., README, ADRs, design docs, code comments). |
| style | Code style changes (e.g., formatting, indentation, no logic). |
| refactor | Code refactoring without changing external behavior. |
| perf | Performance improvements (latency, cost, memory). |
| test | Adding or modifying tests. |
| ci | GitHub Actions workflows, CI eval gate, deploy pipelines. |
| chore | Routine tasks, maintenance, dependencies, tooling or release commits. |

> [!Note]
> Scenarios (`evals/scenarios/`), red-team attacks (`evals/redteam/`) and judge-calibration labels (`evals/judges/calibration/`) are written by hand by the maintainer. Commit them as `eval(scenarios): ...`, `eval(redteam): ...` or `eval(evals): ...`, and never mix them with code changes.

### `<scope>`

The `<scope>` is optional, but recommended. It names the part of Relay Support Agents the commit affects. Scopes follow the repository layout:

| Scope | Description |
| --- | --- |
| repo | Project-wide configuration or setup |
| config | Model IDs, prices, budgets, limits, environment |
| deps | Dependency updates (`pyproject.toml`, `uv.lock`, `web/package.json`) |
| policy | `policies/` and the policy engine (`domain/policy.py`) |
| domain | Domain models, services, idempotency and the audit log |
| db | Data model, migrations, roles and the seed generator |
| mcp | MCP server, tools, token auth and rate limits |
| agents | Orchestrator, `CaseState`, budgets, the four agents, their prompts and the single-agent baseline |
| handoff | Handoff contracts, the handoff guard and the confirmation card |
| guards | Input, tool and output guards and spotlighting |
| llm | Model-agnostic LLM interface, pricing and usage tracking |
| api | FastAPI service, SSE, approvals, admin and privacy endpoints |
| web | Next.js customer chat and staff console |
| evals | Scenario runner, simulator, grading, judge, red-team runner, thresholds, reports |
| scenarios | Scenario suites (human-written only) |
| redteam | Red-team attack suites (human-written only) |
| observability | Tracing, metrics, PII redaction |
| infra | Docker, Terraform, AWS (ECS, RDS, ALB, Secrets Manager), Vercel |
| security | Threat model, scans, security controls |
| adr | Architecture decision records in `docs/adr/` |
| release | Version bumps and release commits |

### `<short message>`

A concise summary of what the commit does.

Writing Tips:

- Use the imperative mood (e.g., “Add” not “Added”)
- Capitalize the first letter
- Don’t end with a period
- Keep under 50 characters if possible
- Be specific (e.g., “Add the handoff guard” > “Update agents”)
- Avoid "I", "this commit", or "fix" as vague verbs

### `<optional issue references>`

Optionally link the commit to relevant issues using GitHub keywords such as `fixes`, `closes`, `resolves`, or `refs`. Multiple issues can be referenced by separating them with commas. Every task in Relay Support Agents has a GitHub issue, so most commits should carry at least a `refs #<issue>`.

### `<optional PR reference>`

Optionally link the commit to a pull request using the format `(#<number>)`. This should be placed at the end of the commit message.

### `<optional longer description>`

An optional detailed description of the changes made in the commit. This can include the reasoning behind the changes, implementation details, or any other relevant information. Separate this section from the header with a blank line.

For changes that affect reliability or safety, mention the measured effect (e.g., "pass^1 on dev: 0.78 → 0.84; critical ASR stays 0"), and point to the eval report.

## Issue and PR References

Use GitHub keywords to link commits to issues, and standard notation for PRs.

### Best Practices

- **Pull Requests**: Use `(#<number>)` at the very end of the header. This is automatically recognized by GitHub.
  - ✅ `(#5)`
  - 🚫 `(pr #5)` or `refs #5` for the PR itself.
- **Issues**: Use standard GitHub keywords (`fixes`, `refs`, `closes`, `resolves`) in parentheses before the PR number.
  - 🚫 Don't write a bare `(#26)` for an issue: that is the PR reference slot.

### Examples for Issue and PR References

**Straightforward PR reference:**

```md
chore(deps): Bump langgraph to 1.x (#41)
```

**Issue + PR:**

```md
feat(handoff): Add the ActionRequest handoff guard (fixes #26) (#52)
```

**Multiple Issues + PR:**

```md
eval(evals): Add E1 to E4 ablation rows (refs #38, #36) (#60)
```

### Summary Table for Issue and PR Linking (with Scope)

| Purpose | Example Commit Message | Result |
| --- | --- | --- |
| Close single issue | `feat(handoff): Add the ActionRequest handoff guard (fixes #26) (#52)` | Closes #26, links PR #52 |
| Reference single issue | `docs(adr): Record the approval token decision (refs #16) (#44)` | Links issue #16, PR #44 |
| Reference PR Only | `chore(deps): Bump langgraph to 1.x (#41)` | Links PR #41 |
| Close multiple issues | `eval(evals): Add the runner and the E0 report (fixes #17, #19) (#40)` | Closes #17, #19, links PR #40 |
| Mixed References | `fix(mcp): Return the original refund on retry (fixes #90, refs #16) (#91)` | Closes #90, refs #16, links PR #91 |
| Multiple PRs (rare) | `chore(repo): Sync templates (#5, #6)` | Links PR #5 and #6 |

## Squash Merges

Pull requests are squash-merged into `main`. GitHub uses the PR title as the commit header and appends `(#<PR number>)` automatically, so write the PR title as `<type>(<scope>): <short message> (<issue references>)` and the final commit matches this guide. See [PULL_REQUEST_GUIDELINES.md](PULL_REQUEST_GUIDELINES.md).

## ✅ Summary Examples with All Fields

```md
feat(handoff): Add the ActionRequest handoff guard (fixes #26) (#52)

Rejects unknown fields, unverified order IDs and amounts above the last
eligibility result. Smoke on dev: invalid handoffs executed 3 → 0;
pass^1 0.78 → 0.80. Report in evals/reports/.
```

```md
fix(mcp): Return the original refund on a retried request (fixes #90) (#91)
```

```md
eval(scenarios): Add scenario suite v0 with datasheet (fixes #18) (#38)
```

```md
chore(release): Release v0.1.0 (refs #9) (#45)
```
