# Correcting memories

Agents can't delete or pin memories. Every change keeps history, and every agent change is flagged `needsReview` for the user.

## Which tool

| Situation | Tool |
|---|---|
| A memory is outdated, or wrong, and you know the correct version | `supersede_memory` with the new `text` and a `reason` |
| A memory is wrong and nothing replaces it | `invalidate_memory` with a `reason` |
| Two memories disagree and you can't tell which is right | `record_conflict` (`id`, `withId`, `reason`) |
| You now have evidence for one side of a conflict | `resolve_conflict` (`keepId`, `supersedeId`, `evidence`) |
| You need to know who changed something and when | `memory_history` |

## `supersede_memory`

- **The old memory** is kept as superseded and linked to the new one. Its author and creator are unchanged.
- **The reason** is stored on both records, so later sessions can see why it changed.
- **Refiling:** pass `entity`, `category`, `tags` or `keywords`.
  - An omitted field keeps the old value.
  - `tags: []` or `keywords: []` clears them.
- **`validUntil`:** omitted keeps the old date, a new date replaces it, and `null` clears it.
- **Unrelated text is refused.** A replacement that shares almost no words with the old memory is refused, whatever you refile. That usually means a separate fact, so use `remember_memory`. Pass `confirmUnrelated: true` only when the new text truly replaces the old one.
- **Pins and conflicts:** neither record stays pinned. A disputed memory's open conflicts carry over to the new one.

## Conflicts

- **Record, don't guess.** When two memories compete, record a conflict instead of silently picking one. Both stay in effect, and both show `disputed: true` with each other's id, so the next session sees the disagreement.
- **Resolve with evidence.** `resolve_conflict` supersedes the losing memory with the evidence as its reason, and clears the conflict from the one you keep.
- **The user is the backstop.** If you picked wrongly, they can restore the losing memory in the app.
- **Don't confuse the two "disputed"s.** A conflict is separate from a decision's own `disputed` status.

## History

`memory_history` returns each write, with before and after snapshots, the operation, the time and the actor (`app`, or an API key prefix).
- Use `since`, `id` or `entity` to narrow it.
- `get_context`'s `changes` gives a short summary of the same thing.
