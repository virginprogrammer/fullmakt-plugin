---
description: Summarise Fullmakt workspaces and the most recent API runs
---

Give the user a status overview of their Fullmakt account:

1. Call `list_workspaces`. If it fails with 401 or the connector is not
   connected, stop and follow the fullmakt-setup skill.
2. For each workspace (at most five), call `get_history` with `limit: 10`.
3. Report, per workspace: its name, how many runs are in the page, the
   newest run (method, URL, status, when), and how many of the ten failed
   (status 0, 4xx or 5xx). Quote the failing URLs and statuses.
4. If any workspace has no history, say so in one line.

Keep it to a short table or a few lines per workspace. Do not run any
request as part of this command.
