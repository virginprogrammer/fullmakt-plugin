---
name: fullmakt-setup
description: Connect the Fullmakt credential broker (MCP server at https://fullmakt.ai/mcp), sign in once, and verify the connection by listing workspaces. Use when the user installs this plugin, when a Fullmakt tool returns 401 or "not connected", or when the user asks how to connect Fullmakt to Claude or ChatGPT.
---

# fullmakt-setup

Fullmakt is a credential broker: the user's API credentials live in an
encrypted vault on fullmakt.ai and are injected server-side when a saved
request runs. No tool in this connector accepts or returns a secret, so never
ask the user to paste an API key into chat.

## Connect

1. The plugin declares the remote MCP server `https://fullmakt.ai/mcp`
   (`.mcp.json`). On first use the client opens Fullmakt's consent page.
2. The user signs in with their Fullmakt username and password and clicks
   **Allow**. Authentication is OAuth 2.1 with PKCE and dynamic client
   registration; there is no client id or secret to configure.
3. Access tokens last one hour and refresh automatically. Removing the
   connector makes the client discard its tokens; the access token expires
   within the hour and the refresh token within 30 days.

If the client cannot start the flow, the user can add the connector by hand:

- Claude (web, desktop, mobile, Cowork): Customize → Connectors → Add custom
  connector → URL `https://fullmakt.ai/mcp`.
- Claude Code: `claude mcp add --transport http fullmakt https://fullmakt.ai/mcp`, then `/mcp` to sign in.
- ChatGPT: Settings → Connectors → Developer mode → add an MCP server with the same URL.

## Verify

Call `list_workspaces`. A connected account returns at least one workspace with
an `id`; keep that id, every other tool needs it. An empty list means the user
has no workspace yet: point them at https://fullmakt.ai to create one.

If the call fails with 401, the token expired and the client did not refresh:
ask the user to reconnect the Fullmakt connector in the client settings.

## What to say about safety

- Reads (`list_*`, `get_*`, `fetch_url`) never change anything upstream.
- `run_saved_request`, `set_variable`, `create_request` and `update_request`
  are writes: confirm before running a request whose name or method suggests
  a side effect (POST, PUT, PATCH, DELETE).
- Every run is recorded in the workspace history and audit log the user can
  read in the Fullmakt console.
