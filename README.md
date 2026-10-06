# Apono skills

Agent skills for [Apono](https://apono.io) access management. Install once, and your
coding agent routes access to the systems Apono brokers through Apono — without you
asking for it every time.

## Install

```
npx skills add apono-io/skills
```

The `skills` CLI detects the AI coding agents on your machine — Claude Code, Cursor,
Codex, OpenCode and many more — and installs into each one it finds.

| Action | Command |
| --- | --- |
| Install | `npx skills add apono-io/skills` |
| Update | `npx skills update apono` |
| Remove | `npx skills remove apono` |
| Check what is installed | `npx skills list` |

## What you get

One skill, `apono`.

It activates on its own when a request touches a system Apono brokers — an S3 bucket, an
EKS namespace, a production database, a GitHub pull request, a Jira issue, a Grafana
dashboard, an Okta group. You can also invoke it directly with `/apono`.

Once active, it tells your agent to check what Apono already reaches before using `aws`,
`kubectl`, `psql`, `gh`, `terraform` or a separate MCP server for the same system, to
request access through Apono when none exists, and to report a denial rather than find
another route.

If the Apono gateway is not connected to your agent, the agent says so and does not reach
the system another way. It installs nothing and offers no install command.

## What it does not do

It changes nothing on your machine on its own. It registers no MCP server, edits no
configuration, and stores no credentials.

It states a preference; it does not enforce one. To make the preference binding, an
administrator adds deny rules for the local tools in the host's own settings. That is a
separate, deliberate step and is not part of this skill.

## Install the Apono gateway

The skill works through the Apono gateway, an MCP server that your agent connects to. If
your agent says Apono is not connected, install the gateway on your own machine, not
inside a container or a sandbox.

On macOS, Linux or WSL:

```
curl -fsSL https://apono-agentic-releases.s3.amazonaws.com/install.sh | bash
```

On Windows without WSL, in PowerShell:

```
irm https://apono-agentic-releases.s3.amazonaws.com/install.ps1 | iex
```

Either script registers the gateway in every AI client on the machine and signs you in.
Both need Node 20 or later.

Each installer ends by offering optional rules that steer AI clients away from the local
cloud credentials on the machine: deny rules for `~/.aws` and `~/.kube/config` in Claude
Code, and matching instructions in Cursor. Each file is backed up first. To apply the
rules without being asked, add the flag:

```
curl -fsSL https://apono-agentic-releases.s3.amazonaws.com/install.sh | bash -s -- --harden
```

```
&([scriptblock]::Create((irm https://apono-agentic-releases.s3.amazonaws.com/install.ps1))) -Harden
```

## Updating

Merge a change to the default branch. Users receive it with `npx skills update apono`.
There is no package to publish and no version to bump.

## Layout

```
skills/apono/SKILL.md   the skill
```
