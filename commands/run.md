---
description: Run the Emm AI recurring agent cycle (`agent_run`) — drain tasks, update the dashboard, write outputs.
---

Call the `agent_run` Emm AI MCP tool to start the user's recurring cycle.

`agent_run` accepts a `mode` argument (default `full`):
- `full` — the core (below) plus every *enabled* task's name in `## Tasks this run`. Use for a regular cycle.
- `preview` — assembles the same core but does NOT start a run; no `run_id` is minted and `agent_run_complete()` should NOT be called.

`agent_run` returns a **core**, not the whole cycle: the standing `agents` brief, the active `tasks` doc (custom procedures pointed at, not inlined), the shared core of `default_tasks`, a generated `## Tasks this run` list naming each task's declared instruction docs, and a pointer to the current `actions` dashboard. In `preview` mode, the response is prefixed with `⚠️ PREVIEW MODE — NOT YET STARTED`.

Pull each task's own procedure and declared docs as the cycle reaches it: `agent_run_task(task="<name>", run_id="<id>")`. Pass `already_loaded` with the short names of instruction docs already in context so they aren't re-sent; every reply prints the value to pass next. Hand a task to a sub-agent with `fresh_context=true` (it then also gets `agents` and the `default_tasks` shared core) plus the `run_id` copied verbatim — one instruction: pull, do the task, write its outputs, report back, never close the run.

Execute the cycle in a single response, pulling as you go. Output writes (`output_create`, `output_update`) are pre-authorised by the trigger — don't ask for permission on each task. Deliverables are the outputs themselves plus an end-of-run summary; not a description of what you intend to do.

When a cycle finishes (success or partial) in `full` mode, call `agent_run_complete(run_id="<id>")` **exactly once**. `agent_run`'s payload carries no structured fields at all — the `run_id` lives only in the prose, on the `**Run ID:**` line of the preamble, and again in the `agent_run_complete(run_id="…")` reminder at the end; read it from there. `agent_run_complete`, by contrast, *is* structured: it answers `{ status: "ok", marked_done: true, run_id }` (first close) or `{ status: "ok", already_complete: true, run_id }` (already closed, abandoned, or unknown / stale `run_id`). Treat `already_complete: true` as harmless — don't surface to the user, don't retry. Preview mode does not need this call.

Another run may be open while yours runs — a scheduled cycle, or the user working in another client. That is supported: proceed with your own run, re-read shared pages (the dashboard especially) before overwriting them, and close **only** the run you started. `agent_run` lists any other open runs in its preamble.

If `agent_run` returns `agent_os_not_enabled`, tell me the mission-control pillars are turned off in Emm AI settings and stop — only memory is available.
