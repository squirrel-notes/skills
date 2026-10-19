# Working with notes

Notes are the user's own documents, in Markdown. Read before you write, and change only what you were asked to.

## Finding

- `list_notes`: titles, optionally by collection, tag or pinned. The usual starting point.
- `get_recent_notes`: what the user worked on lately, titles only.
- `search_notes`: matches titles. Fast.
- `search_note_content`: matches titles and full text. It decrypts every note, so it's slow on large libraries; use it only when a title search won't do.
- `get_note`: the full Markdown content and tag ids.

## Writing

- **`create_note`:** title plus Markdown. Omit `collectionId` to put it in the Scratchpad. Never use a smart collection's id.
- **`append_to_note`:** adds to the end. Best for logs, journals and running lists.
- **`update_note`:** replaces the title and/or content. Read the note first and keep everything you weren't asked to change.
- **`set_note_tags`:** replaces all tags. To add one, read the current `tagIds` and pass the merged list.
- **`move_note_to_collection`:** to organise notes. Pinning goes through `pin_note`.
- **`delete_note`:** moves a note to Trash for 30 days and revokes its public link. Only when the user asks.
- **The `markdown_capabilities` prompt** (an MCP prompt, not a tool): what renders: standard Markdown, including tables, and Mermaid diagrams (flowcharts, sequence, ER, Gantt and more). Read it before writing rich content, if your client offers prompts.

## Sharing

`share_note` publishes a public, passphrase-protected link (Pro only). Do it only when the user asks. Give them the link and passphrase, and remember that `revoke_share` exists.

## Notes and memories together

When a note holds durable facts, such as meeting decisions or a project plan's current state:
- save each fact as a memory with `sourceNoteId` set to the note;
- leave the narrative in the note;
- don't copy whole notes into memories.
