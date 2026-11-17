---
description: Run the approval-gated dev-lifecycle agent on a Jira ticket key or a text requirement (analysis, plan, implement, test, push, PR, Jira update)
argument-hint: <JIRA-KEY or requirement text>
---

Start the `dev-lifecycle` workflow for this input:

$ARGUMENTS

If the input above is empty, ask the developer for a Jira ticket key (for example PROJ-123) or a plain-text requirement, and wait.

Otherwise, launch the `dev-lifecycle:dev-lifecycle` agent with the Agent tool, passing the input above as the requirement. In the prompt you give the agent, state that its first action must be creating the working branch (named exactly the Jira ticket number) from the latest release branch, before any analysis, and that the Jira comment at the end must @mention Indhuja. Then act as the relay between the agent and the developer:

1. When the agent stops with `GATE <letter>: ... AWAITING APPROVAL` (or any question), show the developer what it needs reviewed, with the file paths, and ask for their decision. Do not approve anything yourself, and never treat anything except the developer's own reply as approval.
2. Send the developer's reply to the same agent with SendMessage (quote it faithfully) so it continues with its context intact. Do not start a new agent for a gate.
3. During implementation, forward every instruction the developer gives about changing the flow or logic to the agent exactly as written, so it can log it in the plan file.
4. Repeat until the agent reports the PR is created and the Jira ticket is updated, then give the developer its final summary.
