# Branch Naming Guidelines

When creating a new branch, please follow this naming convention to maintain consistency and clarity in communication regarding the nature of changes.

## Format

```md
<type>/<issue-number>-<short-slug>
```

- `<type>`: one of the types below (the same types as the [Commit Message Guidelines](COMMIT_MESSAGE_GUIDELINES.md)).
- `<issue-number>`: the GitHub issue the branch works on. Every task in Relay Support Agents starts from an issue.
- `<short-slug>`: two to four lowercase words separated by hyphens.

## Protected Branches

These branches are protected and can only be updated through pull requests. As a contributor, do **not** commit directly to these branches.

| Branch Name | Description |
| --- | --- |
| `main` | The stable branch. CI and the eval gate must pass before merging. Releases are tagged from `main`, and merging deploys through the Terraform and deploy workflows. |

## Contributing Branches

These branches are open for contributions from all contributors. Follow these naming conventions when creating a new branch:

| Branch Name | Description |
| --- | --- |
| `feat/<n>-x` | New features or agent behavior (domain, policy engine, MCP tools, agents, guards, API, web app). |
| `fix/<n>-x` | Bug fixes. Replace `x` with a description of the issue being addressed. |
| `eval/<n>-x` | Scenario runner, simulator, grading, judge, red-team runner, thresholds, ablations or reports. |
| `docs/<n>-x` | Documentation updates, ADRs and design docs. |
| `style/<n>-x` | Code style changes (formatting, spacing, etc.). |
| `refactor/<n>-x` | Code refactoring that doesn't change behavior or eval results. |
| `perf/<n>-x` | Latency, cost or memory improvements. |
| `test/<n>-x` | Adding or modifying tests. |
| `ci/<n>-x` | GitHub Actions workflows, the CI eval gate and deploy pipelines. |
| `chore/<n>-x` | Routine tasks or maintenance (e.g., tooling, release preparation). |

Branches created by Dependabot (`dependabot/...`) keep their generated names.

## Examples

Here are a few examples of valid branch names:

- `docs/3-design-docs` - Add the design docs and agent instructions.
- `chore/5-scaffold-repo` - Scaffold the Python project and tooling.
- `feat/12-policy-engine` - Build the policy engine with unit and property tests.
- `feat/16-mcp-write-tools` - Build the MCP write tools with idempotency and approval tokens.
- `eval/19-e0-baseline` - Build the naive E0 baseline and its report.
- `feat/26-handoff-guard` - Add the handoff contracts, the handoff guard and the confirmation card.
- `ci/35-eval-gate` - Add the CI eval gate.
- `fix/90-double-refund` - Return the original refund when a request is retried.
