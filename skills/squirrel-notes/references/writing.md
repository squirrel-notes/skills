# Writing memories

## When to save

Save:
- a durable preference ("prefers British spelling");
- a standing fact about the user or someone in their life;
- where a project stands;
- a decision and its status;
- a lookup value they'll want again.

Don't save:
- one-off context ("is tired today");
- anything already in a note (link to the note instead);
- secrets such as passwords, tokens or keys, unless the user explicitly asks;
- your own guesses presented as facts. If you must record an inference, pass `origin: "agent_inferred"`.

Ask yourself: *would a future session give a better answer because of this?*

## Shape

| Field | Guidance |
|---|---|
| `text` | One statement, up to 2,000 characters, usually one sentence. Write it so it reads correctly with no other context: "Sam's daughter starts school in February 2027", not "she starts in Feb". |
| `category` | `preference`, `entity_fact`, `project_state`, `mental_model`, `decision` or `reference`. |
| `entity` | Who or what it's about, e.g. `user`, a person's name or a project. Keep spellings consistent; the entity filter matches it whole. |
| `tags` | Up to 20 short labels for filtering (`work`, `personal`, `health`). Tags are for narrowing searches, not for search synonyms. |
| `keywords` | Up to 10 other words a later question will use that the text doesn't, each up to 50 characters. Examples: synonyms, abbreviations, the question's own wording. No commas. Each one makes the memory come up for that word in every search, so don't stuff them. |
| `validUntil` | `YYYY-MM-DD`, the last day the memory holds. Use it on `project_state` and anything temporary. The date is read in `timezone`, or else the user's default zone. |
| `origin` | `user_stated`, `agent_inferred`, `agent_suggested_user_accepted` or `external_source`. Omit it when unsure. |
| `sourceRef` | Where an `external_source` claim came from: a URL, ticket or channel. It is stored as text and never fetched. |
| `sourceNoteId` | The note a memory was taken from, if any. |

### Keywords: the biggest lever on being found

Most failed searches are a question worded differently from the memory. Add the words a person would ask with:

| Memory text | Good keywords |
|---|---|
| Runs Pi-hole with Unbound on a Raspberry Pi | `DNS`, `ad blocking` |
| Based in Sydney (AEST/AEDT) | `timezone`, `where he lives` |
| Signs off emails with "Cheers, G" | `email sign-off`, `end an email` |
| Personal projects are written in TypeScript | `programming language`, `side project` |

## Decisions

- Category `decision` needs `decisionStatus`: `proposed`, `discussed`, `decided`, `reversed` or `disputed`.
- Optionally add `decisionRationale`, `decisionDate` (`YYYY-MM-DD`) and `decidedBy`.
- Record a proposal as `proposed`, not `decided`. Searches return only decided decisions by default, so a proposal won't read as agreed.

## References

A lookup value, such as a cloud account ID or a URL, uses `category: "reference"`, with the key it's looked up by as `entity` and the value in `text`. References stay out of the profile and out of ordinary searches.

## After saving

`remember_memory` returns `nearDuplicates`. If one says the same thing in an older or less accurate way, supersede it instead of keeping both, and tell the user what you replaced.
