# Apono skills

Agent skills for [Apono](https://apono.io) access management. Install once, and your
coding agent routes access to the systems Apono brokers through Apono — without you
asking for it every time.

## Install

```
npx skills add apono-io/skills
```

The `skills` CLI detects the AI clients on your machine — Claude Code, Claude Desktop,
Cursor, Codex and others — and installs into all of them.

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

If the Apono gateway is not registered on the machine, the skill offers to install it and
waits for your confirmation.

## What it does not do

It changes nothing on your machine on its own. It registers no MCP server, edits no
configuration, and stores no credentials.

It states a preference; it does not enforce one. To make the preference binding, an
administrator adds deny rules for the local tools in the host's own settings. That is a
separate, deliberate step and is not part of this skill.

## Updating

Merge a change to the default branch. Users receive it with `npx skills update apono`.
There is no package to publish and no version to bump.

## Layout

```
skills/apono/SKILL.md   the skill
docs/plans/             design specs
```
