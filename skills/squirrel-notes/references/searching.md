# Searching memories

`search_memory` runs on the user's machine. The local MCP server decrypts the memories and ranks them.

## How a memory is found

- **By words.** A memory matches when it shares a word with your query. The matching counts its text, entity, tags and keywords.
  - Case and Latin accents are ignored: `cafe` finds *Café*.
  - Common English words such as "what", "does" and "is" are ignored, unless the query has nothing else.
  - A hyphenated word matches whole (`pihole` or `Pi-hole`), and each of its parts counts half.
  - A query word of four or more letters also half-matches a longer memory word that it starts, up to twice its length ("deploy" finds "deploys").
  - A three-letter word only half-matches its plural ("tag" finds "tags", "use" doesn't find "user").
  - The entity's words match only whole, or by a hyphenated part.
- **By meaning**, only with the optional `@squirrelnotes.app/mcp-embeddings` package.
  - A memory with no word in common is returned only if its meaning score reaches the model's floor (0.837 for the bundled gte-small).
  - Below the floor it is dropped, unless you ask for near misses.

## A reliable search routine

1. **Search with the user's wording.**
2. **Rephrase with the memory's likely wording.** Name the product, place, tool, person or abbreviation the memory would contain.
3. **Retry once with `nearMisses: true`.**
   - This adds up to 3 of the closest memories that didn't match, after the matches, with `nearMiss: true`, `score` 0 and `lexicalScore` 0.
   - `total` doesn't count them.
   - They are candidates to check against the question, not answers.
   - Without the embeddings package, none are returned.
4. **If nothing answers, say nothing is stored about it.** Don't invent, and don't save a duplicate of something you failed to find.

## Reading results

- **Order:** results come in rank order. A memory holding every query word ranks first.
- **`score`:** orders this response only. Don't compare it across searches or treat it as a threshold.
- **`lexicalScore`:** 0–1, the share of your query's words the memory contains.
- **`semanticScore`:** cosine similarity, null without embeddings. With gte-small almost everything scores 0.75–0.95, so compare hits within one response.
- **`stale` / `overdueDays`:** past its `validUntil`. A stale memory can still rank first, so check before relying on it.
- **`disputed` / `conflicts`:** the memory is in an open conflict with another. Present both views, or the resolution if there is one.
- **`decision`:** for decisions, its `status`. Searches return decided decisions only, unless you pass `decisionStatus`.
- **`needsReview`:** written or last changed by an agent and not yet confirmed by the user. It is still in effect, but say so if it matters.
- **`unverifiable`** (the count): memories that failed to decrypt or verify. If it's above zero, tell the user to check their memories in the app.

## Filters and listings

- `category`, `entity` (whole entity, ignoring case and Latin accents), `tags` (any of), `excludeTags` (wins over `tags`) and `disputed` narrow the eligible set.
- Tags are a convenience, not access control: a search without them sees everything.
- With a filter and no query, `search_memory` lists the matching memories newest first, `limit` (at most 20) at a time, with `nextCursor`.
- **References** (IDs, URLs and other lookup values) come back only with `category: "reference"`. Fetch one by key: `category: "reference"`, `entity: "<key>"`, no query.
- **Decisions:** pass `decisionStatus` (for example `["proposed", "discussed"]`) to see ones not yet decided.
- `includeSuperseded` / `includeInvalidated` bring back retired memories, for history questions.

## The profile and the briefing

- `get_memory_profile` returns the memories the user has pinned. It's the short, always-relevant set.
- `get_context` includes that profile, plus conflicts, decisions, due dates, the review count and recent changes. Use it at session start, and again only if the session is long and things changed.
