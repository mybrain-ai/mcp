---
name: mybrain
description: Use the myBrain MCP tools (mcp__mybrain__*) as the user's personal knowledge base and long-term memory. Invoke whenever the user refers to their own knowledge, notes, files, past decisions, skills or preferences, asks "what do I know about…" or "what did I save about…", wants an answer grounded in their own material, or asks you to keep something in their brain. Searches silently before asking the user about themselves; writes only what the user would keep, and only as a source pending their approval.
---

# myBrain skill

myBrain is the user's personal knowledge base: everything they upload, link, record or
write is turned into a knowledge graph (sources, passages, entities, knowledge about the
person) plus a set of personal memories. The MCP server exposes it to you as
`mcp__mybrain__*` tools. This skill tells you when to reach for them and how to use them
without pestering the user with questions they already answered somewhere in their brain.

Everything is scoped to the authenticated user. There is no shared or team scope.

## When to use it

Search **before asking the user** when:

- They refer to their own material: "my notes on X", "that article I saved", "the PDF
  about Y", "what I said in the interview".
- They ask what they know, think or have done about a topic.
- You are about to write something *for* them (a post, a proposal, an email) and their
  own voice, experience or prior work on the subject would change the result.
- They mention a person, project or tool you have no context for, and it plausibly lives
  in their brain.
- They ask "what should I read next" or "what is connected to X" about their own content.

Write when:

- The user explicitly asks you to keep something: "save this", "remember this", "add
  this article to my brain".
- The user shares a link or video and says it matters to them.
- A work session produced a result the user wants to keep (a decision, a summary they
  asked for). Store one consolidated note, not the transcript.

Do **not** use myBrain for:

- Your own scratch state, intermediate reasoning or task lists.
- Code, diffs, logs or anything that lives in a repository.
- Secrets, tokens or credentials.
- Facts about the world that are not about the user or their material (that is what the
  web is for).
- Every turn of a conversation. One good note beats ten fragments.

Never announce "I will now search your brain". Search, then use what you found.

## Preflight

If no `mcp__mybrain__*` tool is available, do not improvise. Tell the user once how to
connect (myBrain → Settings → Connected Apps → MCP Integrations, then the client setup in
the server README) and continue without it.

Tools require a key scope: `mcp:read` for every read tool, `mcp:write` for
`add_source_*`, `approve_source` and `import_memories`, `mcp:orchestrator` for
`ask_mybrain`. A `403` means the key lacks the scope; say so and stop retrying.

## Reading: which tool, in this order

Start cheap and specific. Escalate only when the previous step did not answer.

1. **`recall_memories`** — facts and preferences about the person (how they like to
   work, what they are doing, who they are). First stop for anything about the user as a
   person. `limit` 5–10.
2. **`search_user_knowledge`** — synthesized knowledge about the user by facet:
   `SKILL`, `EXPERIENCE`, `AFFINITY` (interests), `VALUES`. Use when the question is
   "what does the user know / can do / care about". Filter by `facet` when you can.
3. **`search_passages`** — literal passages cited with the source file. Pass **three
   phrasings** of the same query (bare entity name, a question ending in `?`, a
   declarative sentence); they run as three retrieval branches and are fused, so one
   phrasing degrades recall. `limit` ≤ 10. This is the tool for "what did the source
   actually say".
4. **`read_source_passages`** — go deeper into *one* file: `match_mode="expand"` with a
   passage handle from step 3 to read around it, `"search"` with a `query` to look inside
   the file, `"page_up"`/`"page_down"` to paginate. `limit` ≤ 6.
5. **`search_knowledge`** — single-query semantic search when you just need the top
   hits fast. `limit` ≤ 10.
6. **`explore_node`** / **`trace_path`** — relationships: what is connected to a concept
   or a file, and how two concepts connect. Use when the question is about links, not
   content. Keep `depth` at 1 unless the user asks for more.
7. **`search_sources`** / **`list_sources`** — find the *files* (by title, type, origin,
   date), not their content. Use before writing, to avoid duplicates, and when the user
   asks "did I save X".
