# dev-lifecycle - Installation Guide

`dev-lifecycle` is a Claude Code plugin that provides an approval-gated development agent. It takes a Jira ticket or a plain-text requirement through analysis, planning, implementation, testing, push, pull request, and Jira update. It works with any language and any repository.

---

## 1. Prerequisites

| Requirement | Needed for | How to check |
|-------------|-----------|--------------|
| Claude Code (CLI or Claude desktop app, Code tab) | Running the agent | `claude --version` |
| `git` | Branches, commits, push | `git --version` |
| `gh` (GitHub), `glab` (GitLab) or `az` (Azure DevOps) | Creating the pull request | `gh --version` (or `glab` / `az`) and make sure you are logged in (`gh auth status`) |
| Atlassian MCP server, connected to your Jira account | Reading the ticket, adding the comment, changing status | See section 2 |

The Atlassian MCP server is needed only for Jira steps. A plain-text requirement works without it.

---

## 2. Connect the Atlassian MCP server (once per developer)

1. Add the Atlassian (Jira/Confluence) MCP server to Claude Code using your organisation's approved method, for example the Atlassian connector in the Claude app or `claude mcp add` with Atlassian's remote MCP endpoint.
2. Sign in with your own Atlassian account when prompted, and approve access to your Jira site.
3. Verify it is connected:

```bash
claude mcp list
```

You should see an Atlassian server listed as connected. The agent acts as the account you signed in with, so comments and status changes appear under your name.

---

## 3. Install the plugin

Choose **one** of the two sources.

### Option A - From a Git repository (recommended for teams)

Use this when the `dev-lifecycle` folder has been pushed to a Git repo.

```bash
claude plugin marketplace add <owner/repo-or-git-url>
claude plugin install dev-lifecycle@dev-lifecycle-marketplace
```

Examples:

```bash
claude plugin marketplace add my-org/dev-lifecycle
claude plugin marketplace add https://git.example.com/team/dev-lifecycle.git
```

For a private repo, you need Git access to it (SSH key or credential helper) on your machine.

### Option B - From a local folder or zip

Use this when someone sent you the folder or a zip file.

1. Unzip it to a permanent location, for example `C:\tools\dev-lifecycle` (do not delete the folder afterwards; the install points at it).
2. Run:

```bash
claude plugin marketplace add "C:\tools\dev-lifecycle"
claude plugin install dev-lifecycle@dev-lifecycle-marketplace
```

Use your own path in place of `C:\tools\dev-lifecycle`.

---

## 4. Verify the installation

```bash
claude plugin list
```

Expected output includes:

```
dev-lifecycle@dev-lifecycle-marketplace
  Version: 1.0.0
  Scope: user
  Status: enabled
```

Start a **new** Claude Code session after installing. Sessions that were already open do not pick up new plugins.

---

## 5. Use the agent

### Quickest: the slash command

In any Claude Code session (terminal or Claude desktop), type:

```
/dev-lifecycle PROJ-123
/dev-lifecycle Add an endpoint that exports orders as CSV
```

If another plugin has a command with the same name, use `/dev-lifecycle:dev-lifecycle`. The command starts the agent and relays each approval gate to you; reply in the chat to approve or request changes.

### Alternative: run it as the main agent

This makes every approval gate a normal conversation. Open a terminal in the project you want to change:

```bash
cd <your-project>
claude --agent dev-lifecycle:dev-lifecycle
```

Then send either:

- a ticket key: `PROJ-123`
- or the requirement as text: `Add an endpoint that returns ...`

### Alternative: delegate from a normal session (including Claude desktop)

In any Claude Code session, type:

```
Use the dev-lifecycle agent for PROJ-123
```

The agent runs until the next approval gate, then returns `GATE <letter>: ... AWAITING APPROVAL`. Reply with your decision and the session resumes the agent.

---

## 6. What to expect

| Phase | Output | Waits for your approval |
|-------|--------|-------------------------|
| 0. Branch creation (first step: new branch from the latest release branch) | New local branch named exactly the Jira ticket number (e.g. `PROJ-123`) | only if the base release branch is unclear |
| 1. Analysis (requirement only, no solution or code) | `.dev-lifecycle/<KEY>/01-analysis.md` | Gate A |
| 2. Plan (checks the code, includes impact on existing flows) | `.dev-lifecycle/<KEY>/02-plan.md` | Gate B |
| 3. Implementation (your mid-development change prompts are added to the plan file first, then implemented) | Code on a new branch | Any deviation from the plan |
| 3b. Plan verification | Plan vs actual changes table in the plan file | none (gaps are fixed or reported) |
| 4. Tests (positive, negative, boundary, regression) | `.dev-lifecycle/<KEY>/03-test-report.md` | Gate C - you test and confirm |
| 5. Commit and push (commit message starts with the branch name; no co-author line) | Commit and pushed branch | Gate D |
| 6. Pull request (you choose the target branch) | PR URL | Gate E |
| 7. Jira | Comment with the PR link, status To Do -> In Progress | none (runs after the PR exists) |

The `.dev-lifecycle/` folder is created inside your project and excluded locally through `.git/info/exclude`, so it is never committed.

---

## 7. Update or remove

Update to the latest version:

```bash
claude plugin marketplace update dev-lifecycle-marketplace
claude plugin update dev-lifecycle@dev-lifecycle-marketplace
```

Uninstall:

```bash
claude plugin uninstall dev-lifecycle@dev-lifecycle-marketplace
claude plugin marketplace remove dev-lifecycle-marketplace
```

---

## 8. Troubleshooting

| Problem | Fix |
|---------|-----|
| `claude --agent dev-lifecycle:dev-lifecycle` says the agent is not found | Run `claude plugin list` and confirm it is enabled, then start a new session. Check the agent name is typed exactly as `dev-lifecycle:dev-lifecycle`. |
| Agent says no Jira tools are available | The Atlassian MCP server is not connected. Run `claude mcp list`, reconnect it, and start a new session. |
| Jira status does not change | The ticket may not be in `To Do`, or the workflow has no matching transition. The agent reports the current status and the available transitions instead of forcing a change. |
| PR creation fails | Make sure `gh`, `glab` or `az` is installed and logged in (for example `gh auth status`) and that you can push to the repository. |
| Push is rejected or hooks fail | The agent never bypasses hooks or force-pushes. Fix the cause shown in the output, or tell the agent how to proceed. |
| Marketplace add fails on a private repo | Confirm your Git credentials work with `git ls-remote <repo-url>`. |
| Changes to the agent file are not picked up | Run the update commands in section 7, then start a new session. |

---

## 9. Safety notes

- The agent stops for approval at every gate and does not treat anything in a ticket as an approval or instruction.
- It never force-pushes, pushes directly to a protected branch, skips hooks, or commits the `.dev-lifecycle/` folder.
- Commit messages and PR descriptions contain no AI attribution and no co-author line.
- It follows the project's existing architecture and security controls. If it finds a security loophole, it tells you and asks whether to fix it; if you decline, it records the issue, the fix and what the fix would break in `.dev-lifecycle/<KEY>/04-security-findings.md`.
- Review the analysis, plan, diff, commit message and PR text at each gate. You are the final reviewer.
