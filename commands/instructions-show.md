---
description: List Emm AI standing instructions (or load one by short name).
---

Standing instructions in Emm AI are the standing brief and procedures an agent loads for an agent run: `agents`, `tasks`, `default_tasks` (loaded up front by `agent_run`), plus `personal` and `style` (pulled per-task by `agent_run_task` as the cycle reaches a task that declares them). Argument: $ARGUMENTS

If `$ARGUMENTS` is empty, call `instruction_settings` first: on a guided account, show its summary (who I am, my voice, my tasks in run order with their notes and schedules). Then call `instruction_list` and show each installed instruction with its short name, maintainer (`maintained_by`: `emm` or `user`), whether `update_available` is set, and version.

If `$ARGUMENTS` names a short instruction (e.g. `agents`, `personal`), call `instruction_load(name="$ARGUMENTS")` and render the document body so I can review it.
