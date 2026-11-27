# myBrain plugin for Claude

myBrain turns what you save (files, notes, articles, videos, interviews and personal
memories) into a knowledge graph. This plugin connects Claude to your myBrain, so Claude
can search it, cite your own sources, follow how topics connect and keep new notes in it
when you ask.

## What you need

A myBrain account. myBrain is in controlled beta: sign in at [mybrain.ai](https://mybrain.ai)
or join the waiting list there. The plugin signs you in with OAuth the first time Claude
uses it; there is no key to copy.

## What it adds

- **The myBrain connector** (`https://mcp.mybrain.ai/mcp/`): tools to search passages and
  sources, explore the knowledge graph, recall the memories you saved in myBrain, read
  your myBrain chat history, ask myBrain for a synthesized answer, and add sources. If you
  already added myBrain as a connector, the plugin uses the same server, so you see one
  set of tools.
- **The `mybrain` skill**: when to reach for your brain, which tool to use first, how to
  cite what comes back, and how to save only what you asked to keep.

## Install

- **Claude (web, desktop, mobile)**: find myBrain in the directory, under Customize,
  Plugins, and add it.
- **Claude Code**:

```bash
claude plugin marketplace add mybrain-ai/mcp
claude plugin install mybrain@mybrain
```

Then run `/mcp` and choose Authenticate for `mybrain`.

## Your data

Everything stays in your myBrain account. Claude receives only what a tool returns for
your request, and that answer then lives in your Claude conversation. New sources wait
for your approval by default; Claude adds one at once, or approves a pending one, only
when you ask it to. Memory imports and skills always wait for your approval in the
myBrain app. To disconnect, revoke the access in myBrain, under Settings, Connected Apps,
MCP Integrations.

- Privacy policy: https://mybrain.ai/privacy
- Terms of use: https://mybrain.ai/terms

## Support

Write to support@mybrain.ai, or open an issue at https://github.com/mybrain-ai/mcp/issues.
For privacy requests, write to privacy@mybrain.ai.
