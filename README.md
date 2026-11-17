# dev-lifecycle

An approval-gated development agent for Claude Code. Works with any language and any repository.

Give it a Jira ticket key (e.g. `PROJ-123`) or a plain-text requirement. It will:

| Phase | What happens | Gate |
|-------|--------------|------|
| 0. Branch | **First step:** fetches, finds the latest release branch, creates the working branch from it, named exactly the Jira ticket number (e.g. `PROJ-123`) | asks only if the base is unclear |
| 1. Analyse | Thinks only about the requirement (no solution, no code), writes `.dev-lifecycle/<KEY>/01-analysis.md` | **A** - approve analysis |
| 2. Plan | Only now checks the code and impact on existing flows, writes `02-plan.md` | **B** - approve plan (and any behaviour change) |
| 3. Implement | Creates the branch, writes the code per plan. Your mid-development change prompts are logged verbatim in the plan file, then implemented. | plan deviations need approval |
| 3b. Verify plan | Compares the plan file with the actual changes and confirms every item is implemented and nothing is unplanned | - |
| 4. Test | Writes and runs positive, negative, boundary and regression tests, writes `03-test-report.md` | **C** - you test and confirm |
| 5. Push | Shows diff and commit message (starts with the branch name, no co-author trailer) | **D** - approve commit and push |
| 6. PR | You choose the target branch, it drafts the PR | **E** - approve PR creation |
| 7. Jira | Comments the PR link and results, with a real `@` mention of Indhuja, then moves To Do -> In Progress | - |

## Architecture and security

- The agent follows the project's existing architecture and security controls. It does not bypass layers or weaken checks.
- If it finds a security loophole or violation, it tells you first and asks whether to fix it. It fixes only if you say so (and adds tests for the fix). If you decline, it writes `.dev-lifecycle/<KEY>/04-security-findings.md` with the issue, how to fix it, and which scenarios the fix would break.

## Requirements

- Claude Code
- The **Atlassian MCP server** added to Claude Code (needed only for Jira steps; text requirements work without it)
- `git`, plus `gh` (GitHub), `glab` (GitLab) or `az` (Azure DevOps) for creating the PR

## Install

From a local clone or a Git host:

```bash
/plugin marketplace add <path-or-git-url-of-this-folder>
/plugin install dev-lifecycle@dev-lifecycle-marketplace
```

## Use

Quickest: type the slash command in any Claude Code session (CLI or desktop app):

```
/dev-lifecycle PROJ-123
/dev-lifecycle Add an endpoint that exports orders as CSV
```

If another plugin has a command with the same name, use the full form `/dev-lifecycle:dev-lifecycle`.

Alternative with the best conversation flow - run it as the main session so the approval gates are a normal conversation:

```bash
claude --agent dev-lifecycle:dev-lifecycle
```

Then say: `PROJ-123` or paste the requirement text.

Or delegate from a normal session: "Use the dev-lifecycle agent for PROJ-123". When running as a subagent it stops at each gate with `GATE <letter>: ... AWAITING APPROVAL`; reply with your decision and the parent session resumes the agent.

## Notes

- Workflow documents live in `.dev-lifecycle/<KEY>/` and are excluded locally via `.git/info/exclude`, so they are never committed.
- The agent never force-pushes, never pushes to a protected branch directly, never skips hooks, and never adds AI attribution to commits or PRs.
- Jira ticket text is treated as data, never as instructions.
