# Working with notes

Notes are the user's own documents, in Markdown. Read before you write, and change only what you were asked to.

## Finding

- `list_notes`: titles, optionally by collection, tag or pinned. The usual starting point.
- `get_recent_notes`: what the user worked on lately, titles only.
- `search_notes`: matches titles. Fast.
- `search_note_content`: matches titles and full text. It decrypts every note, so it's slow on large libraries; use it only when a title search won't do. It returns up to five matching lines per note with their line numbers, not whole notes: read around a match with `read_note`. Pass `includeContent: true` only when you need every matching note in full.
- The title tools (`list_notes`, `get_recent_notes` and `search_notes`) download titles only, never note bodies. `search_note_content` reads the bodies in batches, so on a large library it takes several API calls; if it can't read every note, it fails and matches nothing, and trying again is safe. That includes one note it can't read in full (a long note stored in parts that fails its checks): the whole search is refused as `unreadable`, not just that note, and trying again won't help until the note is fixed in the app. Search by title, or read other notes by id, meanwhile.
- `get_note`: the full Markdown content and tag ids. For a long note, start with the outline instead (see below).

## Writing

- **`create_note`:** title plus Markdown. Omit `collectionId` to put it in the Scratchpad. Never use a smart collection's id.
- **`append_to_note`:** adds to the end. Best for logs, journals and running lists. It also takes `expectedUpdatedAt`.
- **`update_note`:** replaces the title and/or content. It doesn't check `updatedAt`: it overwrites whatever the note holds, including an edit the user made a moment ago, and notes keep no history. Read the note just before you call it, and keep everything you weren't asked to change. For a long note, change only the part you need instead (see below).
- **`set_note_tags`:** replaces all tags. To add one, read the current `tagIds` and pass the merged list. Tag ids come from `list_tags` (see [Organising](#organising)).
- **`move_note_to_collection`:** to organise notes. Pinning goes through `pin_note`.
- **Size limits:** a note's content can be up to 3,145,728 bytes of UTF-8 text (long content is stored in parts, below), and a title up to 12,000 bytes. A write that would pass either is refused with `too-long` or `title-too-long`, whose limit says which cap applied, and nothing is written; a note over 750,000 bytes can't be shared either (`too-large-to-share`). Put long material in more than one note.
- **Notes stored in parts:** content over 262,144 bytes is stored in encrypted parts, whichever client writes it, these tools included. Every note tool reads and edits it the same way, and it can grow to 3,145,728 bytes. If it can't be checked in full, the tool refuses with `unreadable` and changes nothing; tell the user rather than retrying.
- **Long content and a title:** `update_note` with content over 262,144 bytes and a title writes the content first, then the title. If only the title is refused, it answers `title-not-saved` with the HTTP `status`, and a `reason` such as `storage-full` when it is one: the content is saved, so fix the cause and retry with the title alone.
- **Another writer:** a tool that reads the note before it writes (`append_to_note`, `edit_note`, the section and line tools) writes only over what it read. If the user or another agent changed the note in between, it is refused with `write-conflict` and nothing is written: read the note again, then retry once.
- **Storage:** the account's storage counts its notes as well as its files, and notes in the Trash until it's emptied. When it is full, a write that would make the account bigger is refused with `storage-full` and nothing is written; a write that keeps or shrinks a note still works. Tell the user rather than retrying: they can delete notes or files, or empty the Trash.
- **Write rate:** creating and updating notes is limited to 300 a minute. A write refused for that (HTTP 429) wrote nothing: wait a minute before you try it again.
- **`delete_note`:** moves a note to Trash for 30 days and revokes its public link. Only when the user asks.
- **The `markdown_capabilities` prompt** (an MCP prompt, not a tool): what renders: standard Markdown, including tables, and Mermaid diagrams (flowcharts, sequence, ER, Gantt and more). Read it before writing rich content, if your client offers prompts. What it leaves out is under [What renders](#what-renders).

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
4. **Pass `expectedUpdatedAt`** to every tool that takes it: `append_to_note`, `edit_note`, `replace_section`, `insert_in_section`, `delete_section` and `insert_at_line`. Give it the `updatedAt` from your last read or write of the note. If the note has changed since, nothing is written and you get its new `updatedAt`; read it again before retrying. A pin, tag or move also changes it, so a refusal right after one of those is expected. Each write returns the new `updatedAt` for your next one. `create_note`, `update_note`, `pin_note`, `set_note_tags` and `move_note_to_collection` don't take it and never refuse a changed note.

A refusal never writes anything. If `edit_note` says the text occurs more than once, add more of the surrounding text or pass `heading`; if it occurs zero times, read the passage again and copy it exactly (case, spaces, accents and line breaks all count).

Keep `update_note` for short notes, or when the user asks for the whole note to be rewritten.

## Tasks and due dates

A Markdown checkbox in any note is a task, and the app's **Tasks** page gathers the open ones from every note. Give a task a due date by putting `@due(YYYY-MM-DD)` on its line:

```
- [ ] Submit tax return @due(2026-04-10)
- [ ] Write quarterly report @due(2026-04-30)
  - [ ] Gather sales data @due(2026-04-25)
```

- **The form:** `@due(` plus a four-digit year, two-digit month and two-digit day, plus `)`, such as `@due(2026-04-10)`. It's a calendar day, with no time or time zone. Anything else, such as `@due(10 April)`, `@due(2026-4-10)` or `(due 2026-04-10)`, is plain text and the Tasks page doesn't see a date. The app also reads the shorthand `@2026-04-10`, but write the long form.
- **Where it goes:** anywhere on the task's line, usually at the end. Use one per task: only the first counts.
- **What the Tasks page reads:** open tasks (`- [ ]`, or `*` or `+` in place of `-`) with some text after the box. It shows top-level tasks and one level of sub-tasks. Each shows its own date, but the page files a sub-task under its parent's date when it groups tasks by date. Tasks nested deeper, ticked tasks and lines inside a fenced code block don't appear.
- **Turn relative dates into a date.** For "by Friday" or "end of the month", work out the date from today's date for the user, and ask if you can't tell which day they mean. Don't add a due date the user didn't give or ask for.
- **To add a task,** use `insert_in_section` under the list's heading, with `position` "end" to add it after the last item when the list is the last thing under that heading; otherwise use `edit_note` to add the line after the last item. If the list is at the very end of the note, `append_to_note` works too, but pass `separator` as a single line break, or an empty one if the note already ends with a line break: by default it adds a horizontal rule (`---`) between the old text and the new, which splits the list in two.
- **To tick a task or change its date,** use `edit_note` on that line: replace `[ ]` with `[x]` to tick it, or change the date to reschedule it. The same task text can appear more than once, so pass `heading` or copy enough of the line to match it once. Completing a task on the Tasks page also ticks its sub-tasks; when you tick one with sub-tasks, tick those too unless the user wants them kept open. It also ticks a task once its last open sub-task is ticked, so when you tick that one, tick the parent too or ask. A sub-task left open under a ticked task becomes a top-level task.
- **To find tasks with dates,** use `search_note_content` with the query `@due(`, or `@due(2026-04` for one month, and `@20` for the shorthand. It returns 20 notes unless you raise `limit` (at most 50), so check `total`, and at most five matching lines per note, so read the rest of a note with `read_note`. It also returns ticked tasks, lines in code blocks and `@due(…)` on lines that aren't tasks, none of which is on the Tasks page: skip those.

A due date belongs to a task in a note. A memory's expiry is its `validUntil`, which is separate (see [writing](writing.md)).

## What renders

Besides standard Markdown, tables, task lists and Mermaid:
- **Footnotes** (`[^1]` and a `[^1]: …` line) render as numbered notes. `@due(…)` shows as a date wherever it's written outside code, but only a task's reaches the Tasks page.
- **Raw HTML:** only `<kbd>`, `<sub>`, `<sup>` and `<br>`, with no attributes. Any other tag shows as plain text.
- **Not supported**, so it shows exactly as typed: math (`$E=mc^2$`), emoji shortcodes (`:smile:`), wikilinks (`[[Wiki Link]]`) and hashtags (`#hashtag`). Use the note's tags instead of hashtags.
- **Images:** attach the file and embed it as `![name](attachment:<id>)` (see [Attachments](#attachments)). An image from an `https://` address waits until the user clicks to load it, and inside a table it shows only as a card in the editor; `http://` addresses and relative paths never load.
- **Mermaid:** a diagram that would fetch anything from another site, such as an image shape, an icon from an address or a CSS `url(…)`, isn't drawn and shows an error instead. Icons from the bundled packs (`@{ icon: "pack:name" }`) draw normally.
- **Templates:** there are no template tools, and `{{date}}` is filled in only when the app creates a note from a template. Write the real date.

## Organising

Ask before deleting or reorganising anything the user didn't mention.

- **Collections** are folders, and can nest. `list_collections` gives each one's `id`, `name` and `parentId`; smart collections aren't listed, and you can't put notes in them. `get_collection` reads one; `create_collection` (with `parentId` to nest it) and `update_collection` rename, move or recolour one. **`delete_collection` also deletes every collection inside it**, and moves all their notes to the Scratchpad; no note is deleted.
- **Tags:** `list_tags` gives each tag's `id`, `name` and `color`; use those ids with `set_note_tags` and with `list_notes`' `tagId` filter. `get_tag` reads one, `create_tag` makes one, and `update_tag` renames or recolours it. `delete_tag` deletes the tag, so it disappears from `list_tags`, but notes keep its id: `tagIds` can still hold it and the `tagId` filter still matches it. Before `set_note_tags`, drop any id that `list_tags` doesn't return.
- **Colours** are `#rrggbb` hex codes. The app's colour picker offers six: `#3b82f6`, `#8b5cf6`, `#ec4899`, `#f59e0b`, `#10b981` and `#06b6d4`, so use one of those unless the user asks for another.
- **Saved searches** store a query that the app runs: `list_saved_searches`, `create_saved_search`, `update_saved_search` and `delete_saved_search`. A query combines words (any of them matches title, text or attachment names), `"exact phrases"` (all must appear), `tag:name`, `collection:name`, `updatedafter:` and `createdafter:` with `YYYY-MM-DD`, `today` or `-7d`, and `has:attachment`. The MCP server stores a query but doesn't run it: to find notes yourself, use the search tools above.
- **Trash:** `delete_note` moves a note there. `list_trash` lists trashed notes and `restore_note` brings one back to its collection, without its public link. `permanently_delete_note` and `empty_trash` can't be undone: use them only when the user asks for exactly that.

### Attachments

- `list_attachments` lists a note's files, with names, types and sizes.
- `upload_attachment_from_path` uploads a local file, encrypted. The file must be under `SQUIRREL_UPLOAD_ROOT` (a comma-separated list of folders; by default `~/SquirrelUploads`, a private folder in the user's home that the server creates), and a link that leads outside those folders, or a file with more than one hard link, is refused (copy it in instead). A path may start with `~/` (the user's home folder, as the Filesystem server reads it), so `~/SquirrelUploads/report.csv` works. A folder the user sets there that other users can write (`/tmp` included) or that someone else owns is refused, with a message saying how to fix it; suggest the full path of one only they can write, or the default. A Pro account's limit is 10 MB per file.
- **Uploading doesn't put the file in the text.** It appears in the note's file list; to show an image in the note, add `![name](attachment:<id>)` with the returned `attachmentId`, with no square brackets in the name.
- `get_attachment_download_url` returns a link that expires in 5 minutes, to the encrypted file: you can't read its contents from it.
- `delete_attachment` deletes the file for good. Only when the user asks.

## Sharing

`share_note` publishes a public, passphrase-protected link (Pro only). Do it only when the user asks.
- **It shares the text only.** Unlike the app, it doesn't include the note's attachments, so attached images and files don't appear on the share page. If the note has attachments, ask the user to share it from the app.
- **Republishing:** `republish_share` pushes the note's latest text to the link, under the passphrase you give it. If the note was shared from the app with attachments, republishing it here, or calling `share_note` on it again, leaves those attachments unreadable on the share page: ask the user to republish it in the app instead.
- **The passphrase:** use at least 8 characters, as the app requires, though the tool accepts fewer. Give the user the link and the passphrase, and tell them to send the passphrase separately from the link.
- **Sharing again:** `share_note` on a note that's already shared keeps the link but resets its settings: the expiry you pass, or `never`, and download off unless you pass `allowDownload`. To update only the text, use `republish_share`.
- **Settings:** the link expires `never` (the default), or after `7d`, `14d` or `30d`, and offers a Markdown download only with `allowDownload` (off by default). `update_share_settings` changes either without republishing.
- **Status:** `get_share_status` returns the link, view count and settings, or null when the note isn't shared. `revoke_share` turns the link off at once. Deleting the note also revokes it, and restoring the note doesn't bring the link back.

## Notes and memories together

When a note holds durable facts, such as meeting decisions or a project plan's current state:
- save each fact as a memory with `sourceNoteId` set to the note;
- leave the narrative in the note;
- don't copy whole notes into memories.
