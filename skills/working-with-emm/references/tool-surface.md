# Tool Surface — parameter detail & status() field reference

The full tool list with parameter shapes and batch limits, plus the
`status()` return fields SKILL.md's Quick Reference and Session check don't
spell out. **The live tool schema always wins over this file** — it is a
convenience index, not the contract; if a schema and this page disagree,
follow the schema.

## Contents

1. [Memory](#memory)
2. [Outputs — the Wiki](#outputs--the-wiki)
3. [Dashboard Summary block](#dashboard-summary-block)
4. [Instructions](#instructions)
5. [Recurring cycle](#recurring-cycle)
6. [One-off task drain](#one-off-task-drain)
7. [Shared Memories](#shared-memories)
8. [Remote Actions](#remote-actions)
9. [status() — non-safety fields](#status--non-safety-fields)

---

## Memory

- `memory_search()` — keyword/semantic search; supports `last_n`, `recency_days`, `include_remote`. `include_remote=true` requires the once-per-conversation user ask — see [shared memories](shared-memories.md).
- `memory_get()` — retrieve memory details by ID; `id` for one, `ids=[…]` for a batch.
- `memory_save()` — store new memories; `content` for one, `items=[…]` (up to 25) for a batch, auto-categorized. `preview=true` to preview before writing.
- `memory_update()` — change one memory's content by ID. **Single `id` only** — there is no batch form; update several by calling it once per memory.
- `memory_delete()` — remove by ID; `id` for one, `ids=[…]` (up to 25) for a batch.
- `memory_move(id, target_type)` — move a memory to another category; pass `ids=[…]` (up to 25) to move several to the same target in one call. Each item gets a new ID; Emm rewrites inline **canonical** references (`work:40`, wiki links, app URLs) across memories, outputs and instructions, and the response carries the old → new ID mapping. The response also lists documents that mention a moved ID in **free text** (e.g. "memory 40"), which the rewriter cannot safely touch — fix those by hand. Write cross-references as canonical tokens (`memory_work:40`) from the start so future moves keep them in sync.
- `memory_types()`, `memory_create_type()`, `memory_delete_type()` — manage categories.
- `how_to_use()` — personalised account snapshot for clients without this skill. With the skill loaded, call it only when the user asks for the tour.

## Outputs — the Wiki

- `output_search(query, category?, limit?)` — hybrid semantic + keyword search across categories. Excludes `log` — use `output_list(category="log", recency_days=N)` instead.
- `output_list(category)` — list items in a category; each row shows the document's `size` in characters.
- `output_get(id="<category>:<id>")` — fetch one item with full body. The header shows `updated_at` and `size`. For a long document pass `outline=true` to list its headings (each row is the array to pass, with its size and line) instead of the body, then `section=["Heading", …]` to read just that part; a path matching several sections returns every one, labelled, and a path matching none is refused with `section_not_found` and the closest headings.
- `output_create(category, slug, title, content, short_description, ...)` — create.
- `output_update(id="<category>:<id>", ...)` — modify. Pass the `updated_at` you read as `if_match` and a write that lost the race is refused with `revision_conflict` (carrying the current revision) instead of clobbering the other edit. Pass `dashboard=true` / `false` to add or remove this document from the user's **Dashboard page** (a distinct feature from `output_dashboard()`'s singleton actions dashboard, below) — rejected with `not_elevatable` for the `log` category or for any `actions`/`space` document slugged `dashboard` or `actions` (the Actions tile itself), and with `dashboard_limit` once 3 other documents are already shown there (the user unticks one first). No-op in `preview` mode. **An elevated document is rendered by the same Summary-block reader the Actions tile uses, so give it a `## Summary` block** — see [Dashboard Summary block](#dashboard-summary-block) below. Note `output_create` does not take this flag: create first, then call `output_update` with the body again to elevate.
- `output_edit(id="<category>:<id>", edits?, append?, title?, short_description?, if_match?, preview?)` — change part of a document, or add to it, **without resending the rest**. `edits` is a list of `{old, new}` (at most 25 per call): each `old` must appear exactly once in the document (typographic differences such as curly quotes, dashes and non-breaking spaces are tolerated, and the result says when one was used), `new` replaces it, an empty `new` deletes it. `append` adds text to the end after a blank line (a list item continues a list, a table row a table). Put every change to one document in one call: they land together or not at all. An `old` that is missing is refused with `anchor_not_found` (nearest stored line shown), one that appears more than once with `anchor_ambiguous` (the lines it is on), two that change the same text with `edits_overlap`; nothing is written in any of those cases. The result shows each changed part, the new `updated_at` and the size, so **do not re-read the document**; `preview=true` returns the same without writing. Without `if_match`, an `append` whose lines the document already ends with is reported as "No change" rather than repeated (to add a line that really repeats the last one, pass `if_match`). Edits have no such check: a replacement whose `new` no longer contains its `old` is `anchor_not_found` when sent twice, but an insert (`old` → `old` plus a new row) or a replacement whose `new` still contains its `old` (a link or bold around it) is applied again. So when a reply is lost, read that part and, only if your change is not there, resend with the `updated_at` the read returns (it is refused with `revision_conflict` if the first call lands meanwhile). `slug`, `folder` and `dashboard` stay on `output_update`.
- `output_move(id="<category>:<id>", folder?, slug?, target_category?)` — relocate a document **without sending its body**. Use this, not `output_update`, whenever the body is not changing: `output_update` requires the full content, so re-foldering a set would pull every body through the conversation twice and a write cut short by a context limit stores a truncated document. Pass `ids=[…]` (up to 25) to send several to the same destination; `slug` and `if_match` name a single document and are refused alongside `ids`. Pass `folder` and `slug` together to re-folder and rename in one call; if the slug itself carries a folder, an explicit `folder` wins. The document's ID is **permanent**: a folder change, a rename and a `target_category` move all keep it, so `output:<category>/<id>` links keep resolving. The ID's prefix is the category the document was created in and `category` is where it is now, so after a move between categories the two differ — keep passing the ID you were given. Every entry in `moves[]` confirms it with `id_preserved: true`. A move also rewrites links in **other** documents' bodies that named the old path, to the canonical `output:<category>/<id>` form, so the next relocation has nothing to repair; free-text mentions it cannot safely touch come back in `prose_candidates` — each `{"id", "context"}`, with the sentence the name matched in, so fix only the ones that really mean the moved document — and links written with a name that matches more than one document come back separately in `ambiguous_links` — those are left alone deliberately, because guessing which document was meant would silently repoint the others.
- `output_delete(id="<category>:<id>")` — remove (rarely; prefer update). Pass `ids=[…]` (up to 25) to delete several at once; per-item failures are isolated, so one not-found doesn't abort the rest. Optionally pass the `updated_at` you read as `if_match` to have the delete refused (`revision_conflict`) rather than remove an item someone else has since edited — **single `id` only**, since a revision token describes one item; combining it with `ids=[…]` is rejected.
- `output_dashboard()` — return the id and URL of the singleton actions dashboard, if one exists. It never creates one; `agent_run()` recreates a missing dashboard (`mode="preview"` never does, since it never writes).
- `output_categories()` — list the categories that currently exist (defaults + any custom ones). Call before minting a new category to avoid near-duplicates.

## Dashboard Summary block

Anything shown on the Dashboard — the Actions tile and every document elevated
with `output_update(dashboard=true)` — is rendered by **one** reader. It looks
for a `## Summary` block at the top of the body, ending at the first `---`.
Without one, the tile shows the document's first few lines with all markdown
stripped: headings, bullets, checkboxes and links all collapse to plain text.
So a document worth elevating is worth giving a Summary block.

```markdown
## Summary

**Open** 4 · **Blocked** 1 · **Done this week** 6
**Top**: Confirm the vendor shortlist before Thursday's review
*Last run: 2026-09-09 06:10 UTC — 2 items closed*

---
```

Three optional lines, each recognised independently:

- **Counts** — two or more `**Label** number` pairs joined by ` · ` (middle dot,
  a space either side). No colon after the label. One malformed pair discards
  the entire line, so build it mechanically.
- **Top** — one line naming the single most important item. Must start
  `**Top**`. Rendered clamped to two lines, so keep it to one sentence.
- **Last run** — when the block was last rewritten. Must start `Last run`,
  italics optional, and its text may contain no `*` or `_`. Omit it for
  documents the user maintains rather than an agent.

The count labels are yours to choose. **`Top` and `Last run` are fixed
keywords** — `**Priority**:` renders nothing. The heading must be exactly
`## Summary`: level two, that capitalisation, nothing trailing.

When an agent maintains the document, rewrite this block **last**, after every
other edit to the body is final.

## Instructions

- `instruction_list()` — list installed instructions, incl. `maintained_by` (`emm`/`user`) and `update_available`.
- `instruction_load(name)` — load one by short name (`agents`, `tasks`, `default_tasks`, `personal`, `style`).
- `instruction_merge_preview(name)` — preview the 3-way merge for a doc with a pending update (or a self-diff if none). Call before saving an update.
- `instruction_request_update_window(reason)` — asks the owner to open Instructions-Update Mode; `reason` (required, one line) names the change and heads the Accept/Decline banner in their app. You cannot open it yourself. Check `status()` for an active `unlock_window` before saving, and stop if they decline. Never during an agent run.
- `instruction_settings()` — a guided account's settings (profile, voice, tasks in run order with notes and schedules) and the names `instruction_settings_update` takes. On an Advanced account it says so.
- `instruction_settings_update(changes, keep_window_open=false)` — named changes (`set_task`, `add_own_task`, `move_task`, `set_field`, `set_section`, …) applied together; rebuilds the affected documents and closes the window unless `keep_window_open`. Same gate as `instruction_save`.
- `instruction_save(name, content, ...)` — write a standing instruction (on a guided account, not `tasks` / `personal` / `style`: those are `guided_mode`). Pass `applied_update: true` when incorporating a reviewed update, or `apply_clean_merge: true` to accept a clean merge without re-sending the body. Needs Instructions-Update Mode open, except a brand-new empty account's first save (`lock_bypass: "empty_account"` in the response) — check that field, don't assume the window opened.
- `instruction_delete(name)` — remove a standing instruction.

## Recurring cycle

- `agent_run()` — cycle entry point. Returns a core (the `agents` brief, `tasks`, the `default_tasks` shared core, a generated `## Tasks this run` list, a dashboard pointer) plus a `run_id`; execute immediately.
- `agent_run_task(task, run_id=..., already_loaded=[...], fresh_context=...)` — pulls one task's own procedure and the instruction docs it declares, one task at a time as the cycle reaches it. Read-only; `run_id` is optional (fences the pull to a live run when passed).
- `agent_run_complete(run_id=...)` — call once the cycle finishes to clear the in-progress marker.

## One-off task drain

- `work_on_task()` — get one context-prepared one-off task; `list_only=true` to peek; `mark_done=true, task_id=ID` to close.

## Shared Memories

- `memory_search(include_remote=true)`, `list_connections()` — see who shares what.
- Ask the user once per conversation before searching remote memories. Remember the answer for the rest of that conversation; ask again next session. Attribute matches: *"Alice mentioned …"*

See [shared memories](shared-memories.md) for patterns.

## Remote Actions

- `list_connections()`, `describe_method()`, `execute_method()`.
- Confirm with the user before executing unfamiliar methods.

See [remote actions](remote-actions.md) for patterns.

---

## status() — non-safety fields

The run-lifecycle fields (`runs`, `mode`, `suggested_actions`, `unlock_window`)
are safety-relevant and stay documented in SKILL.md's Session check — read
them there. The remaining fields:

- `limits.memory_max_kb` — per-memory body cap (defaults around 400 KB). Check before attempting a large `memory_save`.
- `limits.outputs_per_category` — per-category soft cap (defaults around 500). The `log` category has a lower cap — `limits.outputs_per_category_log` (defaults around 100). Beyond either, suggest the user prune.
- `links.help_page` — absolute URL to the user's in-app help page (the user-facing companion to this skill's content). Give it to the user when they ask where to read more in the web app; don't try to fetch it yourself.
- `links.app_home` — absolute URL to the user's web app root. Use when the user asks to "open Emm" without a specific destination.
- `tools_recommended` — names of the Emm tools this skill assumes will be available. Treat it as an informational contract from the server, not a prescription to drive your MCP loader. If a name on the list isn't in your live tool list, your host will surface it when you actually need it (deferred-loading clients) or it really is unavailable; don't try to second-guess your platform's loading mechanism.
- `instructions_pending_updates` — names of Emm-maintained instruction documents with a newer template available (`null` when instructions aren't enabled). Non-empty means `suggested_actions` will carry an `apply_pending_updates` entry; in normal mode applying needs the unlock window (see `instruction_request_update_window`).
- `your_client_has_only_used_reads` — server observed your client only making read calls. If `true`, mention it to the user once: "I'm only seeing reads on this connection — if you intended writes, your MCP client may need permission adjustments."

`status().conventions` — display rules, link forms, attribution cap, search freshness — mirrors SKILL.md's Display Rules table live; see SKILL.md for the full table with examples.
