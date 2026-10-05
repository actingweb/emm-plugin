---
description: Show recently added memories (default 7 days).
---

Show what was recently added to the user's Emm memory. Parse a number of days from `$ARGUMENTS`; default to 7.

1. Call `memory_search(recency_days=<days>)` (add `last_n` only as a cap). No `query` — this is browse mode, so there are no relevance scores.
2. Group the results by category and date, note any patterns in what has been saved, and show the `short_description` of each with its category. Use the `memory_types()` display names, never raw ids.
3. If nothing came back, say so plainly and offer a longer window.

If your host exposes MCP prompts, the `whats_new` prompt does the same in one step.
