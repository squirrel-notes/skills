# Working with notes

Notes are the user's own documents, in Markdown. Read before you write, and change only what you were asked to.

## Finding

- `list_notes`: titles, optionally by collection, tag or pinned. The usual starting point.
- `get_recent_notes`: what the user worked on lately, titles only.
- `search_notes`: matches titles. Fast.
- `search_note_content`: matches titles and full text. It decrypts every note, so it's slow on large libraries; use it only when a title search won't do. It returns up to five matching lines per note with their line numbers, not whole notes: read around a match with `read_note`. Pass `includeContent: true` only when you need every matching note in full.
- `get_note`: the full Markdown content and tag ids. For a long note, start with the outline instead (see below).

## Writing

- **`create_note`:** title plus Markdown. Omit `collectionId` to put it in the Scratchpad. Never use a smart collection's id.
- **`append_to_note`:** adds to the end. Best for logs, journals and running lists. It also takes `expectedUpdatedAt`, like the other writes.
- **`update_note`:** replaces the title and/or content. Read the note first and keep everything you weren't asked to change. For a long note, change only the part you need instead (see below).
- **`set_note_tags`:** replaces all tags. To add one, read the current `tagIds` and pass the merged list.
- **`move_note_to_collection`:** to organise notes. Pinning goes through `pin_note`.
- **`delete_note`:** moves a note to Trash for 30 days and revokes its public link. Only when the user asks.
- **The `markdown_capabilities` prompt** (an MCP prompt, not a tool): what renders: standard Markdown, including tables, and Mermaid diagrams (flowcharts, sequence, ER, Gantt and more). Read it before writing rich content, if your client offers prompts.

## Long notes

A note over about 8,000 characters (about 2,000 tokens) is expensive to read whole and risky to rewrite whole: a retyped note can silently lose or change text far from your edit. Work on the part you need.

1. **`get_note_outline`** lists the headings with their line numbers and sizes, and the note's `updatedAt`. It returns no body text.
2. **`read_note`** reads one section by `heading` (add `occurrence` when the same heading text appears more than once), or a range with `startLine` and `endLine`. Lines come back numbered. When `truncated` is true, call again from `nextStartLine`.
3. Change only what you need:
   - **`edit_note`** replaces one exact passage. Copy `oldText` from what you read, with enough around it to occur exactly once. If the same text appears under several headings (a checklist item, a repeated line), pass `heading` too, and it is matched only inside that section. Use it to fix a line, insert next to a unique phrase, delete text or rename a heading.
   - **`replace_section`** rewrites everything under a heading, subsections included. The heading line stays.
   - **`insert_in_section`** adds lines under a heading when there is nothing unique to match, such as a new item under "## Todo". With `position` "end" they go after the heading's own text, before its first subheading; with "start", at the start of the heading's text, after the blank line under the heading. A new list item next to a list continues it.
   - **`delete_section`** removes a heading and everything under it, subsections included. If the same heading text appears more than once, pass `occurrence`. It cannot be undone, so use it only when the user asks.
   - **`insert_at_line`** inserts lines before a line number, for the rare addition nothing else can place. Give it the `line` you read, and `expectedUpdatedAt` is required, because line numbers go stale as soon as the note changes.
   - The section tools keep the note's own blank lines around a heading, so don't add blank lines at the start or end of `content` for spacing.
4. **Pass `expectedUpdatedAt`** on every write: the `updatedAt` from your last read or write of the note. If the note has changed since, nothing is written and you get its new `updatedAt`; read it again before retrying. A pin, tag or move also changes it, so a refusal right after one of those is expected. Each write returns the new `updatedAt` for your next one.

A refusal never writes anything. If `edit_note` says the text occurs more than once, add more of the surrounding text or pass `heading`; if it occurs zero times, read the passage again and copy it exactly (case, spaces, accents and line breaks all count).

Keep `update_note` for short notes, or when the user asks for the whole note to be rewritten.

## Sharing

`share_note` publishes a public, passphrase-protected link (Pro only). Do it only when the user asks. Give them the link and passphrase, and remember that `revoke_share` exists.

## Notes and memories together

When a note holds durable facts, such as meeting decisions or a project plan's current state:
- save each fact as a memory with `sourceNoteId` set to the note;
- leave the narrative in the note;
- don't copy whole notes into memories.
