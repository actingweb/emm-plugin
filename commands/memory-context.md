---
description: Load Emm AI memories relevant to a topic.
---

Load everything Emm holds about the topic in `$ARGUMENTS` and return a structured context summary.

1. Call `memory_search(query="<topic>")`. Keep hits scoring above 50, treat 25–50 as background, drop the rest.
2. If the account has the wiki, also call `output_search(query="<topic>")` for prior documents, plans and research.
3. Summarise under short headings (facts, preferences, decisions, open items, related documents) with a one-line pointer per item. Ask whether the user wants more detail before quoting full bodies.

If your host exposes MCP prompts, the `load_context` prompt does the same in one step.
