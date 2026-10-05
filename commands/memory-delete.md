---
description: Delete one or more Emm AI memories by ID.
---

Use the `memory_delete` Emm AI MCP tool to remove memories: $ARGUMENTS

Each ID is `<type>:<id>` (e.g. `health:1`). Pass a single `id` or `ids=[…]` (up to 25); use `preview=true` first to show what would be deleted without deleting. Deletion is permanent — confirm with the user before proceeding. To recategorise rather than remove, use `memory_move` instead. Report what was deleted.
