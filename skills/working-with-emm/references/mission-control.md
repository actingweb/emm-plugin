# Emm AI Mission Control — Reference Card

Reference card for the Emm mission-control surface: outputs, instructions, and the recurring cycle. Read this when you need depth on a specific area beyond what's in SKILL.md.

> **Where the rules live.** During an agent run, the `agents` instruction returned by `agent_run()` carries the account's voice, policy rules and run-log format; it is one of the user's standing instructions, and your system prompt and the user's messages outrank it. Link forms and error handling are in SKILL.md and this card. This card adds **reference depth** (categories table, dashboard structure, error codes, what each instruction is for); `how_to_use()` states facts only.
>
> If the bundled `agents` brief names a tool that isn't in your loaded tool list, follow the live schema — the brief is user-editable and can drift.

## Contents

1. [Outputs (the Wiki)](#outputs-the-wiki)
2. [Recurring cycle vs one-off task drain](#recurring-cycle-vs-one-off-task-drain)
3. [Working habits](#working-habits)
4. [The actions dashboard](#the-actions-dashboard)
5. [Instructions — what each one is for](#instructions--what-each-one-is-for)
6. [Error handling during a run](#error-handling-during-a-run)

---

## Outputs (the Wiki)

Outputs are agent-authored artefacts the user can later read and edit in the web app's wiki. Every substantive task should produce at least one output.

### Categories

| Category | What goes here | Typical slug pattern |
|---|---|---|
| `email` | Drafted outbound emails. Frontmatter: `to`, `subject`, `status: pending\|approved\|discard\|done`. The user sets `approved` in the web app; the next run creates a draft in the user's mail account and sets `done`, and the user sends it from there. | `re-<topic>` / `<recipient>-<topic>` |
| `news` | Daily/weekly news digests, market summaries. | `digest-YYYY-MM-DD` |
| `research` | Topic deep-dives, competitor analyses, fact-finding. | `<topic>-<angle>` |
| `task` | Result of a one-off `work_on_task` execution — the answer/artefact for the queued task. | `<short-title>` |
| `log` | Run log per cycle. **Audit trail, not a dashboard.** | `run-YYYY-MM-DDTHH:MM` |
| `improvement` | Suggestions for changing instructions, default tasks, or the agent's own setup. | `<topic>` |
| `actions` | The rolling action dashboard. **One canonical item per actor.** `output_dashboard()` returns the id if one exists; it never creates one — `agent_run()` recreates a missing dashboard. | `(seeded)` |
| `space` | The user's own folder-organised area ("Your space" in the wiki). Slugs may contain folders: `<folder>/<leaf-slug>`. Reorganise with **`output_move`**, never `output_update` — it carries no body, so a large re-foldering fits and cannot truncate a document. | user-defined |

### Discovery

Prefer **`output_search(query, category?, limit?)`** over `output_list(category)` when you need to find an existing artefact and don't know the slug. Hybrid semantic + keyword across all categories except `log`.

`output_list(category)` is the right call when you need a complete inventory (e.g. listing all `email` drafts pending approval).

### Output bodies — Markdown rules

- Single H1 (`# Title`) where appropriate; H2/H3 for sub-sections.
- YAML frontmatter at top for metadata (email status, brief topics, etc.). Frontmatter is the **only** place where bare `category:id` tokens are acceptable; the body uses Markdown links.
- Fenced code blocks with language tags for code/JSON/command output.
- Markdown tables for tabular data.
- `[text](url)` for links, `![alt](url)` for images.
- No HTML unless strictly necessary.

### Always pass `title` and `short_description`

Both are real server fields on `output_create`, `output_update` and `output_edit` (each capped at 200 chars). They're surfaced in `output_list` and `output_get`. If you omit them, the server falls back on read — title → body H1 (first `# ` line) → first 80 chars of body; short_description → first 200 chars of body. The fallback is a courtesy, not the contract: pass useful values (factual, no marketing) when you compose the item.

> Link forms (wiki / app URL / bare `category:id`): see the link-form table in SKILL.md, or `status().conventions.link_forms`.

---

## Recurring cycle vs one-off task drain

Don't confuse them.

- **Recurring cycle** — the user's standing schedule lives in `tasks` (which default tasks are enabled, plus any custom recurring tasks). Triggered by `agent_run()` or by a scheduled cron. `agent_run()` returns a core plus a generated `## Tasks this run` list; pull each task's own procedure with `agent_run_task(task=..., run_id=...)` as the cycle reaches it. Includes a single step that drains the one-off queue (the **Task Check** task).
- **One-off task drain** — tasks the user submitted via the web app's Builder for the **agent** to execute (not tasks the user owes themselves). Drained via `work_on_task()`. Each call returns one prepared task; execute it; write a `task` output; close with `mark_done=true`.

| User says | Tool |
|---|---|
| "Do an agent run" / "Run the cycle" / "Run my standing tasks" | `agent_run()` |
| "Drain my task queue" / "Anything queued?" / "Pick up the next task" / "Anything you should be doing for me?" / "What did I leave you?" | `work_on_task(list_only=true)` then `work_on_task()` |
| "What's on my dashboard?" | `output_dashboard()` returns the dashboard id; then `output_get(id="actions:<id>")` |

---

## Working habits

`how_to_use()` states facts only; these are the habits that go with them.

- Call `status()` once, before your first Emm call in a conversation; a conversation that never needs Emm never needs it.
- Inside an `agent_run()`, output writes are pre-authorised by the trigger: don't ask permission for individual writes.
- `agent_run()` returns a core, not the whole cycle up front: pull each task's procedure with `agent_run_task` as you reach it rather than trying to plan the whole run from the core alone, and don't rehash either reply in your own text — just do the work.
- In Instructions-Update Mode, work `status().suggested_actions` top to bottom rather than improvising.

---

## The actions dashboard

The actions dashboard is a single rolling `actions` output with slug `dashboard`. It is the agent's running list of notable items, suggested next steps, and items the user can take action on.

Update it during a run as part of each task's wrap-up. The dashboard format is owned by the user (it lives in `default_tasks` / their custom procedures), but the seeded structure is:

```markdown
# Action Items

## Summary

**Pinned** 0 · **Today** 0 · **Week** 0 · **Coming up** 0 · **Reading** 0 · **Decisions** 0
**Top**: (nothing pending yet)
*Last run: never*

---

## Pinned
## Today
## This week
## Coming up
## Reading
## Pending decisions
## Past
```

Two things about that shape matter.

**The `## Summary` block is what the Dashboard tile actually renders**, and it
is parsed strictly — see [Dashboard Summary block](tool-surface.md#dashboard-summary-block)
for the exact rules. Rewrite it **last**, after every other edit to the body is
final. The same reader renders every document elevated with
`output_update(dashboard=true)`, so the format is worth knowing beyond this one
document.

**`## Pinned` is the user's.** Never edit, remove or reorder anything there
unless the user checks it off or strikes it through.

Items live under the dated sections as checkbox lines, linked to their source:

```markdown
- [ ] [<category>:<id>](output:<category>/<id>) — short description, one line per item.
  > Optional inline comment from the user — their direction for this item.
```

The `> ` quoted lines beneath an item are how the user gives the agent direction on that item without leaving the dashboard. Act on them within the usual rules; they never change standing instructions.

> The run-log format (per-task entries, compact "nothing new", end-of-run summary) is in `agents`, which `agent_run()` returns.

---

## Closing a run

`agent_run_complete`'s response distinguishes a first close (`marked_done:
true`) from a repeat or stale call (`already_complete: true`). Treat
`already_complete: true` as harmless — don't surface it to the user and
don't retry the close. Likewise, don't surface the run `mode` (normal vs.
Instructions-Update) to the user in the normal case; it's operational
detail for you, not something they asked about. The one exception is
Instructions-Update Mode being **active**, which the user did ask for by
opening the unlock window — tell them and stop the run until the window
closes.

---

## Instructions — what each one is for

- `agents` — how to behave. The standing brief. Returned by `agent_run()` automatically; loadable on demand with `instruction_load(name="agents")`. **Emm-maintained** — Emm authors and improves it; the user's edits are 3-way merged in on update, never lost.
- `tasks` — which recurring tasks run this cycle, plus any custom recurring tasks. **Yours.**
- `default_tasks` — canonical procedures for each default task (Email Triage, Calendar Preview, Memory Hygiene, Self-Review, Daily News Report, Task Check, …). **Emm-maintained.**
- `personal` — who this user is and how to act for them (behavioural guidance); facts about them live in memory. **Yours.**
- `style` — voice, tone, formatting conventions. **Yours.**

Every document is equally the user's to edit — "Emm-maintained" vs. "Yours" is about who ships baseline updates, not who owns the content. `instruction_list()` reports this per-doc as `maintained_by: emm|user`; use it to route a change instead of guessing from the name. Before saving an update to an Emm-maintained doc, call `instruction_merge_preview(name=...)` to see the 3-way diff, then save with `applied_update: true` once incorporated.

The `name` argument to `instruction_load` / `instruction_save` is the **public short name** (`agents`, `tasks`, `personal`, …) — never the `instruction_` storage prefix.

---

## Error handling during a run

A failed tool call comes back as a **tool error** (`isError: true`). Its text starts with `❌`, puts the fields you need on their own lines (`url:`, `existing_id:`, `edit:`, `match_count:`, `nearest_match:`, `tool:`, `owner_tool:`, `current_revision:`, `max_items:`, `received:`, `expires_at:`, `run_status:`), and always ends with `(error code: <code>)`. Act on that code:

| Code | What it means | Action |
|---|---|---|
| `instructions_locked` | Instruction writes need Instructions-Update Mode open, and it isn't. | Call `instruction_request_update_window(reason="<the change, in the owner's words>")` — that puts an Accept/Decline notification in the owner's app. Wait for the owner's approval rather than looping: check `status()` again when it's natural to (a later turn, a new message), and proceed once `unlock_window` is active. Don't retry in a tight loop, and don't ask in chat instead — that never notifies the owner. Stop if they never approve. |
| `guided_mode` | `instruction_save` or `instruction_delete` on `tasks`, `personal` or `style` in an account that keeps them as guided settings, or on `agents` or `default_tasks` there (Emm maintains those; the owner approves their updates in the app). | Call `instruction_settings()` to see the setting names, then `instruction_settings_update(changes=[...])` with named changes. Don't request the instructions window for a whole-document save — it would still be refused. Hand-editing is only for when the owner asks to switch to Advanced mode in the app. |
| `advanced_mode` | `instruction_settings_update` on an account whose owner keeps the instructions as documents (Advanced mode). | Read the document with `instruction_load` and save it whole with `instruction_save` (the instructions window rules apply). |
| `memory_write_locked`, `outputs_write_locked` | Instructions-Update Mode is open, so memory or output writes are paused. The text carries the `url:` and `expires_at:` lines. | Surface the URL to the user and stop the write loop until the window closes. Don't retry. Reads remain available. |
| `agent_os_not_enabled`, `premium_required`, `suspended` | A gate on the account — the operation is blocked until the user changes a setting or plan. The text carries the `url:` line. | Surface the URL to the user and stop. Don't retry. |
| `system_type_readonly` | You tried to write to a system-managed memory type (e.g. `memory_requests` is owned by `work_on_task`). | Switch to the tool on the `owner_tool:` line. Don't retry the generic write. |
| `slug_exists` | Output category + folder + slug collision, from `output_create`, `output_update` or `output_move`. The existing record's id is on the `existing_id:` line. Two documents at one path is the one state that makes a document unreachable, so every write path refuses it. | Pivot to `output_update` using that id, or pick a different slug or folder. Inside an `output_move` batch this is a **per-item** result — the other items still moved, so re-issue only the ones that failed. |
| `duplicate_memory` | Soft duplicate-detection blocked a `memory_save` (similarity ≥ ~0.88). The prior record's id is on the `existing_id:` line; for a batch where every item was a duplicate, each item's id is on its `items[i]=duplicate→<id>` entry. | Pivot to `memory_update(id=<existing_id>, content=…)` rather than retrying the save. |
| `explicit_run_id_required` | `agent_run_complete(last_open=true)` found more than one run open account-wide, so "the last open run" is ambiguous. The candidates are listed in the text, one per line. | Pass the `run_id` from your own `agent_run()` response. The by-id close is exact and identity-independent. Never guess — the other run is someone's live cycle. |
| `not_found` | `memory_get` / `memory_update` / `memory_delete` / `output_get` / `output_edit` / `output_move` / `output_delete` / `output_delete_category` was called with an id that doesn't exist. The `tool:` line names the recovery search tool. A `memory_delete`, `output_delete`, `memory_move` or `output_move` batch in which **every** id is missing is also `not_found`, with each id named in the text (`<id>=not found`); a delete or move batch that changes nothing for mixed reasons is `tool_error`, and its text names each id's outcome (`not found`, `changed`, `failed`, `unconfirmed`, `duplicated`, with where any copy is) so you can act per id. The one zero-change batch that stays a success is an `output_move` whose every document is already at its destination (`unchanged`). | Pivot to that search tool (`memory_search` or `output_search`) or `output_categories`; don't retry the id. |
| `not_a_peer` | You passed an OAuth2 session's id to `describe_method` / `execute_method`. Methods run only on trusted peers, not on the AI sessions sharing this account. | Call `list_connections` and pick a `peer_id` from `peers`, not `oauth2_sessions`. |
| `revision_conflict` | An `output_update` / `output_edit` / `output_delete` / `output_move` whose `if_match` no longer matches — the item changed since you read it, and the write was refused rather than clobbering the other edit (the `current_revision:` line carries the current `updated_at`) — or a **per-item** result inside a `memory_delete` or `output_move` batch, where that item changed between the batch's read and its write. Note `output_move` can report this with **no `if_match` supplied at all**: another writer touched the document mid-move. The move rolled back, so the document is untouched. | For an `output_edit` sent with `if_match`: read the part you are changing (`output_get(id, section=[…])`, a plain `output_get` when the document has no headings, the end of it for an append) and, only when your change is not there, send the call again with the `updated_at` it shows (an insert, a wrapping replacement or a pinned append would otherwise be applied twice). For an `output_edit` sent without `if_match`, which lost a race inside the call: send the same call again; each `old` is checked against the current text and a repeated append is reported as no change. For another output write: re-read with `output_get`, merge into the current body, retry with the revision it returns. For a move: just re-issue it. For a batch item: re-read it and re-issue only that id. Note this is *not* `not_found` — the item still exists, so don't go searching for a replacement. |
| `anchor_not_found` | An `output_edit` `old` text is not in the document. The text names the edit (`edits[1]`), shows the start of its `old`, and may add a `nearest_match:` line, the stored line nearest to it. Nothing was written. | Read the part of the document you mean, copy the text from it, and send the call again. A `nearest_match:` line says where to look; it is shortened, so don't copy from it. If the message says the text is in the document but starts or ends inside a character, extend `old` to the whole word or character on that side. If the message says the replacement is already in the document, the change may have landed earlier (from a call of yours or from someone else): read that part to confirm before retrying. |
| `anchor_ambiguous` | An `output_edit` `old` text appears more than once. The `match_count:` line says how many times and the text lists the lines. Nothing was written. | Extend that `old` with the text around the occurrence you mean until it is unique, then send the call again. |
| `section_not_found` | An `output_get` `section` names no heading in the document. The text lists up to five closest headings. | Retry with one of the listed headings, or call `output_get(id, outline=true)` to see every heading with the array to pass. |
| `edits_overlap` | Two edits in one `output_edit` call change the same text. Nothing was written. | Merge them into one edit and send the call again. |
| `run_not_open` | A write carried a `run_id` whose run is closed or past its 3-hour deadline. This check runs *before* the write, so if you're seeing this, the run really is no longer live — a run your own agent closed moments ago gets a short grace window and never reaches this error. | The `run_id` argument is optional: re-issue the same call **without** it and the write goes through. Do not retry with this `run_id`, and do not start a new run with `agent_run()` to recover — only start one if you actually want a new cycle. |
| `batch_too_large` | A batch call carried more items than the tool accepts. Every batch tool caps at **25** items (`memory_save`, `memory_delete`, `memory_move`, `output_move`, `output_delete`, and the `edits` of `output_edit`), and the cap is enforced *before* any work — nothing was saved, deleted or moved. The `max_items:` and `received:` lines carry the cap and what you sent. | Split your list into chunks of at most `max_items` and call the tool once per chunk. The whole batch was rejected, so re-send every item — don't assume a prefix went through. |
| `storage_unavailable` | `output_create` couldn't allocate an id under a sustained burst of concurrent creates against the same account. Nothing was written. | Retry the same call. This isn't a bad argument — the request is fine, storage was just briefly contended. |
| `wrong_id_family` | An `output_get` / `output_update` / `output_edit` / `output_delete` / `output_move` was handed a memory id (`memory_food:1`) instead of an output id. The `tool:` line names `memory_get`. | Call `memory_get` with the same id; don't search outputs for it. |
| `ambiguous_id` | A bare/legacy output id (`44`, or a pre-counter id that predates permanent ids) matches more than one document. The candidates are named in the text. | Address it by its full `<category>:<id>` form — the one on the candidate you mean — instead of the bare number. Don't guess. |
| `task_not_found` | `agent_run_task(task=…)` matched no task in `## Tasks this run`. The text names the available tasks and, if there's a close match, a `Did you mean` suggestion. | Retry with the exact name printed in `## Tasks this run` (or the suggested one). Don't guess a name that wasn't listed. |
| `task_ambiguous` | `agent_run_task(task=…)` matched more than one task — the same name in both `default_tasks` and a custom definition, or two custom headings that normalise alike. Every matching heading is named in the text. | If the headings differ (a parenthetical distinguishes them), retry with the **full heading** of the one you mean, exactly as printed. If the headings are identical (a default and a custom task, or two customs, sharing the exact same name), no retry can tell them apart — the message says so; ask the user which one they mean rather than guessing. Never pick one for the user. |
| `not_elevatable` | `output_update(dashboard=true)` on a document that can't be shown on the Dashboard: anything in the `log` category, or an `actions` or `space` document slugged `dashboard` or `actions`. | Don't retry; leave that document off the Dashboard. |
| `dashboard_limit` | `output_update(dashboard=true)` when three other documents are already on the Dashboard. | Tell the user. Only remove another document from the Dashboard if they ask you to. |
| `title_invalid_type`, `title_too_long`, `short_description_invalid_type`, `short_description_too_long` | An `output_create` / `output_update` `title` or `short_description` was the wrong type or too long; the message says which and the limit. | Fix that argument as the message says and retry once. |
| `category_excluded` | `output_search` was given the `log` category, which is excluded from search. The `tool:` line names `output_list`. | Use `output_list(category="log", recency_days=N)` instead. |
| `category_unknown` | `output_search` was given a category that doesn't exist. The message lists the searchable categories; a custom category becomes searchable once it contains items. | Retry with one of the listed categories, or call `output_categories()`. |
| `category_is_standard` | `output_delete_category` on a built-in category. Only custom categories minted by `output_create` can be removed. | Don't retry; standard categories can't be deleted. |
| `category_not_empty` | `memory_delete_type` or `output_delete_category` on a category that still holds items. The `tool:` line names the tool that clears it (`memory_delete` or `output_delete`). | Tell the user. Only delete the items first if they asked you to remove them; then retry. |
| `tool_error` | The tool refused the call, and the message says why — a malformed or missing id, a missing argument, an unknown category, a name that already exists. | Fix what the message names and try again. Don't retry the call unchanged; if the message doesn't tell you what to change, log it with `status: failed` and continue. If the message says not to retry — an `execute_method` call that failed after it was sent, where the peer may already have acted — don't. If it says to wait and retry with the same parameters — a peer that rate-limited the call and did not run it — wait, then retry once unchanged. |
| `invalid_params` | The arguments were rejected as sent — missing, the wrong type, or in conflict with each other. | Fix the arguments. Don't retry the same call unchanged. |
| `method_not_found` | The call named something the server doesn't have. | Check the name against your loaded tool list. Don't retry. |
| `internal_error`, `server_error` | Something failed on the server side. | Retry once. If it fails again, log it to the run log with `status: failed` and continue to the next task. |

Batch results can also carry per-item codes that are not tool errors of their own. `move_unconfirmed` (from `output_move`, or `delete_unconfirmed` from `output_delete`) means a write raised and whether it landed could not be checked: look the document up — and, for a move, where `copied_to` says it may have landed — before retrying, and delete nothing: with a permanent ID both places hold the same ID, so a delete removes the document itself. `move_partial` means the document really is in two places. `copied_to` is normally just a category (`space`): both places then hold the **same** id, so delete nothing — a delete by that id removes the document, not the spare. It reads normally from its new category; the leftover is hidden and only keeps its old path reserved, so tell the user if that path is needed again. Only when `copied_to` is a full id of its own (`space:51`) is there a separate copy to remove once you have checked it. Never answer either with a blind retry.

Anything that is not a tool error at all — an auth failure or a network error before the tool ran — is handled the same way as `server_error`: retry once, and if it fails again, log it with `status: failed` and continue. Don't halt the run.

> URL → MCP-tool translation lives in SKILL.md and in `how_to_use()`; the agent-run pre-authorisation rule lives in `agents`, in the `agent_run` bundle itself, and under *Working habits* above. See SKILL.md / `how_to_use()` for the full tables.
