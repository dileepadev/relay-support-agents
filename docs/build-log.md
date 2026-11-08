# Build log

Notable coding-agent sessions: what was asked, what the agent got wrong, and how it was caught. The rules are in [AGENTS.md](../AGENTS.md).

Entry template:

```markdown
## YYYY-MM-DD: [task]

- Asked: ...
- Agent got wrong: ...
- Caught by: ... (a test? an eval drop? review?)
- Verified: ...
- Lesson: ...
```

---

## 2026-11-09: Project docs, community standards and GitHub issues

- Asked: finalize the docs like verity-rag (a design index with section files, verity-style agent instructions), add the community standards files with no git operations, rename the project to Relay Support Agents, review everything, then create the GitHub issues (one epic → phases → tasks as sub-issues).
- Agent got wrong:
  - The first lint run used an older markdownlint build that has no MD060 (table column style), so it would have missed the table problem verity-rag already had to fix. The first `VERSIONING.md` release table then failed MD060.
  - After the career sections (why this project gets interviews, the CV kit, interview talking points) were removed from the public design, career wording was still left in the body text ("Interviewers probe the harness", "on your CV", "the most-requested framework in postings").
- Caught by: markdownlint (current release); a grep sweep for career wording; a link-check script for relative links and anchors.
- Verified: markdownlint reports 0 issues; every relative link and anchor resolves; 61 issues created with the planned numbers (#1–#61), and the sub-issue counts per phase match design §22 (6, 10, 22, 13, 3).
- The final review also caught rules that contradicted the design: the E0 baseline's `customer_id` tools vs the AGENTS.md rule, and the single-agent baseline's `actions:write` vs "only `agents/actions/`". Both are now explicit, eval-only exceptions.
- Also fixed in the design while splitting it: `agent/` vs `agents/` (§24); `docs/security.md` vs `docs/threat-model.md`; a `prompts/` folder that doesn't exist (prompts live in each agent's package); no dev/test split for scenarios and attacks even though the standards require one (§12.1); no API endpoint or SSE event for the secure one-time-code input (§10); the audit-log role rules disagreed between §8.8 and §9; tables used in the design but missing from §9; `docs/error-analysis.md` missing from the repository tree (§20).
- Lesson: pin the same linter version in the editor, the CLI and CI; after removing a section, search the rest of the docs for the ideas it carried, not just its heading.
