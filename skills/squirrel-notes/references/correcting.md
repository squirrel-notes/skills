# Correcting memories

Agents can't delete or pin memories. Every change keeps history, and every agent change is flagged `needsReview` for the user.

## Which tool

| Situation | Tool |
|---|---|
| A memory is outdated, or wrong, and you know the correct version | `supersede_memory` with the new `text` and a `reason` |
| A decision's status has changed (proposed to decided, decided to reversed) | `supersede_memory` with the new `decisionStatus`, the text, and a `reason` |
| A memory is wrong and nothing replaces it | `invalidate_memory` with a `reason` |
| Two memories disagree and you can't tell which is right | `record_conflict` (`id`, `withId`, `reason`) |
| You now have evidence for one side of a conflict | `resolve_conflict` (`keepId`, `supersedeId`, `evidence`) |
| You need to know who changed something and when | `memory_history` |

## Pins

Any change you make unpins the memories it touches: both sides of a supersede, an invalidated memory, both memories when you record a conflict, and both when you resolve one. Pinned memories are the user's profile, so a memory you change drops out of it. If you changed a pinned memory, tell the user, so they can pin it again in the app.

## `supersede_memory`

- **The old memory** is kept as superseded and linked to the new one. Its author and creator are unchanged.
- **The reason** is stored on both records, so later sessions can see why it changed.
- **Refiling:** pass `entity`, `category`, `tags` or `keywords`.
  - An omitted one keeps the old value.
  - `tags: []` or `keywords: []` clears them.
- **`validUntil`:** omitted keeps the old date, a new date replaces it, and `null` clears it. `timezone` is refused without a new date.
- **Decisions:** to change a decision's status, supersede it with the new `decisionStatus` and a `reason`; don't save a second decision beside it. An omitted `decisionStatus`, `decisionRationale`, `decisionDate` or `decidedBy` keeps the old value, and `null` clears a detail (the status can't be cleared). `text` is always required: pass the same text if only the status changed.
- **`origin` isn't carried over.** It describes the new text, so pass it again if it still applies; omitted, the new memory has none. `sourceRef` omitted keeps the old one, and `null` clears it.
- **Unrelated text is refused.** A replacement that shares almost no words with the old memory is refused, whatever you refile. That usually means a separate fact, so use `remember_memory`. Pass `confirmUnrelated: true` only when the new text truly replaces the old one.
- **Conflicts:** a disputed memory's open conflicts carry over to the new one.

## Conflicts

- **Record, don't guess.** When two memories compete, record a conflict instead of silently picking one. Both stay in effect, and both show `disputed: true` with each other's id, so the next session sees the disagreement.
- **Pairs only.** A conflict links two memories: for three that disagree, record each pair. A memory can be in at most 10 open conflicts.
- **Resolve with evidence.** `resolve_conflict` supersedes the losing memory with the evidence as its reason, and clears the conflict from the one you keep.
- **The user is the backstop.** If you picked wrongly, they can restore the losing memory in the app.
- **Don't confuse the two "disputed"s.** A conflict is separate from a decision's own `disputed` status.

## History

`memory_history` returns each write, with before and after snapshots, the operation, the time and the actor (`app`, or an API key prefix).
- Use `since`, `id` or `entity` to narrow it.
- It returns 20 entries at a time unless you pass `limit` (at most 50); for the next page, pass its `nextCursor` back as `cursor` with the same filters. Reads are limited to about 60 a minute.
- `get_context`'s `changes` gives a short summary of the same thing.
