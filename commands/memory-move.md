---
description: Move (recategorise) one or more Emm AI memories to another category.
---

Use the `memory_move` Emm AI MCP tool to move memories to a different category: $ARGUMENTS

Pass a single `id="<type>:<id>"` (e.g. `work:40`) or `ids=[…]` (up to 25) together with the `target_type`. A move assigns a new ID in the target category and rewrites inline references (`work:40`, wiki links, app URLs) across memories, outputs and instructions — never re-create + delete. The response carries the old → new ID mapping plus `prose_candidates`: documents that mention the old ID in free text (e.g. "memory 40") that can't be auto-rewritten — offer to fix those by hand. Confirm the target category with the user if it's ambiguous.
