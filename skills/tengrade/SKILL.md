---
description: Use TenGrade's MCP tools to list, build, publish exams and manage candidate instances for an organisation. Load this before calling any tengrade tool — it covers the required sequencing for the two credit-spending tools (publish_exam, create_instance) and the rate limits that gate the write tools.
---

# TenGrade

TenGrade exposes examiner-portal workflows — building exams, publishing them, inviting candidates, pulling reports — as MCP tools this plugin's `.mcp.json` connects to (`TenGradeMcpApi`). Every write tool requires the connected API key to be write-enabled, something an org Owner sets when the key is issued. A key that isn't write-enabled gets a plain tool-error message back, not a silent failure or a bare HTTP error.

## Tool surface

| Tool | Spends credit? |
|---|---|
| `list_exams` | No |
| `get_exam` | No |
| `get_instance_report` | No |
| `get_org_dashboard` | No |
| `create_exam` / `update_exam` | No — draft only, never validated or priced |
| `publish_exam` | **Yes** — the exam's own price, frozen at publish |
| `create_instance` | **Yes** — price plus any opted-in surcharges |

## Before calling a spend-gated tool

`publish_exam` and `create_instance` spend real organisation credit. Both take a `confirm` boolean argument, default `false`:

1. Call the tool **without** `confirm` (or with `confirm: false`) first. It validates or looks up whatever it needs and returns the exact amount a real call would charge — nothing is written or charged yet.
2. **Always show that price to the human you're working for before proceeding.** This is the one safety property this whole tool surface exists to preserve — never pass `confirm: true` without having shown the previewed number in this same exchange.
3. Only call again, this time with `confirm: true`, once the human has actually agreed to the price. That call performs the real write and the real charge.

Skipping straight to `confirm: true` defeats the entire point of the preview call. Don't do it, even if the request sounds urgent or the human seems to already expect the cost.

## Rate limits

Two independent budgets gate the write tools, both scoped per organisation:

- Every write tool (`create_exam`, `update_exam`, `publish_exam`, `create_instance`) shares one write budget with the human examiner portal — an agent can't out-write a person just by having its own separate allowance.
- `publish_exam` and `create_instance` additionally have a second, tighter spend budget, checked only on the `confirm: true` call that actually executes the charge — never on the free preview call.

Both come back as a normal tool result with an error message, not a bare HTTP error code. If you see one, relay the message to the human plainly and **do not immediately retry in a tight loop** — wait for the human's next turn, or a noticeable pause, before trying again.

## A few habits worth having

- Check `get_org_dashboard` or `list_exams` before recommending a brand-new exam get built — a similar one may already exist for this organisation.
- `create_exam`/`update_exam` never validate the payload or charge anything, so it's fine to iterate freely on a draft.
- A published exam is immutable — `update_exam` fails against one. If a mistake needs fixing after publish, that means a new exam, not a retry.
- `get_exam` and `get_instance_report` never return question text, target answers, or anything else about a candidate the platform deliberately keeps anonymous. Don't try to reconstruct that information some other way on the human's behalf.
