---
description: Move (re-folder, rename or recategorise) one or more Emm AI wiki documents.
---

Use the `output_move` Emm AI MCP tool to relocate output documents: $ARGUMENTS

Use this and **never** `output_update` when the body is not changing — `output_update` requires the full content, so re-foldering a set of documents would pull every body through the conversation twice, and a write cut short by a context limit stores a truncated document. `output_move` sends no body in either direction.

Each ID is the `<category>:<id>` string shown on the item (e.g. `space:48`). Pass a single `id="<category>:<id>"`, or `ids=["space:48", "space:49"]` (up to 25) to send several to the same destination; `slug` (a rename) and `if_match` name one document and are refused alongside `ids`. `folder="…"` re-folders a `space` document, `folder=""` moves it to the top level, and `target_category="…"` moves it to another category. `folder` and `slug` combine, so one call can re-folder and rename; a slug that carries its own folder loses to an explicit `folder`.

A document's ID is permanent: a folder change, a rename and a move to a different `target_category` all keep it, and each entry of `moves` confirms that with `id_preserved: true`. The ID's prefix is the category the document was created in, while `category` says where it is now — after a move between categories the two differ, so keep using the ID you were given. The move also repairs links in other documents' bodies; anything it could not resolve comes back in `prose_candidates`, each with the sentence it matched in (`context`) — the scan keys on the document's name as a whole word, so for a generic name most hits are not references to it; read each `context` and offer to fix only the ones that really mean this document. Links written with a name matching more than one document come back in `ambiguous_links` — those were left alone on purpose; ask the user which document was meant rather than guessing. Confirm the destination with the user if it's ambiguous.
