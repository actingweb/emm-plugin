---
description: Drain the Emm AI one-off task queue (work_on_task).
---

Use the `work_on_task` Emm AI MCP tool to retrieve and work on saved one-off tasks from the Builder.

Workflow:

1. `work_on_task(list_only=true)` — see what's queued.
2. `work_on_task()` — get one context-prepared task (with the user's framing and attached context).
3. Execute it; write the result as an output in the `task` category.
4. `work_on_task(task_id=ID, mark_done=true)` — close it.

If `$ARGUMENTS` names a topic or task id, focus on that item.

Retrieving a task **claims** it for an hour, so another run working the queue at the same time gets a different one. Two consequences: `work_on_task()` returning nothing does not mean the queue is empty — it can mean every ready task is currently claimed (`list_only=true` shows those as `ready` with `claimed: true`) — and asking for a specific `task_id` overrides the claim, so use it when the user names a task, not to jump the queue. `mark_done=true` releases the claim immediately.
