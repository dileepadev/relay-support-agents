# Pull Request

<!--
about: Template for Pull Requests
title: "<type>(<scope>): <message> (<issue references>)"
# Example: "feat(handoff): Add the ActionRequest handoff guard (fixes #26)"
-->

## 📜 Description

<!-- [REQUIRED] Briefly describe the changes in this PR and the reasoning behind them. -->

## ❗ Related Issues

<!-- [REQUIRED] List related issue(s) or PR(s), e.g., "Closes #26" or "Related to #36". One issue per PR where possible. -->

Closes #

## ⚙️ Type of Change

<!-- [REQUIRED] Check the type of change your PR introduces -->

- [ ] 🌱 New Feature / Agent Behavior (`feat`)
- [ ] 🐞 Bug Fix (`fix`)
- [ ] 📊 Evaluation: runner, simulator, grading, judge, red-team, thresholds, ablation or report (`eval`)
- [ ] 📝 Documentation / ADR (`docs`)
- [ ] ♻️ Refactor / Style / Performance (`refactor`, `style`, `perf`)
- [ ] 🧪 Tests (`test`)
- [ ] ⚙️ CI / Infrastructure (`ci`)
- [ ] 🧹 Chore / Dependencies / Release (`chore`)
- [ ] 💡 Other (please explain in Description)

## 🔒 Security-Sensitive Paths

<!-- [REQUIRED] Pick one. The paths are listed in .github/CODEOWNERS and AGENTS.md. -->

- [ ] **Not touched.**
- [ ] **Touched:** <!-- list the paths --> and reviewed line by line by the maintainer.

## 📊 Eval Impact

<!-- [REQUIRED] Pick one. -->

- [ ] **No eval impact.** Reason: <!-- e.g., docs only; tooling; console styling -->
- [ ] **Measured.** Results below, or see the CI smoke-eval comment.

<!--
Fill in for changes to agents, prompts, tools, guards, policies, configs or eval code.
Use the dev split only. Never tune on test.

Experiment: E? · Scenarios: scenarios_v? (dev, N, k = ?) · Attacks: redteam_v? (dev, N) · Agent mode: multi | single
Models (router / support / actions / escalation): <model IDs> · Simulator: <model ID> · Judge: <model ID> · Policy: <policy_version>

| Metric                         | main | this PR | Δ |
| ------------------------------ | ---- | ------- | - |
| Task success (pass^1)          |      |         |   |
| Reliability (pass^4, full runs)|      |         |   |
| Critical policy violations     |      |         |   |
| Critical attack success rate   |      |         |   |
| Overall attack success rate    |      |         |   |
| Routing correctness            |      |         |   |
| Handoff accuracy               |      |         |   |
| Invalid handoffs executed      |      |         |   |
| Escalation accuracy            |      |         |   |
| Communication quality          |      |         |   |
| Cost per resolved conversation |      |         |   |
| Agent turn latency p95         |      |         |   |

Eval run cost (agents / simulator / judge):
-->

## 📸 Screenshots

<!-- [OPTIONAL] For chat or console changes, include before/after screenshots. For report changes, include the chart or trace. Remove if not applicable. -->

## ✅ Checklist

<!-- [REQUIRED] Tick all that apply -->

<!-- Use "x" to check. -->
<!-- example: - [x] I've followed the [CONTRIBUTING.md](https://github.com/dileepadev/relay-support-agents/blob/main/CONTRIBUTING.md). -->

- [ ] I've followed the [CONTRIBUTING.md](https://github.com/dileepadev/relay-support-agents/blob/main/CONTRIBUTING.md).
- [ ] I've followed the [Commit Message Guidelines](https://github.com/dileepadev/relay-support-agents/blob/main/COMMIT_MESSAGE_GUIDELINES.md).
- [ ] I've followed the [Branch Naming Guidelines](https://github.com/dileepadev/relay-support-agents/blob/main/BRANCH_NAMING_GUIDELINES.md).
- [ ] I agree to follow the [Pull Request Guidelines](https://github.com/dileepadev/relay-support-agents/blob/main/PULL_REQUEST_GUIDELINES.md).
- [ ] I agree to follow the [Code of Conduct](https://github.com/dileepadev/relay-support-agents/blob/main/CODE_OF_CONDUCT.md).
- [ ] `make lint test` passes locally, and `make eval-smoke` passes if `agents/`, `mcp_server/` or `policies/` changed.
- [ ] No tool argument identifies a customer; identity comes from the token only.
- [ ] Only `agents/actions/` holds `actions:write`; the Router, Support and Escalation agents stay read-only.
- [ ] No free-text field is added to `ActionRequest`; any contract change updates the handoff guard, its tests and the schema snapshot.
- [ ] No policy rule is moved into a prompt.
- [ ] New tools have a strict schema, a "use when / don't use when" description, an owning agent and scope, an audit record, a contract snapshot update, at least 2 scenarios and 1 attack case.
- [ ] No model IDs, prices or secrets are hardcoded outside config and `.env`.
- [ ] No human-labeled data (`evals/scenarios/`, `evals/redteam/`, `evals/judges/calibration/`) is changed, or this PR only contains labels written by hand by the maintainer.
- [ ] `evals/thresholds.yaml` is not weakened, or an ADR explains why.
- [ ] Significant design decisions are recorded as an ADR in `docs/adr/`.
- [ ] Documentation and [CHANGELOG.md](https://github.com/dileepadev/relay-support-agents/blob/main/CHANGELOG.md) have been updated where necessary.
- [ ] Synthetic data only: no real personal data in code, fixtures, prompts, scenarios or logs.
- [ ] If a coding agent helped, I can explain every line, and notable sessions are logged in `docs/build-log.md`.

<!-- example:
- [ ] I've tested the web app changes where applicable:
  - [ ] Screen sizes (mobile/tablet/desktop)
  - [ ] Browsers (Chrome/Firefox/Safari/etc.)
  - [ ] Keyboard navigation and screen-reader labels
  - [ ] Theme modes (Dark mode / Light mode)
-->

## 💬 Additional Comments

<!-- [OPTIONAL] Any additional context, edge cases, considerations, or explanations for reviewers. Remove if not needed.  -->
