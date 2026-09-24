# fibtec-tengrade-plugin

A Claude Code Plugin that connects Claude to TenGrade's examiner tools: list and read exams and instance reports, build and publish exams, and invite candidates — all from inside an agentic coding or ops workflow, without leaving Claude Code.

This repository is the whole distributable unit: a plugin manifest, an MCP server reference pointing at your org's own `TenGradeMcpApi` deployment, and a Skill that teaches Claude how to use the tools safely (in particular, the two tools that spend organisation credit).

## What's in here

| Path | Contents |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | A local marketplace listing this one plugin, for private/internal distribution |
| `.mcp.json` | Points Claude Code at your `TenGradeMcpApi` deployment over remote HTTP, authenticated with a TenGrade API key |
| `skills/tengrade/SKILL.md` | Teaches Claude the required sequencing around the credit-spending tools and the rate limits that gate every write tool |

## Prerequisites

You need a TenGrade organisation with:

1. A **write-enabled API key**. In the examiner portal, an org Owner goes to **API keys**, creates a new key, and checks *"Allow this key to create and publish exams (spends credit)"*. A key without this box checked can still use the read-only tools (`list_exams`, `get_exam`, `get_instance_report`, `get_org_dashboard`), but every write tool will refuse it with a plain error rather than fail silently.
2. Your organisation's **`TenGradeMcpApi` URL** — the endpoint of the `TenGradeMcpApi` CDK stack your TenGrade deployment runs. This is account- and region-specific, not a single fixed address, so it isn't hardcoded in this repository. Your TenGrade admin can give you this (it's the `McpApiUrl` output of the `TenGradeMcpApi` CloudFormation stack).

## Install

Add this repository as a marketplace, then install the plugin from it:

```shell
/plugin marketplace add fibtecltd/fibtec-tengrade-plugin
/plugin install tengrade@tengrade
```

Set the two environment variables `.mcp.json` expects, in whatever shell or environment Claude Code runs in:

```bash
export TENGRADE_MCP_URL="https://<your-api-id>.execute-api.<region>.amazonaws.com/mcp"
export TENGRADE_API_KEY="tgak_..."
```

`TENGRADE_API_KEY` is the full key string the portal shows you exactly once at creation time (`tgak_` followed by a key id and secret) — store it the same way you'd store any other bearer credential; TenGrade cannot show it to you again after that.

If Claude Code reports the install as `Run /reload-plugins to activate.`, run that command. Then try a read-only tool first, e.g. ask Claude to list your organisation's published exams, to confirm the connection works before trying anything that writes or spends credit.

## What this plugin will not do

Per TenGrade's own B9 design decision, this v1 tool surface deliberately excludes: taking an exam or grading an answer on Claude's own initiative, managing API keys (an agent should never be able to mint or revoke the very credential authorising it), and anything from the candidate-facing webcam-proctoring consent flow. `publish_exam` and `create_instance` — the only two tools that spend credit — always require an explicit `confirm: true` on a second call after a free price preview; see `skills/tengrade/SKILL.md` for the exact sequencing Claude follows.

## Status

Early v1, MIT licensed. Currently distributed as a private marketplace within Fibtec's own GitHub organisation (`/plugin marketplace add fibtecltd/fibtec-tengrade-plugin`) — this repository is still private, and flipping it to public is a separate, not-yet-taken step (tracked as TenGrade's own R11, `docs/tengrade-r11-plugin-publishing-design.md` in the main `tengrade` repo). Once public, `/plugin marketplace add` works for anyone with no further action needed; a listing in Claude Code's own community marketplace (a real, documented submission process, distinct from the officially-curated one) is a further, also not-yet-taken step.
