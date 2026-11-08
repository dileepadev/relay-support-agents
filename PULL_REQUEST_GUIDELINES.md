# Pull Request Guidelines

Pull requests are squash-merged into `main`, so the PR title becomes the commit header. Write it using the [Commit Message Guidelines](COMMIT_MESSAGE_GUIDELINES.md) convention. GitHub appends the PR number when it squash-merges.

## PR Title Format

The pull request title should follow the format:

```md
<type>(<scope>): <message> (<issue references>)
```

- `<type>`: Type of the change. Use one of the following:

  - `feat`: A new feature or agent behavior.
  - `fix`: A bug fix or correction of an issue.
  - `eval`: Scenario runner, simulator, grading, judge, red-team runner, thresholds, ablations or reports.
  - `docs`: Documentation updates (e.g., README, ADRs, comments).
  - `style`: Code style changes (e.g., formatting, indentation).
  - `refactor`: Code refactoring or restructuring without changing functionality.
  - `perf`: Latency, cost or memory improvements.
  - `test`: Adding or modifying tests.
  - `ci`: GitHub Actions workflows, the eval gate, deploy pipelines.
  - `chore`: Routine tasks, maintenance, or tooling changes.

- `<scope>`: The part of Relay Support Agents the change affects (e.g., `policy`, `mcp`, `agents`, `handoff`, `guards`, `api`, `web`, `evals`). The full list is in the [Commit Message Guidelines](COMMIT_MESSAGE_GUIDELINES.md#scope).

- `<message>`: A short, clear, and concise description of the changes, in the imperative mood with a capital first letter.

- `<issue references>`: The issue the PR works on, using a GitHub keyword (`fixes #26`, `refs #26`).

## Examples

- `feat(handoff): Add the ActionRequest handoff guard (fixes #26)`
- `eval(evals): Add the E0 baseline report (fixes #19)`
- `fix(mcp): Return the original refund on a retried request (fixes #90)`
- `docs(adr): Record the approval token decision (refs #16)`

## Rules

- **One issue per PR** where possible. Put `Closes #<issue>` in the PR body.
- **Small PRs.** Split large changes into reviewable steps. Open a draft PR early if you want feedback.
- **Fill in the Eval impact section** of the [pull request template](.github/PULL_REQUEST_TEMPLATE.md). Any change to agents, prompts, tools, guards, policies or eval code needs before/after numbers on the dev split, or the CI smoke-eval comment. Never tune on the test split.
- **CI must pass**, including the eval gate. Any critical attack success or critical policy violation fails the build. Don't weaken `evals/thresholds.yaml` to get a PR through. A threshold change needs an ADR.
- **Security-sensitive paths** ([.github/CODEOWNERS](.github/CODEOWNERS)) get a line-by-line review by the maintainer, even when a coding agent wrote the change.
- **Follow the security rules** in [AGENTS.md](AGENTS.md): no customer-identifying tool arguments, no write scope outside the Actions agent, no free text in `ActionRequest`, no policy rules in prompts.
- **New tools** need a strict schema, a "use when / don't use when" description, an owning agent and scope, an audit record, a contract snapshot update, at least 2 scenarios and 1 attack case.
- **Don't edit human-labeled data** (`evals/scenarios/`, `evals/redteam/`, `evals/judges/calibration/`) in a PR that also changes code.
- **Coding agents are welcome**, but you must be able to explain every line you submit.

Please adhere to this naming convention to maintain consistency and clarity.
