# Fullmakt plugin

Governed API access for agents. This plugin connects Claude to the
[Fullmakt](https://fullmakt.ai) credential broker so you can run your saved
API requests from chat while your credentials stay in the Fullmakt vault.
Every call is policied and recorded in your workspace history.

The connector is one remote MCP server, `https://fullmakt.ai/mcp`, with
twelve tools split by effect:

- **Read**: `list_workspaces`, `list_collections`, `list_collection_items`,
  `get_request`, `get_history`, `list_environments`, `list_variables`,
  `fetch_url` (GET/HEAD only).
- **Write**: `set_variable`, `create_request`, `update_request`,
  `run_saved_request`.

No tool accepts or returns a credential. Vault values appear as `vault://`
references and are injected server-side when a request runs.

## Install

Claude Code or Cowork, from this repository:

```
/plugin marketplace add virginprogrammer/fullmakt-plugin
/plugin install fullmakt@fullmakt
```

Or add the connector directly in any client with the URL
`https://fullmakt.ai/mcp`. Sign in with your Fullmakt account on the consent
page; there is no client id or secret to configure (OAuth 2.1 with PKCE and
dynamic client registration).

## Contents

| Path | Purpose |
|---|---|
| `.mcp.json` | the remote MCP server |
| `skills/fullmakt-setup` | connect, sign in, verify with `list_workspaces` |
| `skills/fullmakt-run-collection` | find and run saved requests, environments, history, errors |
| `skills/fullmakt-govern-agent` | the human side: agent principals, policies, approvals, audit |
| `commands/fullmakt-status.md` | `/fullmakt-status`: workspaces and recent runs |

The skills are plain Markdown and are the same bundle uploaded to the OpenAI
plugin portal, which has no manifest of its own: the MCP URL is entered in
the submission form and `skills/` is uploaded as the skill bundle.

## Privacy Policy

The connector runs as the signed-in Fullmakt user. Requests it runs, their
responses and timing are stored in that user's workspace history under the
retention settings the user controls in the Fullmakt console. Credentials are
never sent to the assistant. Access tokens are valid for one hour and refresh
automatically; removing the connector makes the assistant discard its tokens,
after which the access token expires within the hour and the refresh token
within 30 days. The full
policy, including data handling for the connector, is at
https://fullmakt.ai/privacy#connectors.

## Support

Documentation: https://fullmakt.ai/docs/agents#connector.
Problems and questions: https://fullmakt.ai/#contact.

## Source

This repository is a mirror of the `plugin/` directory of the Fullmakt
source repository, which is private. Every change is validated there with
`claude plugin validate --strict` and synced here on merge, so pull requests
against this mirror are overwritten. Report problems through
https://fullmakt.ai/#contact.

## License

Apache-2.0, see `LICENSE`.
