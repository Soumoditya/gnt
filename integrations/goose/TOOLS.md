# gnt-brain tools

This file is intended to be loaded by Goose as project hints or persistent instructions. The MCP
connection itself is configured separately; see [`CONNECT.md`](CONNECT.md).

gnt-brain exposes `check_action`, `search_rules`, `get_rule`, `list_skill_packs`, and
`get_skill_pack`.

## Before you act

Before any action that sends a message, moves money, deletes data, or is otherwise hard to undo,
call `check_action` first with a plain-English description of what you are about to do.

This file is an instruction for Goose; it is not an enforcement boundary by itself. The client or
action executor that performs the side effect must reject a call without an `allowed` verdict for
the exact action. Bind that approval to the recipient, amount, target, and scope, and run a fresh
check if any of those details change. Never reuse a verdict from a different action or an earlier
request.

- If the verdict is `allowed`, proceed with the action.
- If the verdict is `blocked`, do not proceed. Tell the user why and cite the rule returned by
  gnt-brain.
- If the verdict is `needs_human`, stop and ask a human to approve the action.

Never treat a missing, failed, or unclear verdict as permission to act.

## Before answering a policy question

Use `search_rules` to find the organization's approved rules. Use `get_rule` to retrieve a rule by
ID when more detail is needed. Only approved rules are returned; do not infer policy from drafts,
rules in review, rejected rules, or deprecated rules.

Use `list_skill_packs` and `get_skill_pack` for the organization's compiled context instead of
guessing at company policy.

## When a human is needed

On a `needs_human` verdict, stop. Do not retry with a softer description, route around the check,
or choose a branch on the human's behalf. Surface the proposed action and gnt-brain's reason, then
wait for the human's decision.