8. **`ask_mybrain`** — the full RAG pipeline. It reasons over the whole brain and returns
   a written answer with citations. It is the most expensive call (rate limit 10/min, and
   it spends the user's plan quota). Use it when the user explicitly wants an answer *from
   their brain* on a question that needs synthesis across many sources, not as a generic
   search. Do not call it for things steps 1–5 answer.

Also available: `get_knowledge_overview`, `list_knowledge_fields`, `get_knowledge_node`,
`get_related_concepts` (graph views), `list_chat_threads` / `get_chat_history` (the
user's myBrain chats), `get_suggested_questions`, `get_ingestion_status`,
`get_usage_summary`.

Reading principles:

- Search in the user's language. Content is stored as written; do not translate queries
  into English if the brain is in Portuguese.
- Keep limits small and raise them only when results are thin.
- Cite the source title when you use a passage. The user wants to know *where* it came
  from.
- Treat memories as the user's current state; treat old passages as what a document said
  at the time.
- An empty result from `search_sources` / `search_user_knowledge` comes back as
  `status: "error"` with a `summary` of suggestions (widen filters, try
  `match_mode="lexical"` with part of the name, or `"metadata"` to browse). That is not
  a failure; follow one suggestion, then stop.
- If search returns nothing, say what you looked for and ask one precise question. Do
  not guess the user's background.

## Writing: every write is a source pending approval

There is no "memory store" write. Writes create a **source** that enters the user's
knowledge graph only after approval, and processing counts against their plan. Act
accordingly:

- `add_source_text(title, content, category?, context?)` for notes, decisions and
  summaries. Write a clear `title`, a `content` in the user's language, and a short
  `context` saying why it matters. One note per topic.
- `add_source_url(url, …)` for articles and pages; `add_source_youtube(youtube_url, …)`
  for videos. Pass `context` when the user said why they care.
- Leave `await_approval=true` (the default). The source lands as `approval_pending`
  and the user approves it in the myBrain UI. Only call `approve_source` yourself when
  the user explicitly told you to approve, in this conversation.
- Before `add_source_text`, run `search_sources` with the intended title. If a similar
  source exists, tell the user instead of duplicating.
- After writing, report the source id and that it is pending approval. Do not poll
  `get_ingestion_status` in a loop; check once if the user asks.
- `import_memories(provider, payload)` only when the user hands you an export from
  ChatGPT, Claude or Gemini (or pasted memories). Leave `reconcile=false` unless the
  user asks for deduplication; it is a paid LLM pass.

Ingestion is idempotent (same content hashes to the same source), so a retry after a
network error is safe.

## Limits and errors

- Rate limits are per user per tool; `429` returns `Retry-After`. Wait that long, do not
  hammer.
- `401`: the key is invalid or revoked, or points at the wrong environment. Tell the
  user; do not retry.
- `403`: missing scope (see Preflight).
- Passage handles (8 hex chars) are valid for this session only. Do not store them.
- Tool outputs are already compact. Do not re-fetch the same passage to "double check".

## Privacy

The brain is the user's private material. Do not send its content to other tools, web
searches or third-party services unless the user asks. Do not quote it back into places
the user did not ask for (commit messages, issues, shared documents).

## Examples

**User:** "Write a LinkedIn post about event-driven architecture, in my voice."
→ `recall_memories("how the user writes and what they care about")`,
`search_user_knowledge("event-driven architecture", facet=["SKILL","EXPERIENCE"])`,
`search_passages(["event-driven architecture", "what did I say about event-driven architecture?", "my experience with event-driven systems"])`. Write the post from what came back, cite nothing in the post, and tell the user which sources shaped it.

**User:** "What did that article on Kafka say about exactly-once?"
→ `search_passages(["Kafka exactly-once", "how does Kafka achieve exactly-once?", "Kafka exactly-once semantics in the saved article"], limit=5)`; if the right passage is there, `read_source_passages(target=<handle>, match_mode="expand")` and quote it with the source title.

**User:** "Save this: we decided to move the ingestion queue to Temporal because of retries."
→ `search_sources(query="ingestion queue Temporal", limit=3)`; if nothing similar,
`add_source_text(title="Decision: ingestion queue moves to Temporal", content="…", context="Decision taken on <date> in a working session; reason: retries and visibility.")`. Reply with the id and that it is pending approval.

**User:** "Based on everything in my brain, what are my strongest arguments for remote work?"
→ This needs synthesis across sources: `ask_mybrain("What are my strongest arguments for remote work, based on my sources?")`. Return the answer with its citations.

**User:** "Who do I know that worked with Rust?"
→ `search_passages(["Rust", "who worked with Rust?", "people I know with Rust experience"])` then `explore_node` on the entities that came back. Do not invent names.
