# Versioning

This project follows a versioning pattern similar to [Semantic Versioning](https://semver.org/) (SemVer) for managing releases.

## Table of Contents

- [Versioning](#versioning)
  - [Table of Contents](#table-of-contents)
  - [Format](#format)
    - [Examples](#examples)
  - [Planned Releases](#planned-releases)
  - [What Else Is Versioned](#what-else-is-versioned)
  - [Release Process](#release-process)
    - [Typical Steps](#typical-steps)
  - [Pre-release Versions](#pre-release-versions)
  - [Viewing Tags \& Differences](#viewing-tags--differences)
  - [Questions or Issues?](#questions-or-issues)

## Format

Version numbers follow the structure:

`MAJOR.MINOR.PATCH`

- **MAJOR** – Incompatible changes to the public API (`/v1/...`), the MCP tool contracts, the handoff contracts, the config schema or the CLI
- **MINOR** – Backward-compatible functionality, agent and reliability improvements and new features
- **PATCH** – Backward-compatible bug fixes and small improvements

While Relay Support Agents is below `1.0.0`, any release may contain breaking changes.

### Examples

| Version | Meaning                          |
| ------- | -------------------------------- |
| `0.1.0` | First public release / early dev |
| `0.1.1` | Bug fix                          |
| `0.2.0` | New non‑breaking feature         |
| `1.0.0` | Official stable release          |
| `1.0.1` | Bug fix after release            |
| `1.1.0` | New feature added                |
| `2.0.0` | Major breaking change            |

## Planned Releases

Each release phase of the [development plan](https://github.com/dileepadev/relay-support-agents/issues/1) ends with a tagged release:

| Version  | Milestone            | Phase                                                                                                    |
| -------- | -------------------- | -------------------------------------------------------------------------------------------------------- |
| `v0.1.0` | M1: baseline numbers | [Phase 1 · Domain, tools, baseline](https://github.com/dileepadev/relay-support-agents/issues/9)         |
| `v0.5.0` | M2: safe agents      | [Phase 2 · Agents, reliability and safety](https://github.com/dileepadev/relay-support-agents/issues/20) |
| `v1.0.0` | M3: deployed v1.0    | [Phase 3 · Production](https://github.com/dileepadev/relay-support-agents/issues/43)                     |

## What Else Is Versioned

The software version is separate from the other versioned parts of Relay Support Agents. A release note names the version of each one it was measured with.

| Item               | Scheme                                              | Where                                                      |
| ------------------ | --------------------------------------------------- | ---------------------------------------------------------- |
| Store policy       | `policy_version` (date), e.g. `2026-11-01`          | `policies/` and on every decision and refund               |
| Scenario suite     | `scenarios_v0`, `scenarios_v1`, `scenarios_v2`, ... | `evals/scenarios/` and its datasheet                       |
| Red-team suite     | `redteam_v1`, `redteam_v2`, ...                     | `evals/redteam/` and its datasheet                         |
| Prompts            | Versioned files, reviewed like code                 | Each agent's package in `src/relay_support_agents/agents/` |
| MCP tool contracts | Per-agent tool list snapshots and pinned hashes     | `tests/contract/` and `agents/pinning.py`                  |
| Handoff contracts  | Schema snapshots                                    | `tests/contract/`                                          |
| Experiments        | `E0` ... `E12`                                      | `evals/reports/`                                           |
| Eval runs          | Date + git SHA + config hash + model IDs            | `evals/reports/`                                           |

## Release Process

All notable changes are documented in the [CHANGELOG.md](CHANGELOG.md) file. **Every release note includes eval numbers.**

### Typical Steps

1. Meet the exit criteria of the phase issue (all `priority: must` sub-issues closed)
2. Run the full suite on the **test split** (k = 4) and the full red-team suite with the chosen default models, for the multi-agent design and the single-agent baseline, and keep the report in `evals/reports/`
3. Update the `CHANGELOG.md` with categorized entries:  
   - **Added**, **Changed**, **Fixed**, **Removed**
   - plus an **Eval results** table: metric, multi-agent, single-agent, v1 target
4. Bump the `version` in `pyproject.toml`
5. Open a release PR and squash-merge it (e.g. `chore(release): Release v0.1.0`)
6. Tag the merge commit on `main`:

   ```bash
   git tag v0.1.0
   git push origin v0.1.0
   ```

7. Create a GitHub release and paste the changelog section, including the eval table, the scenario and red-team suite versions, the policy version, the model IDs and the git SHA of the eval run. Attach the SBOM:

   ```bash
   gh release create v0.1.0 --title "v0.1.0" --notes-file <release-notes.md>
   ```

## Pre-release Versions

For beta or release candidates, we use suffixes:

- `1.0.0-beta.1` – Beta release
- `1.0.0-rc.1` – Release candidate

These versions are intended for testing and may not be fully stable.

## Viewing Tags & Differences

List all version tags:

```bash
git tag
```

View differences between versions:

```bash
git log v0.1.0..v0.5.0
```

## Questions or Issues?

If you have questions about the versioning strategy or encounter version-related problems, feel free to open an issue on the [GitHub repository](https://github.com/dileepadev/relay-support-agents/issues).
