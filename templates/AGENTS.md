## Personal knowledge base (myBrain)

The user's personal knowledge base is available through the myBrain MCP server
(tools prefixed `mybrain` in your client). It contains everything they saved: files,
articles, videos, interviews, notes and personal memories, turned into a knowledge
graph scoped to this user only.

Rules of use:

1. **Search before you ask.** When the task touches the user's background, preferences,
   past work or saved material, query the brain first and silently. Start with
   `recall_memories` (facts about the person), then `search_user_knowledge` (skills,
   experiences, interests, values), then `search_passages` with three phrasings of the
   same query (entity name, a question, a declarative sentence) for literal evidence
   with the source file. Use `read_source_passages` to read more of one file,
   `explore_node` / `trace_path` for relationships, `search_sources` to find files.
2. **`ask_mybrain` is the expensive path.** It runs the full pipeline and spends the
   user's quota. Use it only when the user wants an answer synthesized from their brain
   across many sources.
3. **Write only what the user would keep.** `add_source_text`, `add_source_url` and
   `add_source_youtube` create a source that stays pending until the user approves it in
   myBrain. Write when asked or when the user shares something that matters to them; one
   consolidated note per topic, in their language, with a clear title and a short
   `context`. Check `search_sources` first to avoid duplicates. Leave `await_approval`
   at its default and do not call `approve_source` unless the user explicitly asked.
4. **Never store** scratch state, code, logs, secrets or general facts about the world.
5. **Keep the content private.** Do not forward brain content to other services or into
   shared artifacts the user did not ask for.
6. **Respect limits.** Keep `limit` small; on `429` wait `Retry-After`; on `401`/`403`
   tell the user (key invalid, or missing `mcp:read` / `mcp:write` /
   `mcp:orchestrator` scope) and stop retrying.
