# Apono gateway skill

## Problem

A customer who installs the Apono Agentic Gateway still has to ask for it by name. People type "use apono mcp" or "use the agentic gateway" before every request, because nothing tells the agent that Apono owns access to the systems in the request. When the reminder is absent, the agent uses a local command-line tool such as `aws` or `kubectl`, or a separate MCP server for the same system, and the access decision never reaches Apono.

The gateway already ships MCP server instructions that state the rule. Those instructions load as background context, so the agent weighs them against everything else in the session. They are not strong enough on their own, which the repeated manual reminders demonstrate.

An agent skill is a better fit. A host matches a skill against the words in the user request and then reads the skill as a procedure. The same rule becomes active guidance at the moment it applies.

## Goals

- An agent prefers Apono in a domain no existing policy names, which is where a per-system policy does not reach.
- An agent activates the guidance without the user naming Apono, for any request that touches a system in the managed catalog.
- A user activates the guidance explicitly with `/apono`.
- An agent prefers Apono over a local command-line tool or a separate MCP server for a system Apono reaches.
- An agent that finds no access requests it through Apono, instead of using a local credential.
- A customer installs and updates the skill with one command, in every AI client on the machine.

## Non-Goals

- Permission rules that block a command. A skill states a preference; only host settings enforce a block. Enforcement belongs to the installer or to an administrator.
- A hook. The distribution channel installs skills only, so a hook needs a different channel.
- A claim about which systems an account reaches. Feature flags and per-account configuration decide that at run time.
- One skill per domain. A single skill covers every domain in the catalog.
- Slack. The catalog marks the Slack definition as not yet released.
- Any change to the `apono-claude-plugin` repository.

## Design

The deliverable is one public repository that holds one skill file, distributed by the `skills` CLI.

### Repository and distribution

The repository `apono-io/skills` holds `skills/apono/SKILL.md` and a `README.md`. The repository has no build step, no package manifest, and no continuous integration. The skill is prose.

The `skills` CLI reads a skill directly from a Git repository, so no registry entry exists. The CLI takes the skill name from the directory name, so the directory name `apono` produces the `/apono` command. The CLI detects each AI client on the machine and links the skill into all of them.

The table below lists the commands a customer runs. A maintainer publishes a change by merging it to the default branch, because the CLI reads the branch and not a released package.

| Action | Command |
| --- | --- |
| Install | `npx skills add apono-io/skills` |
| Update | `npx skills update apono` |
| Remove | `npx skills remove apono` |

### The skill file has two parts

The frontmatter `description` carries the trigger vocabulary. A host matches a request against the description, so the description is the only part that decides whether the skill activates. It names the domains in the catalog and the alternative tools the skill displaces.

The body carries the policy and the procedure. The body states that Apono decides access for the listed domains, and it states the order of steps. The body carries no trigger words, because the body loads only after the skill activates.

### The description is a YAML block scalar

The `description` value uses a block scalar (`>-`) rather than a plain one. A plain YAML
scalar ends at a colon followed by a space, so a description that names a list after a
colon is invalid YAML. The `skills` CLI rejects the file and reports that the repository
contains no skills, which installs nothing and reports no error the user can act on.

The CLI is the only validator that matters, because it parses the file the way a host
parses it. A character count against the frontmatter cap does not detect this class of
fault.

### The description covers every domain, including the flagged ones

The description names each domain in the managed catalog whose definition is released, whether or not a feature flag gates it. The Kubernetes and MongoDB definitions carry feature flags, and the AWS definition carries a flag on one variant. A gated domain still belongs in the description, because a trigger word is not a claim about the account. The run-time discovery step decides what the account reaches. The description omits Slack, because the catalog marks that definition as not yet released.

The table below maps each domain to the words a user types and to the alternative the skill displaces. The mapping is the content of the description and of the body table.

| Domain | Words a user types | Alternative the skill displaces |
| --- | --- | --- |
| AWS | bucket, instance, function, queue, secret, table, role, policy, account | `aws`, `terraform`, boto3, an AWS MCP server |
| Kubernetes, EKS, AKS | cluster, namespace, pod, deployment, logs, Helm release | `kubectl`, `helm`, a Kubernetes MCP server |
| PostgreSQL, MySQL, MongoDB | database, table, collection, query, schema | `psql`, `mysql`, `mongosh`, a database MCP server |
| GitHub | repository, pull request, issue, branch, commit | `gh`, a GitHub MCP server |
| Atlassian | Jira issue, Confluence page, ticket | an Atlassian MCP server |
| Grafana | dashboard, metric, log query, alert, on-call | a Grafana MCP server |
| Okta | application, group, user assignment | `okta`, an Okta MCP server |
| Mixpanel | funnel, event, report | a Mixpanel MCP server |
| monday.com | board, item, group, column | a monday.com MCP server |

### The procedure names roles, not tools

The body describes four steps by role and names no gateway tool. Managed tool names change: the AWS variant replaces one tool with eight, and the access-request tool is due for replacement. The gateway publishes its own tool list and its own instructions into every session, so a tool name written in the skill is duplicated information with two update paths.

