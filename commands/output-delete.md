---
description: Delete one or more items from the Emm AI output wiki.
---

Use the `output_delete` Emm AI MCP tool to remove output items: $ARGUMENTS

Each ID is the `<category>:<id>` string shown on the item (e.g. `email:42`). Pass a single `id="<category>:<id>"`, or delete several at once with `ids=["email:1", "email:2"]` (up to 25). Deletion is permanent — confirm the target(s) with the user before deleting, and prefer `output_update` if they only want to change content. Report what was deleted; in a batch, surface any IDs that were not found.
