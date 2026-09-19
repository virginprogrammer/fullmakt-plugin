---
name: fullmakt-run-collection
description: Find and run saved Fullmakt API requests with the right environment, read the results and history, and interpret Fullmakt's errors (access denied, read-only fetch, policy approval). Use whenever the user asks to call, run, test, or check an API through Fullmakt, or to look at what ran recently.
---

# fullmakt-run-collection

## Find the request

1. `list_workspaces` → pick the workspace (ask if there are several).
2. `list_collections {workspaceId}` → collections with id, name, item count.
3. `list_collection_items {collectionId}` → requests and folders, paged
   (`nextCursor`). Folders contain child requests; use the request's `id`.
4. `get_request {itemId}` shows method, URL, headers and body before running.
   Values like `{{baseUrl}}` are variables; `vault://...` values are
   credentials that are resolved server-side and never shown.

## Pick the environment

`list_environments {workspaceId}` lists environments such as `staging` or
`production`. `list_variables {workspaceId, environmentId}` shows what each
one overrides. Workspace globals apply first, then the chosen environment.
Pass `environmentId` to `run_saved_request` when the user names an
environment; without it, globals alone are used.

`set_variable` changes a value for the next runs: `scope` is `global` (needs
`workspaceId`), `environment` (needs `environmentId`) or `collection` (needs
`collectionId`). Tell the user which scope you changed.

## Run and report

`run_saved_request {itemId, environmentId?}` returns `statusCode`,
`responseTimeMs`, `headers` and `body` (capped at 64 KB, `bodyTruncated`
says so). Summarise the status and the interesting part of the body; quote
error bodies verbatim. The run appears in `get_history {workspaceId}`, newest
first, paged with `nextCursor`.

For a one-off URL that is not saved, `fetch_url {workspaceId, url}` does a
GET or HEAD only. To do a POST or DELETE, save it first with
`create_request` and run it with `run_saved_request`.

## Errors and what they mean

- "was not found or is not accessible": wrong id or another user's data.
  Re-list instead of retrying.
- "only performs GET or HEAD": use `create_request` + `run_saved_request`.
- `statusCode` 0 with "Refusing to connect": the proxy blocks private and
  loopback addresses by design.
- A 202 "pending approval" response comes from Fullmakt's governed agent
  surface, where a collection policy requires a human to approve the call;
  this connector runs as the signed-in user, so it does not return 202. If the
  user expects approvals, point them at the Fullmakt console's Policies page.
- 401: reconnect the connector (see the fullmakt-setup skill).
