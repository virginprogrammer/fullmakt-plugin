---
name: fullmakt-govern-agent
description: Explain and guide the human side of Fullmakt governance: giving an autonomous agent its own scoped access (agent principal and OAuth client), writing a collection policy, and handling the approval loop. Use when the user asks how to let an agent call an API safely, how to require approval for writes, how to revoke an agent, or where the audit log is.
---

# fullmakt-govern-agent

This connector acts as the signed-in human. Autonomous agents get their own,
narrower access through Fullmakt's agent surface, and those objects are
created by the human in the Fullmakt console, not through this connector.
Guide the user; do not pretend to create principals, clients or policies
yourself. Reference: https://fullmakt.ai/docs/agents

## Give an agent access

1. Console → Agents → **New agent principal**. One principal per agent.
2. Create an OAuth client for it, choosing the collections it may call.
   The client secret is shown once; the agent uses it to mint a token at
   `/mcp-oauth/token` and calls `https://fullmakt.ai/mcp/{collection}`.
3. The agent sees only the requests in those collections. Credentials
   referenced by the requests are injected server-side; the agent never
   receives them.

## Write a policy

Console → collection → **Policy**. A policy is JSON with three parts:

- `hardDeny`: rules that always block, e.g. `{"field":"_operation","pattern":"^http\\.delete$","message":"No deletes"}`.
- `requireApproval`: rules that park the call as a 202 until a human
  approves it in the console, e.g. writes: `{"field":"_operation","pattern":"^http\\.(post|put|patch|delete)$"}`.
- `behavior`: rate and novelty limits such as `maxCallsPerMinute` or
  `newToolFirstUse`, each with `Deny` or `RequireApproval`.

`_operation` is `http.<method>` for HTTP requests; other protocols use their
own names (`smtp.send`, `sftp.upload`). Patterns are case-insensitive
regular expressions. Suggest the smallest policy that covers the user's
concern, and tell them to save it in the console.

## Approvals and audit

- Pending calls appear in Console → Approvals with the agent, tool and
  arguments; approving releases the call, rejecting returns an error to the
  agent.
- Every agent call, allowed or not, is in the audit log with the upstream
  host, status and latency. The signed-in human's runs from this connector
  are in the workspace history.
- To cut an agent off, rotate its client secret in the console: tokens
  minted before the rotation are refused within about 30 seconds. Deleting
  the client stops new tokens but leaves an already-minted token valid until
  it expires: one hour by default, up to 24 hours if the client was created
  with a longer token lifetime. So rotate first, then delete.

## When asked to do these things through this connector

Explain that the connector has no tools for principals, clients, policies or
approvals on purpose, so that agent governance always needs a human in the
console, and give the steps above.
