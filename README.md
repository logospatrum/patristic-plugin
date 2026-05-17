# Patristica — Claude Code plugin

Russian Orthodox patristic corpus search and citation tools for any
MCP-capable agent. Backed by [logospatrum/logospatrum](https://github.com/logospatrum/logospatrum).

## Install (Claude Code)

```
/plugin marketplace add https://github.com/logospatrum/patristic-plugin
/plugin install patristic
```

This installs:
- **MCP server** `patristic` (HTTP, `https://logospatrum.com/api/mcp`) —
  six tools: `read_passage`, `lexical_search`, `semantic_search`,
  `list_authors`, `list_works`, `expand_concept`.
- **Subagent** `teo-search` — search-only, returns candidate citations
  with snippets. Cannot quote directly (no `read_passage` in its toolset).
- **Skill** `theology-router` — auto-activates on patristic/theological
  questions, routes search to `teo-search` then quotes via
  `mcp__patristic__read_passage` in the main loop.

## Install (other clients)

For Cursor, Cline, langchain-mcp-adapters, custom agents — paste into your
`mcpServers` config:

```json
{
  "patristic": {
    "type": "http",
    "url": "https://logospatrum.com/api/mcp"
  }
}
```

Or just register the MCP without subagent/skill via Claude Code CLI:

```
claude mcp add --transport http patristic https://logospatrum.com/api/mcp
```

## Tools

| Tool | What it does |
|------|--------------|
| `read_passage` | Verbatim paragraph by citation slug, with metadata (author, work, chapter, source URL). |
| `lexical_search` | Postgres tsvector + ts_rank search; best for verbatim terms. |
| `semantic_search` | bge-m3 + pgvector cosine; best for paraphrastic/conceptual queries. |
| `list_authors` | All 86 authors with slugs, name, century, section. |
| `list_works` | Works of one author by `author_slug`. |
| `expand_concept` | Resolve Church-Slavonic / archaic synonyms via glossary. |

## Why a subagent + skill, not just the MCP?

The agent contract enforced by the backend: **search returns candidates,
main reads passages, never quote without `read_passage`**. The subagent
restricts itself to search tools so it can't cheat. The skill teaches your
main loop the two-step pattern automatically.

If you don't want the subagent/skill — install just the MCP via
`claude mcp add` above.

## Self-hosting

The plugin's MCP URL points at `https://logospatrum.com`. If you fork and
run your own backend, change `mcpServers.patristic.url` in
`.claude-plugin/plugin.json`.

## License

MIT.
