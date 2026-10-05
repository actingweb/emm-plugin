---
description: Generate a personalized news summary from stored preferences.
---

Produce a personalised news summary. `$ARGUMENTS` may name a source (a site, a newsletter, or "email"); if empty, use every source you have.

1. Call `memory_search(query="in news: *")` for the user's followed topics, sources and formats, plus `memory_search(query="news preferences")` for anything filed elsewhere.
2. Gather today's items from the named source (web search, or the user's email tools if they asked for email) and keep only what matches those preferences.
3. Write the summary in the format the preferences describe; otherwise a short headline list grouped by topic, each with a source link.

If your host exposes MCP prompts, the `daily_news` prompt does the same in one step.
