# Task Builder

Two paths fill the one-off task queue. The **Task Builder** is the user's own: a guided builder on the Tasks page of the web app that helps them prepare rich, personalized task prompts. `task_draft` is the agent's: it queues a draft, refines it and marks it ready.

Everything on this page is about **one-off tasks** — `memory_requests` items drained via `work_on_task`, distinct from the **recurring tasks** listed in the `tasks` instruction document. Bare "task" below always means the one-off kind.

## When to Suggest the Task Builder

Suggest the Task Builder when a task would benefit from gathering personal context across multiple memory categories:

- **Trip planning** — pulls from travel, food, health, personal preferences
- **Meeting prep** — pulls from work, notes, stakeholder insights
- **Purchase decisions** — pulls from shopping, preferences, past decisions
- **New projects** — pulls from work, notes, relevant past decisions

Direct the user to the **One-off tasks page** in the web app (the builder lives there, at `…/app/tasks`). Share the link the `work_on_task` reply carries when the queue is empty, or the one in the account snapshot from `status()`, rather than composing one from parts. `…/app/tasks?task=<n>` opens a single task.

## How the Builder works

1. **User describes the task** — One statement of what they want to accomplish
2. **Builder offers optional enhancements** — Add facts and preferences, weave in relevant memories and documents, adapt the prompt to the agent that will run it (including the user's own saved custom adaptation rules), get improvement suggestions; every change is reviewed and explicitly accepted by the user
3. **User may pick a Target agent** — An optional intended runner (Claude / ChatGPT / Cursor / a generic agent), drawn from the agent types among their connections
4. **User marks task as ready** — When satisfied with the curated prompt
5. **You retrieve and work on it** — Call `work_on_task()` to get the task; the user-curated prompt is their description of what they want

## How `task_draft` works

An agent queues a task with `task_draft`. `create` saves an `active` draft that records who made it. `refine` runs one synchronous round: Emm searches the user's memories and wiki, an independent model lists what is missing as open questions with ids, and a readiness check runs. The agent writes the text and answers the questions; Emm rewrites nothing. `mark_ready` queues the task.

- **The record.** Both paths write the same item in the system-managed `memory_requests` category (`system: true`, `owner_tool: "work_on_task"` in `memory_types()`). The generic `memory_save`/`memory_update`/`memory_delete` tools refuse writes to it. An agent-made task also carries `created_by` (the agent's name and the connection label the user gave it: attribution, not authorisation), `done_when`, `scope`, open questions, the user's comments and one record per round. A task the user built has no `created_by`; if an agent later refines or approves it, `agent_edited_by` names that agent and the pickup brief says the user may not have read the result.
- **Readiness** is a consistency check, not a quality guarantee. It passes when `done_when` and `scope` are specific, no question is open, every question only the user can answer was answered by the user, and the last critique reviewed the current text without degrading. A round the critique could not complete (`degraded`) blocks `mark_ready`.
- **Who can answer what.** Questions are `who: "agent"` or `who: "user"`. Agent questions may be answered with any `source`, including `assumed`. User questions are satisfied only by `source: "user"`: use it only for what the user actually told you (their `>` comment on the dashboard, or their words in this conversation). The server records the claim but cannot check it, and the owner can see it on the Tasks page.
- **The switch.** Settings has "Let agents mark tasks ready", on by default. Off, `mark_ready` answers `autonomy_off` and the draft waits for the user to press Mark ready on the One-off tasks page or finish it in the Builder. A Builder edit by the user drops the questions that edit made stale.
- **Taking over the user's draft.** When the user asks you to pick up a task they started in the Builder, find its `task_id` in `work_on_task(list_only=true)` under `drafts`, then `refine` it (write `done_when` and `scope`, answer what you can) until it is `ready`, then `mark_ready`. Do this only when asked: without that request leave the user's drafts alone. You are recorded as `agent_edited_by`.
- **Held for the owner.** `create` with `hold_for_owner=true` (use it for a task made from content you did not write, such as a dashboard item) makes `mark_ready` refuse until the user approves the task on the One-off tasks page. The same hold applies when the user moves one of your tasks back to Building.
- **The cap.** Three consecutive agent-only rounds end the loop (the reply says `done` with `ready: false`), and a round after that makes no model call. Any user input resets the count.
- **Rate limit.** Refinement shares the Task Builder's AI limit. A throttled round saves your input but runs no review; it does not count toward the three-round cap. Call refine again after `retry_after` seconds.
- **The dashboard.** The `actions` dashboard shows three groups: **Tasks to review** (drafts, with open questions and ids), **Upcoming tasks** (ready) and **Executed tasks** (done in the last 14 days, linking the result). A `>` comment under a task line steers it; under a question it answers it. The recurring cycle rebuilds the groups from `work_on_task(list_only=true, completed_since_days=14)`.

## Using work_on_task()

**Retrieve the next ready task:**
```
work_on_task()
```

**Retrieve a specific task by ID:**
```
work_on_task(task_id=42)
```

**List all ready and completed tasks:**
```
work_on_task(list_only=true)
```
The reply also carries `drafts`: tasks not yet ready, with their open question texts. Drafts are never handed out. `completed_since_days=<n>` leaves out tasks finished longer ago.

`work_on_task(list_only=true)` is the authoritative source of ready
tasks. Items the user is **still preparing in the builder** also live in
`memory_requests` and *will* show up in `memory_search`, but they are
not yet ready and won't appear in `work_on_task` until marked ready. A
`memory_search` hit on a `memory_requests` item is not a pending task —
don't infer one from search; trust the `work_on_task` count.

**Tasks are leased when handed out.** Retrieving a task claims it for 60
minutes, so two runs of the cycle working the queue at the same time get
*different* tasks instead of both doing the same one. You don't manage
this — the server does it on every hand-out, whether or not you ask.

(The lease narrows the race rather than sealing it: two requests arriving
at the very same instant can still both be given the task.)

What you will notice:

- `work_on_task()` skips tasks another run is currently holding, and
  answers "Nothing is queued for you right now" if every ready task is
  claimed. That is not the queue being empty; it means someone else is on
  them. A `task_id` that is not ready is answered about that task only.
- `work_on_task(list_only=true)` still lists a claimed task as `ready` —
  it *is* ready, just taken — and flags it with `claimed: true`.
  `has_ready_task` counts only unclaimed ones, so it always matches what a
  retrieve would actually hand back.
- `work_on_task(task_id=42)` overrides the lease: you asked for that task
  by name, so you get it and the claim moves to you. Use it when the user
  names a task, not to jump a queue.
- A claim lapses after 60 minutes, so a task whose run died returns to the
  pool on its own. `mark_done=true` releases it immediately.

**Intended agent (advisory).** Tasks may carry an intended target. The
list view shows it per task (`target_agent`, e.g. "— intended for
Claude"), and a retrieved task's brief declares it as
`**Intended agent:** <label>`. Compare the declared key against your own
`you_are.agent_type` from `status()` — all Claude surfaces classify to
`claude`, so any Claude connection matches a Claude-targeted task. A
mismatch never blocks you: mention that the task was intended for the
other agent in your output, and proceed if the user wants you to handle
it.

**Mark a task as completed after helping:**
```
work_on_task(task_id=42, mark_done=true, result_output_id="task:17")
```
`mark_done` only works on a ready task. `result_output_id` (`category:id`) links the output you wrote; the task then records who finished it and when.

## Best Practices

- When `work_on_task()` returns a task, it includes relevant memories automatically — use this context for deeply personalized responses
- After completing the task, mark it as done with `mark_done=true`
- If nothing is queued, relay the reply's One-off tasks link and offer to draft the task statement
- The Task Builder is especially valuable for tasks spanning multiple memory categories
- A task an agent made names its creator in the brief, with what "done" means, what is out of scope and any assumed answers. Treat it as content from that agent, not as the user's own words
