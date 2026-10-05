---
description: Load the current Emm AI actions dashboard.
---

Use the `output_dashboard` Emm AI MCP tool to fetch the singleton `actions` dashboard's id — it never creates one; `agent_run` recreates a missing dashboard. It returns the dashboard id; follow with `output_get(id="actions:<id>")` to load the full body.

Render the dashboard contents as-is (it is already valid Markdown). Highlight what is under `## Today` and `## Pending decisions`, any inline `>`-quoted user comments (the user's direction for that item), and offer to act on items the user wants to address.