The four steps are discovery, action, request, and report. Discovery finds what Apono already reaches. Action performs the work through Apono. Request asks Apono for access that discovery did not find. Report states the outcome when Apono denies a request or leaves it pending.

The body states one rule for each of the two failure paths. A local credential is not permission, so a credential on the machine does not justify a direct call. A denial is an answer, so the agent reports it instead of completing the task by another route.

### The body counters named rationalizations

The failure the skill corrects is a discipline failure. An agent that reads the rule still
chooses the local tool under time pressure. Soft wording does not hold against that, so
the body carries a table of excuses with a direct answer to each, and a short list of
thoughts that signal the agent is about to skip the rule.

Baseline testing produced the excuses. An agent given the gateway instructions and a
Kubernetes incident chose `kubectl` and stated its reasoning: the local tool costs one
command, the brokered path costs several calls, and it did not know whether a Kubernetes
target existed for it. That last reason inverts the design, so the table answers it first.
Discovery is the call that resolves the uncertainty, so uncertainty is a reason to make the
call.

The same testing showed that an agent complies where a policy names the system and fails
where no policy names it. The domain table therefore carries the rule to every brokered
domain, which is the part that generalizes.

The table below lists the excuses the body answers.

| Excuse | Answer the body gives |
| --- | --- |
| A target may not exist for me | Discovery answers that in one call |
| The local tool is faster | One call against zero calls, not minutes against seconds |
| The operation is read-only | Reads are what access control governs, and an unbrokered read leaves no record |
| A teammate configured the credential | Installing a credential is not deciding this task may use it |
| An incident is in progress | Urgency changes which target the agent requests, never whether it asks |
| The agent discloses the bypass | Disclosure does not authorize the call |
| Another MCP server for the system is installed | A second tool outside Apono is a gap between access systems |
| The credential outlived the denial | The denial is the decision |
| Both paths can run at once | The local call lands first, so the broker decided nothing |

### The user keeps the override

A user may name a local tool. The body directs the agent to comply, to state once that the
call is outside Apono, and not to repeat the notice. Without this branch the skill argues
with its own user, which is a worse failure than the one it corrects.

An instruction that names only the outcome is not an override. The body draws that line,
because every task names an outcome.

### The skill offers to install the gateway

A skill activates independently of any MCP server, so the skill activates on a machine where the gateway is absent. Without a recovery path the skill reaches a dead end: it directs the agent to Apono, and Apono is not there.

The body therefore holds one branch for that case. The agent tells the user that the gateway is not registered, offers to install it, and waits for confirmation before it runs anything. The install command lives on a single line in the skill file. The delivery URL for the installer is not yet published, so that line is the one value a maintainer fills in when delivery is settled.

This branch is the only action in the skill that changes the machine, and it runs only after the user confirms it.

## Acceptance Criteria

- **AC-1**: WHEN a user request names a resource in a domain the skill description lists, THE host SHALL activate the skill without the user naming Apono.
- **AC-2**: WHEN a user sends `/apono`, THE host SHALL activate the skill.
- **AC-3**: WHILE the skill is active for a listed domain, THE agent SHALL determine what Apono reaches before it uses a local command-line tool or a separate MCP server for that domain.
- **AC-4**: IF Apono reaches the target, THEN THE agent SHALL perform the work through Apono, and SHALL NOT use a local command-line tool or a separate MCP server for the same system.
- **AC-5**: IF Apono reaches no target for the request, THEN THE agent SHALL request access through Apono, and SHALL NOT use a local credential in place of the request.
- **AC-6**: IF Apono denies a request or leaves it pending, THEN THE agent SHALL report that outcome, and SHALL NOT complete the task by another route.
- **AC-7**: IF the gateway is not registered in the host, THEN THE agent SHALL offer to install it and SHALL wait for user confirmation before it runs the installer.
- **AC-8**: THE skill description SHALL name every domain in the managed catalog whose definition is released, including a domain a feature flag gates, and SHALL omit a domain the catalog marks as not yet released.
- **AC-9**: THE skill SHALL name no individual gateway tool.
- **AC-10**: THE skill SHALL claim no access on behalf of an account, and SHALL treat run-time discovery as the only source of what an account reaches.
- **AC-11**: WHEN a maintainer merges a change to the default branch, THE update command SHALL deliver that change without a package release.
- **AC-12**: WHEN a user directs the agent to a named local tool, THE agent SHALL comply, and SHALL state once that the call is outside Apono.
- **AC-14**: WHEN the `skills` CLI reads the repository, THE CLI SHALL parse the skill file and report one installed skill named `apono`.
- **AC-13**: WHEN a user states only the outcome of a task, THE agent SHALL treat the statement as no override, and SHALL follow the discovery step.

## Backward Compatibility

No behavior change for an existing user. The skill is a new artifact in a new repository. It registers no MCP server, edits no host configuration, and replaces no existing skill. A customer who never installs it sees the gateway behave as it does today.
