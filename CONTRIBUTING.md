# Contributing to Relay Support Agents

Thank you for your interest in contributing to [Relay Support Agents](https://github.com/dileepadev/relay-support-agents)! We welcome contributions, including bug fixes, agent and evaluation improvements, new scenario and attack ideas, documentation improvements, and reports of wrong actions or misstated policies.

Relay Support Agents is safety-first and evaluation-driven: authorization and policy live in code, not prompts, and every change to agents, tools, prompts or policies has to show its effect on the scenario and red-team suites. Read [docs/design.md](docs/design.md) and [docs/engineering-standards.md](docs/engineering-standards.md) before starting a larger change.

> [!Note]
> Relay Support Agents is in early development. The Python tooling (`uv`, `make` targets, CI) is being set up in [#5](https://github.com/dileepadev/relay-support-agents/issues/5); until it lands, some commands below won't work yet.

## Getting Started

1. **Find or open an issue**  
   Every task starts from a GitHub issue with acceptance criteria. Check the [development plan (#1)](https://github.com/dileepadev/relay-support-agents/issues/1) for open tasks, or open a new issue using one of the [issue templates](.github/ISSUE_TEMPLATE). Comment on the issue before starting larger work.

2. **Fork the repository**  
   Fork this repository to your GitHub account. This creates a copy of the repository in your account, allowing you to make changes without affecting the original repository.  
   To fork the repository, click the **Fork** button in the top right corner of this page or click [here to fork the repository](https://github.com/dileepadev/relay-support-agents/fork).

3. **Clone your fork and set up**  
   Clone your forked repository and install the development environment (Python 3.12, [uv](https://docs.astral.sh/uv/), Node.js for the web app, Docker):

   ```bash
   git clone https://github.com/<your-username>/relay-support-agents.git
   cd relay-support-agents
   cp .env.example .env   # add your own API keys; never commit .env
   make setup             # install dependencies and pre-commit hooks
   make up                # start Postgres (and the API, MCP server and web app)
   make seed              # build the synthetic store database
   ```

4. **Create a new branch**  
   Create a new branch for your changes. Follow the [branch naming guidelines](BRANCH_NAMING_GUIDELINES.md).

   ```bash
   git checkout -b feat/26-handoff-guard
   ```

5. **Make changes, test and commit**  
   Run the checks before every commit, then commit with a descriptive message. Follow the [commit message guidelines](COMMIT_MESSAGE_GUIDELINES.md).

   ```bash
   make lint test
   git commit -m "feat(handoff): Add the ActionRequest handoff guard (refs #26)"
   ```

6. **Measure the effect**  
   If you changed agents, prompts, tools, guards, policies or eval code, run the smoke suite (dev scenarios plus the critical attacks) and compare it with `main`:

   ```bash
   make eval-smoke
   make eval-agent        # fuller runs, when the change needs them
   ```

7. **Push your changes**  
   Push your changes to your forked repository.

   ```bash
   git push origin feat/26-handoff-guard
   ```

8. **Submit a pull request**  
   To submit a pull request:
   - Go to your forked repository.
   - Click the **Compare & pull request** button next to your branch.
   - Add a title and description for your pull request. Follow the [pull request guidelines](PULL_REQUEST_GUIDELINES.md).
   - Fill in the [pull request template](.github/PULL_REQUEST_TEMPLATE.md), including the **Eval impact** section, then click **Create pull request**.

## Guidelines

- **Code style:** typed Python 3.12 (mypy strict on core packages), ruff, small functions, no global mutable state, structured logging instead of `print()`. TypeScript for the web app.
- **Security rules** (the full list is in [AGENTS.md](AGENTS.md)):
  - no tool argument may identify a customer: identity comes from the signed token only;
  - only the Actions agent (`agents/actions/`) may hold the `actions:write` scope;
  - `ActionRequest` never gets a free-text field, and free text never reaches the Actions agent;
  - policy rules live in `policies/*.yaml` and `domain/policy.py`, never in a prompt.
- **Security-sensitive paths** are listed in [.github/CODEOWNERS](.github/CODEOWNERS). Changes there get a line-by-line review by the maintainer.
- **New tools** need a strict schema, a description with "use when / don't use when", an owning agent and scope, an audit record, a contract snapshot update, at least 2 scenarios and 1 attack case.
- **Configuration:** model IDs, prices, budgets and limits come from config and `.env`. Never hardcode them, and never commit secrets.
- **Tests:** unit tests run without network access; integration tests use recorded LLM responses. Keep coverage at 80% or more on core logic and 100% branch coverage on the policy engine.
- **Evaluation:** tune on the dev split only; never tune on test. Don't weaken `evals/thresholds.yaml` without an ADR. Any critical attack success fails CI.
- **Human-labeled data:** files in `evals/scenarios/`, `evals/redteam/` and `evals/judges/calibration/` are written by hand by the maintainer. Don't edit them in a PR. To suggest a scenario or an attack, open an issue that describes it (for a successful attack, follow [SECURITY.md](SECURITY.md) instead).
- **Data:** synthetic data only. Don't put real personal data in issues, fixtures, prompts, scenarios or eval examples.
- **Decisions:** record significant design decisions as an ADR in `docs/adr/` (MADR format).
- **Coding agents:** you may use them, but you must be able to explain every line you submit. Agents follow [AGENTS.md](AGENTS.md).
- Update documentation and [CHANGELOG.md](CHANGELOG.md) where necessary, and keep your pull request focused on one issue.
- Refer to the following templates and guidelines before submitting your changes:
  - [relay-support-agents/](./) - Root directory of the repository
    - [.github/](./.github) - GitHub-specific files (workflows, templates, CODEOWNERS)
      - [ISSUE_TEMPLATE/](./.github/ISSUE_TEMPLATE) - Contains all issue templates
        - [agent_behavior.md](./.github/ISSUE_TEMPLATE/agent_behavior.md) - Template for reporting wrong actions, misstated policies or bad handoffs
        - [bug_report.md](./.github/ISSUE_TEMPLATE/bug_report.md) - Template for reporting bugs
        - [documentation_update.md](./.github/ISSUE_TEMPLATE/documentation_update.md) - Template for documentation updates
        - [feature_request.md](./.github/ISSUE_TEMPLATE/feature_request.md) - Template for suggesting new features
        - [feedback.md](./.github/ISSUE_TEMPLATE/feedback.md) - Template for general feedback
        - [other.md](./.github/ISSUE_TEMPLATE/other.md) - Template for other types of issues
      - [CODEOWNERS](./.github/CODEOWNERS) - Security-sensitive paths that need the maintainer's review
      - [PULL_REQUEST_TEMPLATE.md](./.github/PULL_REQUEST_TEMPLATE.md) - Template for pull request submissions
    - [AGENTS.md](./AGENTS.md) - Rules and workflow for coding agents
    - [BRANCH_NAMING_GUIDELINES.md](./BRANCH_NAMING_GUIDELINES.md) - Branch naming rules
    - [CHANGELOG.md](./CHANGELOG.md) - Record of project changes
    - [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) - Contributor behavior guidelines
    - [COMMIT_MESSAGE_GUIDELINES.md](./COMMIT_MESSAGE_GUIDELINES.md) - Rules for writing commit messages
    - [CONTRIBUTING.md](./CONTRIBUTING.md) - How to contribute to the project
    - [LICENSE](./LICENSE) - Project license
    - [PULL_REQUEST_GUIDELINES.md](./PULL_REQUEST_GUIDELINES.md) - Pull request submission guidelines
    - [README.md](./README.md) - Project overview
    - [SECURITY.md](./SECURITY.md) - Security policy and reporting
    - [TODO.md](./TODO.md) - Where planned work is tracked
    - [VERSIONING.md](./VERSIONING.md) - Versioning strategy for the project
    - [docs/](./docs) - Design, engineering standards, ADRs and other project docs

## Code of Conduct

This project adheres to the **Contributor Covenant Code of Conduct**. By participating, you agree to abide by its terms.  
Read the full Contributor Covenant Code of Conduct in the [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) file.

## Contact

If you have any questions or suggestions regarding Relay Support Agents, feel free to open an issue or submit a pull request in this repository.

You can also reach me via email at: **<contact@dileepa.dev>**
