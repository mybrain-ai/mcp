# myBrain MCP

[myBrain](https://mybrain.ai) is your personal knowledge base: everything you upload,
link, record or write becomes a knowledge graph you can search, explore and reason
over. This repository is the public home of the **MCP server entry** and of the files
that teach an AI agent how to use your brain well.

| Endpoint | Transport | Auth |
|----------|-----------|------|
| `https://mcp.mybrain.ai/mcp/` | Streamable HTTP | `Authorization: Bearer <MCP API key>` |

Create the key in myBrain: **Settings → Connected Apps → MCP Integrations**. Keys carry
scopes: `mcp:read` (search and read tools), `mcp:write` (add sources, approve, import
memories), `mcp:orchestrator` (`ask_mybrain`). Keep the trailing slash in the URL.

## Connect

**Claude Code**

```bash
claude mcp add mybrain --transport http --url https://mcp.mybrain.ai/mcp/ --header "Authorization: Bearer YOUR_MCP_API_KEY"
```

**Claude Desktop, Cursor, VS Code** (`mcpServers` in the client config):

```json
{
  "mcpServers": {
    "mybrain": {
      "url": "https://mcp.mybrain.ai/mcp/",
      "headers": { "Authorization": "Bearer YOUR_MCP_API_KEY" }
    }
  }
}
```

## Teach your agent to use it

Connecting the server is not enough. The agent also has to know *when* to search your
brain instead of asking you, which tool is the cheap one, and that every write becomes
a source pending your approval. Pick what fits your client:

| File | Use |
|------|-----|
| [`skill/mybrain/SKILL.md`](skill/mybrain/SKILL.md) | Claude Code skill. Copy the `skill/mybrain/` folder into `~/.claude/skills/` (personal) or `<project>/.claude/skills/` (project). |
| [`templates/CLAUDE.md`](templates/CLAUDE.md) | Block to paste into a project or global `CLAUDE.md`. |
| [`templates/AGENTS.md`](templates/AGENTS.md) | Same guidance for clients that read `AGENTS.md` (Codex, Cursor, others). |

All three assume the server is connected as `mybrain` (tool names `mcp__mybrain__*`
in Claude Code).

## Tools

Read (`mcp:read`): `recall_memories`, `list_memories`, `search_user_knowledge`,
`search_passages`, `read_source_passages`, `search_knowledge`, `search_sources`,
`list_sources`, `explore_node`, `trace_path`, `get_knowledge_overview`,
`list_knowledge_fields`, `get_knowledge_node`, `get_related_concepts`,
`list_chat_threads`, `get_chat_history`, `get_suggested_questions`,
`get_ingestion_status`, `get_usage_summary`.

Write (`mcp:write`): `add_source_text`, `add_source_url`, `add_source_youtube`,
`approve_source`, `import_memories`. Writes land as a source pending your approval in
myBrain and are processed only after you approve.

Ask (`mcp:orchestrator`): `ask_mybrain`, a full answer synthesized over your brain with
citations. It is rate limited and spends your plan quota.

## Registry

[`server.json`](server.json) is the manifest published to the
[official MCP Registry](https://registry.modelcontextprotocol.io/) as
`io.github.mybrain-ai/mcp`.

Publishing (maintainers only) needs an **Owner** of the `mybrain-ai` org. The
interactive `mcp-publisher login github` only grants the personal namespace
([registry#1468](https://github.com/modelcontextprotocol/registry/issues/1468)); log in
with a PAT instead (classic with `read:org`, or fine-grained with "Organization →
Members → Read-only"), on `mcp-publisher` 1.8.1 or newer:

```bash
./mcp-publisher login github --token "$GH_PAT"
./mcp-publisher publish
```

## License

The files in this repository are released under the [MIT License](LICENSE). See
[TRADEMARKS.md](TRADEMARKS.md) for the marks. The myBrain service and the server
code behind `mcp.mybrain.ai` are not part of this repository.
