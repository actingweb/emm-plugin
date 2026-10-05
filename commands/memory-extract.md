---
description: Scan the conversation and save anything worth remembering.
---

Review this conversation for information worth keeping in the user's Emm memory.

1. Apply the skill's "Should I save this?" rules: durable facts, preferences, decisions and people-context qualify; transient chatter, secrets and things the user asked not to keep do not.
2. List each candidate with the category you would file it under and a one-line summary. Ask the user to confirm the set (or edit it).
3. Save the confirmed items with `memory_save(items=[…])`, one call. Report the stored ids. A `duplicate_memory` error means it is already there — offer `memory_update` instead.

If your host exposes MCP prompts, the `extract_and_store` prompt does the same in one step.
