# Changelog

All notable changes to the **Working with Emm AI** skill (ClawHub slug: `working-with-emm`; previously published under `managing-actingweb-memory`).

## [2.11.0] — 2026-10-06

### Added

- [Available Tools](#available-tools): `output_edit` changes part of a wiki
  document, or appends to it, without resending the rest: each `old` text must
  appear exactly once, the result shows what changed, and no re-read is needed.
- [Available Tools](#available-tools): `output_get` shows a document's `size`,
  lists its headings with `outline=true`, and reads one part with `section`;
  `output_list` and `output_search` show each document's `size`.
- `references/mission-control.md`: remedy rows for `section_not_found`,
  `anchor_not_found`,
  `anchor_ambiguous` and `edits_overlap`, and the `edit:`, `match_count:` and
  `nearest_match:` lines they carry; the `revision_conflict` row covers
  `output_edit`.

## [2.10.0] — 2026-09-29

### Added

- [Instructions](#instructions): guided accounts. `instruction_settings` reads
  the settings `tasks`, `personal` and `style` are built from, and
  `instruction_settings_update` changes them with named changes (a task's
  note, schedule, order, an own task, a profile field); `instruction_save`
  refuses those three documents there (`guided_mode`). The improvement loop
  and `/emm:instructions-edit` / `/emm:instructions-show` use the guided
  tools on those accounts.

### Changed

- `references/mission-control.md`: the `instructions_locked` row shows
  `instruction_request_update_window` with its required `reason`.
- [Instructions](#instructions): the legacy `skills` document is no longer
  listed; Emm's migration removes an unedited copy (an edited one stays the
  user's own document).
- The draft step works with any mail connector: an approved draft is
  created in the user's mail account (Gmail, Microsoft 365 or another), and
  the user sends it from there.

- [The Three Pillars](#the-three-pillars): the Instructions row no longer
  calls the documents "authoritative" or "standing orders from the user"
  (two of them start from an Emm-maintained baseline, see
  [Instructions](#instructions)). It says your system prompt and the user's
  messages outrank them.
- The same stance everywhere else the skill and plugin describe the
  instructions: a Task Builder prompt is "the user's description of what they
  want" (not authoritative), the mission-control card no longer calls the
  `agents` brief authoritative, dashboard `>` comments are the user's direction
  for that item (not instructions that change anything else), and the plugin
  README and `/emm:dashboard` say the same.
- [Agent Runs](#agent-runs-the-recurring-cycle): the worked example and the
  core description match what `agent_run` returns now (the core and the task
  list, no size figure).
- [Available Tools](#available-tools): the table lists every tool, adding
  `agent_run_task`, `status` and `output_delete_category`.
- [Core rules](#core-rules) and the mission-control card: one email-draft
  lifecycle, `pending | approved | discard | done`. An approved draft becomes
  a Gmail draft and the user sends it; the agent never sends.
- The mission-control card points to SKILL.md and `status().conventions` for
  link forms, and describes the trimmed `agents` brief (voice, policy rules,
  run log).
- `instruction_request_update_window` takes a required `reason` naming the
  change; it heads the owner's approval banner. Never request the window
  during an agent run.

### Removed

- The `quick` mode of `agent_run` is gone from the skill and the
  `/emm:run` command; the modes are `full` and `preview`.

## [2.9.0] — 2026-09-27

### Added

- A new tool, `agent_run_task`, pulls one task's own procedure and the
  instruction docs it declares, one task at a time as the cycle reaches it.
  [Agent Runs](#agent-runs-the-recurring-cycle) documents the pull protocol,
  `already_loaded`, and the `fresh_context` sub-agent hand-off.

### Changed

- `agent_run()` now returns a **core** (~30 KB) instead of the full bundle
  (~145 KB on a large account): the `agents` brief, the user's `tasks` doc
  (custom procedures pointed at, not inlined), the `default_tasks` shared
  core, a generated `## Tasks this run` list, and a dashboard pointer.
  `personal`, `style`, `skills` and every task's procedure now arrive via
  `agent_run_task` instead of up front.
  [Agent Runs](#agent-runs-the-recurring-cycle) rewritten accordingly.
- Modes table shrinks to `full` / `preview`; `quick` is a retired alias that
  runs identically to `full` — pull individual tasks via `agent_run_task`
  instead of asking for a narrower mode.
- [Instructions](#instructions): `skills` is documented as legacy — no
  longer installed for new accounts, existing copies left alone.

## [2.8.1] — 2026-09-25

### Changed

- [mission-control](references/mission-control.md): the error table gains
  `wrong_id_family`, returned when an output tool is handed a memory id;
  the row points at `memory_get`.
- [memory-best-practices](references/memory-best-practices.md): a memory
  containing only some of the query's words now counts as a keyword hit,
  scored by the fraction matched, so the score bands read the same for
  partial matches.
- The `agent_run` bundle is described at its real size (tens of thousands
  of tokens, not "several thousand"), with what to do when the host spills
  it to a file.
- Custom memory categories are described as shared with all connections by
  default, matching the reference file and the server; the "per-agent"
  wording was wrong.
- `agent_run_complete(last_open=true)` is described the same way in both
  places: it closes your own run when it is the single open one, and closes
  nothing when the only open run is another client's.
- [tool-surface](references/tool-surface.md) documents
  `status().instructions_pending_updates`.

### Fixed

- The `/emm:memory-new`, `/emm:memory-context`, `/emm:memory-extract` and
  `/emm:memory-news` commands now spell out the tool calls to make, with the
  matching MCP prompt as an optional shortcut. They used to depend on the
  prompt alone, which a tool-only client cannot invoke.

## [2.8.0] — 2026-09-23

### Changed

- **The skill fires on the user's words, even without "Emm".** Its
  description now also covers wanting something written up and kept,
  asking whether there is anything you should be doing for them, and
  asking what you know about them.
- **Document or chat answer?** Under Outputs: when the user wants
  something kept, edited later or returned to, make or update an output
  and hand back the link from the create result; a one-shot answer stays
  in chat.
- One-off tasks: "anything you should be doing for me" and "what did I
  leave you" join the phrases that mean `work_on_task`, in the skill and
  in [mission-control](references/mission-control.md).
- [mission-control](references/mission-control.md) gains **Working
  habits**, the etiquette `how_to_use()` no longer carries (status once
  before the first Emm call, no per-write permission asking in a run, no rehashing
  the bundle, working the Instructions-Update checklist in order).
  [task-builder](references/task-builder.md) notes the empty-queue reply
  carries the One-off tasks link.
- **Prompt-audit cleanup.** `status()` is called once before the first
  Emm call in a conversation, not in every conversation; the memory-search
  trigger gains a when-not line (general coding, factual lookups) in place
  of its open-ended catch-all. The "Critical Rules" section is now "Core
  rules", and the "Don't preview, don't partial-run" rule is "Run to
  completion" so it no longer reads as a ban on `agent_run(mode="preview")`.
- References now agree with SKILL.md: relevance-score thresholds in
  memory-best-practices, `how_to_use()` guidance in tool-surface, and the
  remote-method confirmation rule. `instruction_request_update_window` is
  listed with the other Instructions tools, and the output-category count
  reads 8.
- Removed history notes the reader never needs (the pre-2.2.0 `runs.open`
  shape, the task lease's old behaviour) and a line steering reasoning depth.
- **Tool descriptions now state safety and cross-tool rules as facts
  instead of instructions.** The consent rule on `memory_search`
  (`include_remote=true`), the confirmation rule on `execute_method`, and
  the once-per-run rule on `agent_run_complete` still live in every tool's
  description — nothing here depends on the skill for safety — but the
  etiquette around them (how to phrase the consent question, when a
  re-confirmation isn't needed, not surfacing routine outcomes to the
  user) moved here: [shared-memories](references/shared-memories.md),
  [remote-actions](references/remote-actions.md) and
  [mission-control](references/mission-control.md).
- `output_dashboard()` no longer creates the actions dashboard on its
  first call (it became read-only in the server); the skill's references
  and command are updated to match — a missing dashboard is recreated by
  the next `agent_run()`, not by calling `output_dashboard()` again.

## [2.7.0] — 2026-09-21

### Changed

- **An output's ID is now permanent.** `output_move` with a
  `target_category` used to mint a new ID in the destination; it now keeps
  the ID the document was created with, exactly as a folder change or a
  rename always has. The ID's prefix is the category the document was
  created in and `category` says where it is now, so after a move between
  categories the two differ — keep passing the ID you were given. Every
  entry in `moves[]` still carries `id_preserved`, which is now `true` on
  every move. `memory_move` is unchanged: a recategorised memory still gets
  a new ID. Updated in the routing table, the move guidance and the
  [tool surface](references/tool-surface.md).
- **`move_partial` recovery no longer says to remove a copy.** With a
  permanent ID both places hold the *same* ID, and a delete by that ID
  removes the document rather than the spare. The
  [error table](references/mission-control.md) now says to delete nothing:
  the document reads normally from its new category, and the hidden
  leftover only keeps its old path reserved. A separate copy to remove exists only when `copied_to` is a full
  ID of its own.

## [2.6.2] — 2026-09-17

### Added

- **`storage_unavailable` and `ambiguous_id` error codes**, in the error
  remedy table in [mission control](references/mission-control.md).
  `storage_unavailable` means `output_create` could not allocate an id while
  storage was briefly contended and wrote nothing; retry the same call.
  `ambiguous_id` means a bare or legacy output id matches more than one
  document; address it by the full `<category>:<id>` of the candidate you mean.

## [2.6.1] — 2026-09-10

### Added

- **`output_update(dashboard=true|false)`.** Add or remove a document from
  the user's Dashboard page — a distinct feature from `output_dashboard()`'s
  singleton actions dashboard. Same access as any other content edit (no
  extra owner check); rejected with `not_elevatable` for the `log` category
  or the actions-dashboard document itself, and with `dashboard_limit` once
  3 other documents are already shown there. No-op in `preview` mode. See
  [tool surface](references/tool-surface.md). `not_elevatable` covers any
  `actions` or `space` document slugged `dashboard` or `actions`, not only
  the actions dashboard itself; and `output_create` does **not** take this
  flag, so elevating a new document is a create followed by an
  `output_update` carrying the body again.
- **The `## Summary` block format**, in `references/tool-surface.md`. Every
  Dashboard tile — the Actions tile and every elevated document alike — is
  rendered by one reader that looks for this block and falls back to
  markdown-stripped plain text without it. The format was otherwise only
  discoverable from the web app's help page, so an agent could elevate a
  document with no way to learn how to make it render well. Records the
  strict parts: `Top` and `Last run` are fixed keywords while the count
  labels are free, one malformed count pair voids the whole line, and the
  heading must be exactly `## Summary`.

### Changed

- **Tool errors now arrive as errors.** A failed call returns `isError: true`
  with readable text that starts with `❌`, puts the fields you need
  (`url:`, `existing_id:`, `tool:`, `max_items:`, …) on their own lines, and
  ends with `(error code: <code>)`. Errors used to arrive looking like
  successes, carrying a raw JSON-RPC dict. The JSON-RPC outer codes
  (`-32099` … `-32091`) and the `action_required.*` fields no longer reach
  the agent.
- **The error-handling table keys on that code string**, with a row for
  every code the server emits — now including `not_a_peer`,
  `not_elevatable`, `dashboard_limit`, the output-argument validation codes,
  and the plain `tool_error`, `invalid_params`, `internal_error`, `method_not_found` and
  `server_error`.
- **Read tools return JSON, not a Python repr.** `memory_search`,
  `memory_get`, `memory_types`, `list_connections`, `describe_method` and
  `execute_method` put JSON in the text block and the same payload in
  `structuredContent`.
- **A failed `execute_method` is never a retry cue.** A timeout or a 5xx can
  land after the peer acted, so the error says not to retry unless the user
  confirms; `references/remote-actions.md` and the `tool_error` row say the
  same. Any 2xx from the peer is now success, and a peer that rate-limits
  the call says to wait and retry unchanged rather than to fix the parameters.
- **A batch `memory_save` that stores nothing still lists every item.** When
  all items are duplicates it is a `duplicate_memory` error with each
  `items[i]=duplicate→<id>`; when items raised or timed out it is an
  `internal_error` carrying the reasons, and when any item was refused (access
  denied) it is a `tool_error`. It used to be a bare "Failed to store
  items".
- **`output_move`'s `prose_candidates`** entries are `{"id", "context"}`
  objects carrying the sentence the name matched in. Fix only the mentions
  that really mean the moved document, rather than every hit by hand.

### Fixed

- **`references/custom-categories.md` said the storage form is refused.** It
  claimed `memory_create_type` and `memory_delete_type` return an error for
  `memory_recipes`; both accept it and add the prefix only when it is
  missing. The reference now says the storage form is accepted and the
  short form preferred.
- **`references/mission-control.md` published a stale actions-dashboard
  structure** — wrong top heading, `Inbox / pending review` and
  `Actions taken` sections the seed does not use, and no `## Summary` block
  at all. An agent following it produced a document that rendered as
  flattened plain text. Replaced with the seeded structure plus a cross-link
  to the Summary-block format.

## [2.6.0] — 2026-08-26

### Added

- **Account facts under `how_to_use()`'s Account snapshot.** Memory and
  instruction counts, one-off task counts, last completed run, and connected
  clients render as facts — never a verdict, never a rubric to grade the
  account against. What "set up" means lives on the public setup guide, the
  one-off surface a user actually reaches for that question, sized to three
  tiers the assistant checks against: connected + personalised (required),
  runs on a schedule, and smart things to do. `status()` gains `runs.recent`
  (the last 5 runs, timestamps only) and carries `started_by_client_id` /
  `_description` on `runs.last_completed`, plus `links.setup_guide`.
- **`instruction_save` empty-account exemption.** A brand-new account with
  no memories, outputs, or edited instructions may accept its very first
  `instruction_save` without Instructions-Update Mode being open — check
  the response's `lock_bypass` field rather than assuming the window
  opened; the exemption self-expires on the first write of any kind.
- **`client_context` on `status()`.** Pass `{skill_name, skill_version}` on
  the once-per-session call so the server compares versions for you —
  `status().skill.behind` (`true`/`false`/`null`). See [Session
  check](#session-check-do-this-first).
- **Setup reference: connection is the user's action.** The skill never adds
  the MCP server, edits config, or runs the OAuth helper itself — it relays
  the exact values and hands the user the command to run. No per-app UI
  steps; the setup guide works by outcome and by the app's own docs, which
  move faster than a page can be re-verified.

## [2.5.0] — 2026-08-21

### Added

- **`output_move` — relocating a wiki document without sending its body.**
  `output_update` requires the full content, so re-foldering a set of
  documents meant reading every body out and writing it back: a large job did
  not fit, and a write cut short by a context limit stored a truncated
  document rather than failing. `output_move` carries path metadata only, in
  both directions, one document or up to 25 per call. The ID guarantee is
  stated per branch — a folder or slug change **within** the document's own
  category keeps the ID, `target_category` mints a new one, and every entry in
  `moves[]` says which happened via `id_preserved`. Links in *other*
  documents' bodies that named the old path are repaired to the canonical
  `output:<category>/<id>` form, so the next relocation has nothing left to
  repair; free-text mentions the rewriter cannot safely touch come back in
  `prose_candidates`.

- **`batch_too_large` (`-32091`) in the error-code table.** Every batch tool
  (`memory_save`, `memory_delete`, `memory_move`, `output_move`,
  `output_delete`) now enforces a 25-item cap server-side, rejecting the whole call before any
  item is written — previously the declared limit was advertising only, and an
  oversized batch simply ran. The new row states the cap, that nothing was
  applied, and that recovery is to split and re-send *every* item rather than
  assume a prefix went through.

### Fixed

- **The documented outer-code range was stale.** Three places said errors run
  `-32099` through `-32092`; `batch_too_large` sits at `-32091`, so an agent
  hitting it would not have found it in the table.
- **`revision_conflict` is no longer described as outputs-only.** It can now
  arrive as a per-item result inside a `memory_delete` batch when that memory
  changed between the batch's read and its delete. The row explains the memory
  case separately and warns that this is *not* `not_found` — the item still
  exists, so searching for a replacement is the wrong move.
- **`memory_save`'s batch limit is now stated.** It was the only batch tool
  whose entry gave no cap, while `memory_delete`, `memory_move` and
  `output_delete` all said "up to 25".

## [2.4.0] — 2026-08-17

### Added

- **A troubleshooting section.** New "When Emm isn't responding" section with
  a direct answer for missing tools, structured tool errors, and the
  Claude.ai client-side approval-gate denial — previously reachable only by
  chance, from a deep link inside another section.
- **`references/tool-surface.md`** — the full per-tool parameter reference
  and `status()`'s non-safety field reference, split out so the always-loaded
  part of the skill stays a behavioural guide, not a schema dump.
- **`license: MIT-0`** and **`compatibility:`** frontmatter fields, matching
  how this skill is actually distributed (open on ClawHub, requires the Emm
  AI MCP connector).

### Fixed

- **The `instructions_locked` error row now points at
  `instruction_request_update_window()`** — the old wording told the agent
  to ask in chat, which never notified the account owner.
- **The `run_not_open` error row no longer tells the agent to start a fresh
  cycle.** The correct recovery is to re-issue the same write without
  `run_id` — starting a new cycle to recover from one surplus argument was
  needlessly expensive.
- **The `log` category's soft cap was misreported as 500** (same as every
  other category); it's actually 100. `status()` and this skill now agree.
- **`instruction_merge_preview`'s conflict response now warns against
  blind-saving the auto-draft** — the data-loss rule was previously only in
  this skill, not in the tool response itself.
- **`if_match` now sits next to the re-read rule it enforces.** "Re-read
  immediately before you update" named the habit but not the mechanism,
  which was documented only in the session-check block an agent may never
  revisit mid-run. The tool-surface reference also gained `if_match` on
  `output_update` and `output_delete` — including that `output_delete`
  accepts it with a single `id` only, never alongside `ids=[…]`.
- **`agent_run_complete`'s `last_open` parameter no longer contradicts its
  own tool description.** It claimed the refusal depended on which MCP
  session started the run; it actually fires whenever more than one run is
  open account-wide, whoever started them.
- **The error-envelope code range** quoted in the References table and in §1
  said `-32099` / `-32098` / `-32097`; the table it points at runs through
  `-32092`.
- **The tool reference promised a batch `memory_update` that doesn't exist.**
  Condensing two tools into one line extended `memory_delete`'s `ids=[…]` form
  to `memory_update`, which takes a single required `id` — so following the
  reference produced a call the server rejects. The two now have separate
  entries, and `memory_update` says single-`id`-only outright.
- **The link-form table was missing both memory forms.** Linking to a memory
  from an output body, or from an MCP response, was documented only in
  Display Rules as a worked example — so the table you'd actually consult to
  answer "what form do I use here?" had four of the six answers. It now has
  all six, and its Form column is consistently placeholder notation (the
  bare row reads `<category>:<id>` rather than `category:id`) with the worked
  example beside it.

### Changed

- **`Available Tools` is now a compact per-pillar table**; parameter detail
  moved to the new tool-surface reference.
- Trimmed internal repetition (the reinstall nudge, the `full_id`
  convention, the shared-memory consent rule) down to one canonical
  statement each, adding a dedicated Critical Rules row for shared-memory
  consent so it isn't lost in the process.
- **This CHANGELOG is now trimmed to the current version plus the last few**
  — full history moved to the ActingWeb repository (`docs/CHANGELOG-skill-archive.md`),
  out of the distributed bundle.
- **SKILL.md now has a size budget, enforced in CI.** This is the first
  release in the skill's recorded history that removes more than it adds.
  A ratchet in the ActingWeb repository's test suite caps SKILL.md just
  above its current size, so future growth has to clear the bar
  deliberately rather than accumulating unnoticed — which is what happened
  over the twelve preceding releases.

## [2.3.0] — 2026-08-03

### Fixed

- **`agent_run` returns its bundle again.** Tools now declare whether their
  answer is prose or structured data, instead of the server guessing. Some
  clients discard every text block whenever a response carries structured
  fields, so `agent_run` — whose entire payload is the standing-orders bundle —
  had been answering with nothing but a run id since the field was added.
  A plain `work_on_task()` retrieve was losing its task brief the same way.

### Changed

- **Where to read each answer.** `agent_run` and a plain `work_on_task()`
  retrieve are **prose**: read the response text; they carry no structured
  fields. `agent_run_complete`, `work_on_task(list_only=true)`,
  `work_on_task(mark_done=true)`, `status`, and every memory / output /
  instruction tool are **structured**: read `result.structuredContent`. Note
  the nesting is preserved as it always shipped —
  `result.structuredContent.output.id` for `output_create`, not
  `.structuredContent.id`.
- The `run_id` for `agent_run_complete` comes from the `agent_run` response
  **text** — the `**Run ID:**` preamble line, or the
  `agent_run_complete(run_id="…")` reminder at the end.

### Added

- **`instruction_save` reports two signals as fields**, not only in prose:
  `window_closed` (Instructions-Update Mode was turned off by this save) and
  `server_merged` (a clean 3-way merge was applied server-side). Both are
  always present, so a `false` is distinguishable from a missing field.

## [2.2.0] — 2026-07-30

### Changed — breaking

- **`status().runs.open` is now a list**, not a single object or `null`, and is
  accompanied by `runs.open_count`. Several runs can be open at the same time,
  so "the open run" no longer names one thing. Skills reading
  `runs.open.started_by_client_id` will get `undefined` rather than a wrong
  answer — reinstall to pick up the new shape.
- **The coordination rule is inverted.** Previous versions said *"don't start a
  competing run; talk to the user before forcing close."* Overlapping runs are
  now supported and expected: a scheduled Autopilot run and an interactive one
  can both be open. Proceed with your own run, expect shared surfaces (the
  dashboard, the wiki, the task queue) to change under you, and close only the
  run you started.
- **`agent_run_complete(last_open=true)` acts only when exactly one run is open
  account-wide.** If anything else is open it refuses with
  `-32095 explicit_run_id_required` and names the candidates — even when one of
  them looks like yours. The server cannot always tell two clients apart, and
  it will not guess with a live cycle at stake. Pass the `run_id` from your own
  `agent_run()` response: the by-id close is exact and never depends on caller
  identity. Treat `last_open` as a convenience for the single-run case.

### Added

- **`if_match` on `output_update` / `output_delete`.** Pass the `updated_at` you
  read and a concurrent edit is refused with `-32093 revision_conflict` —
  carrying the item's current revision — instead of being silently clobbered.
  Worth using on the dashboard, which more than one run may rewrite per cycle.
- **`run_id` on output writes and `work_on_task`.** Optional; a write carrying a
  run that has been closed or has expired is refused with `-32092
  run_not_open`. `agent_run` now returns `run_id` as a top-level field, so it
  need not be re-read out of the bundle — record it, because it is the reliable
  way to close your own run when several are open.
- **Tasks are leased on hand-out** (60 minutes). Two runs draining the queue get
  different tasks. `list_only` marks a claimed task `claimed: true` while still
  reporting it `ready`; `has_ready_task` counts only unclaimed ones. The server
  applies this on every hand-out, though it is a read-then-write check, so it
  narrows the duplicate window rather than sealing it.

### Fixed

- A matching `started_by_client_id` is **not** proof a run is yours: two
  sessions of one registered client share it and the server cannot tell them
  apart. Unless you hold the `run_id` from your own `agent_run()`, treat a
  same-client run as someone else's.
- Closing an already-abandoned run no longer flips it back to `done`.
- A run left open is swept to `abandoned` after a 3-hour deadline — there *is*
  now a server-side path that closes a run on its own, contrary to what earlier
  versions of this skill stated.
- The end-of-cycle order is stated consistently everywhere: run log, close,
  then dashboard and memory.

## [2.1.6] — 2026-07-06

Native self-improvement lifecycle. The self-review → standing-instruction
loop is now something the AI can complete on its own, at parity with the web
app — no more hand-holding through every step of applying a proposal.

The old "user-owned" vs "system-owned" framing is gone. Every instruction is
yours to edit; the only distinction left is **who ships baseline updates**,
surfaced as `maintained_by` (`emm` or `user`) on `instruction_list()` and
`instruction_load()`. `emm`-maintained docs (like `agents`, now on that
channel) show an "update available" flag and merge your edits in on apply —
they're never silently overwritten.

### Added

- **`instruction_merge_preview(name)`** — see the 3-way diff between your
  current version, your edits, and a pending baseline update before you touch
  anything. Returns a rendered diff plus the structured strategy
  (`clean` / `conflict`) and per-hunk conflicts.
- **One-call clean applies.** When a pending update merges cleanly,
  `instruction_save(name=..., apply_clean_merge: true)` applies it without
  re-emitting the body. On a conflict, resolve each hunk yourself and save with
  `applied_update: true` — the skill's new **Improvement lifecycle** section
  walks the find → route → apply → retire loop, and guards refuse a no-op or a
  blind conflict apply so you can't clear "update available" without actually
  merging.
- **`status()` improvement playbook.** In Instructions-Update Mode, `status()`
  now returns an ordered, tool-named checklist (review self-reviews → apply
  pending updates → rationalize tasks) plus an `unlock_window` block telling you
  how long the window has left and which writes are paused.
- **Deterministic custom tasks.** The `tasks` doc gained a real
  checklist → numbered-definition wiring for your own recurring tasks, matching
  how default tasks already work — tick a task on, and its procedure ships in
  the cycle bundle.
- **Instruction change history (web app).** Each instruction now records who
  changed it and how, with created/updated timestamps and a read-only view of
  the previous body — so "why did this change?" has an answer.

### Changed

- **Clearer update & reset UX (web app).** Emm-maintained and "Yours" badges,
  a non-destructive "a newer default is available — your version is untouched"
  framing, and an always-available per-doc "Reset to default" (confirm-gated),
  so you never have to reach for a reset-everything button to fix one file.
- **Skill-update reinstall nudge is channel-neutral** — it points you to
  reinstall however you originally added the skill (bundle, plugin, or
  registry) rather than naming one source.

## [2.1.4] — 2026-06-12

Added a rule to re-read each output or memory immediately before
updating it. A cycle can run for many minutes, and if you edit a
document in the web app partway through, the agent could otherwise
write back the older copy it loaded at the start and wipe your change.
The agent now re-fetches the current version right before saving and
merges its update onto your latest text — so concurrent edits no
longer collide. Matters most for the action list and rolling trackers.

Also made the "skill out of date" nudge channel-neutral: it no longer
tells you to reinstall "from ClawHub" specifically (that's only one of
several install paths — re-uploading the bundle, reinstalling the
plugin, or pulling from a skill registry all apply depending on how
you added it).

Full history before 2.1.4 lives in `docs/CHANGELOG-skill-archive.md` in the
ActingWeb repository — not shipped in this bundle.
