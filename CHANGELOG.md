# Changelog

All notable changes to this project are documented in this file.

Changes are organized into the following categories:

- **Added:** New features or functionality introduced to the project.
- **Changed:** Modifications to existing functionality that do not add new features.
- **Fixed:** Bug fixes that resolve issues or correct unintended behavior.
- **Removed:** Features or components that have been removed from the project.

Every release also includes an **Eval results** table measured on the test split. See [VERSIONING.md](VERSIONING.md) for the release process.

## [Unreleased]

Unreleased changes go here.

## [v0.1.0] - Work in progress

### Added - v0.1.0

- Design docs: [docs/design.md](docs/design.md) (index) with sections in [docs/design/](docs/design/), and [docs/engineering-standards.md](docs/engineering-standards.md) (refs #3)
- Agent instructions: [AGENTS.md](AGENTS.md) and [CLAUDE.md](CLAUDE.md) (refs #3)
- Initial [README.md](README.md) (refs #3)
- Build log and ADR template: [docs/build-log.md](docs/build-log.md), [docs/adr/](docs/adr/) (refs #3)
- Community standards (refs #4):
  - [.github/ISSUE_TEMPLATE/](./.github/ISSUE_TEMPLATE) - Issue templates, including agent behavior reports
  - [.github/PULL_REQUEST_TEMPLATE.md](./.github/PULL_REQUEST_TEMPLATE.md) - Pull request template with an **Eval impact** section
  - [.github/CODEOWNERS](./.github/CODEOWNERS) - Security-sensitive paths
  - [BRANCH_NAMING_GUIDELINES.md](./BRANCH_NAMING_GUIDELINES.md), [COMMIT_MESSAGE_GUIDELINES.md](./COMMIT_MESSAGE_GUIDELINES.md), [PULL_REQUEST_GUIDELINES.md](./PULL_REQUEST_GUIDELINES.md)
  - [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md), [CONTRIBUTING.md](./CONTRIBUTING.md), [SECURITY.md](./SECURITY.md)
  - [CHANGELOG.md](./CHANGELOG.md), [VERSIONING.md](./VERSIONING.md), [TODO.md](./TODO.md), [LICENSE](./LICENSE)
- Markdown lint configuration: [.markdownlint.jsonc](./.markdownlint.jsonc), and VS Code settings so `CODEOWNERS` isn't linted as Markdown: [.vscode/settings.json](./.vscode/settings.json) (refs #3)

<!-- Release entry template:

## [v0.1.0] - YYYY-MM-DD

### Added - v0.1.0

- ...

### Eval results - v0.1.0

Scenarios: scenarios_vN (test split, N scenarios, k = 4) · Attacks: redteam_vN (test split, N attacks) · Agent mode: multi · Models: router / support / actions / escalation model IDs · Simulator: model ID · Judge: model ID · Policy: 2026-11-01 · Eval run: <date>-<git SHA>

| Metric                          | Multi-agent | Single-agent | v1 target |
| ------------------------------- | ----------- | ------------ | --------- |
| Task success (pass^1)           |             |              | ≥ 0.85    |
| Reliability (pass^4)            |             |              | ≥ 0.70    |
| Critical policy violations      |             |              | 0         |
| Critical attack success rate    |             |              | 0%        |
| Overall attack success rate     |             |              | ≤ 5%      |
| Routing correctness             |             |              | ≥ 0.95    |
| Escalation accuracy             |             |              | ≥ 0.90    |
| Handoff accuracy                |             |              | ≥ 0.95    |
| Invalid handoffs executed       |             |              | 0         |
| Communication quality           |             |              | ≥ 0.90    |
| Cost per resolved conversation  |             |              | report    |
| Agent turn latency p95          |             |              | ≤ 8 s     |
-->

<!-- e.g., -->
<!-- Unreleased -->
<!-- v1.0.0 -->
<!-- v0.5.0 -->
<!-- v0.1.0 -->

[Unreleased]: https://github.com/dileepadev/relay-support-agents/commits/main
[v0.1.0]: https://github.com/dileepadev/relay-support-agents/commits/v0.1.0
