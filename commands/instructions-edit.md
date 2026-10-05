---
description: Guide me through editing an Emm AI standing instruction (unlock + save).
---

Help me edit a standing instruction in Emm AI. Argument: $ARGUMENTS (short instruction name, e.g. `agents`, `personal`, `tasks`).

Workflow:

1. Call `instruction_settings()`. If it reports guided settings and $ARGUMENTS is `tasks`, `personal` or `style`, this account edits those through its settings: ask me what to change, read the change back to me, call `instruction_request_update_window(reason="<the change>")`, and once the window is open apply it with `instruction_settings_update(changes=[...])` — then stop here. Otherwise call `instruction_load(name="$ARGUMENTS")` to fetch the current body and show it. Note `maintained_by` (`emm` = Emm ships and iterates this baseline; `user` = purely yours — either way it's yours to edit) and whether an update is available.
2. If an update is available and you want to incorporate it, call `instruction_merge_preview(name="$ARGUMENTS")` first to review the diff before proposing the new body.
3. Ask me what to change, and propose the new full body (don't write partial diffs — `instruction_save` overwrites the document).
4. Saving needs **Instructions-Update Mode** (a 60-minute window). If `status().mode` isn't `instructions_update`, call `instruction_request_update_window(reason="<the change>")` — it shows me an Accept/Decline banner headed by that reason. Check `status()` again on a later turn and proceed once the window is open; stop if I decline, and don't loop. An `instructions_locked` error carries the Instructions-page URL if I'd rather unlock it there.
5. Once unlocked and I confirm, call `instruction_save(name="$ARGUMENTS", content=…)` — add `applied_update: true` if this save incorporates a reviewed update — and report the new version.
