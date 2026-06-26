---
name: mcporter
description: Inspect and call MCP server tools from the command line with MCPorter. Use when a task needs an MCP tool (search, docs, integrations) or when checking which MCP servers and tools are available.
license: MIT
---

# MCPorter (`mcporter`)

[MCPorter](https://github.com/openclaw/mcporter) is a CLI for working with [MCP](https://modelcontextprotocol.io/) servers. Use it to discover configured servers, read their tool signatures, and call tools directly from the shell.

The two commands that matter most are `list` (discover and inspect) and `call` (invoke). Always `list` before you `call` so arguments match the real schema.

For exact flags, run `mcporter <command> --help`. MCPorter discovers servers from its config — the global config at `~/.mcporter/mcporter.json`, a project config at `./config/mcporter.json`, or an explicit `--config <path>` — plus editor imports (Cursor, Claude Code, Codex, etc.). Use `mcporter config list` or `mcporter list --verbose` to see where each server comes from.

## Workflow

```bash
# 1. See configured servers and their health
mcporter list

# 2. Read one server's tools as TypeScript-like signatures
mcporter list <server>

# 3. Call a tool using the signature from step 2
mcporter call '<server>.<tool>(arg: "value", count: 5)'
```

## `list` — discover and inspect

```bash
mcporter list                       # all servers, with live status
mcporter list <server>              # tools for one server, as signatures
mcporter list <server> --schema     # raw JSON schema for each tool
mcporter list <server> --json       # machine-readable, full tool metadata
mcporter list <server> --all-parameters   # reveal hidden optional params
```

Single-server output reads like a TypeScript header: a doc comment per tool, a `function name(...)` signature with required params first and optional ones marked `?`, plus an `Examples:` block written in the function-call syntax below. Optional params beyond the first few are hidden unless you pass `--all-parameters` or `--schema`.

```ts
function web_search_exa(query: string, numResults?: number);
```

## `call` — invoke a tool

**Recommended: function-call syntax.** It mirrors the signature from `mcporter list`, is self-documenting, handles nested objects/arrays, and avoids shell-flag ambiguity. Wrap the whole argument in single quotes.

```bash
mcporter call 'exa.web_search_exa(query: "best React state libraries", numResults: 5)'
mcporter call 'linear.create_comment(issueId: "ENG-123", body: "Looks good!")'
```

- Named arguments are preferred; unlabeled values map to schema order.
- Use `--output json` for structured results; connection/auth/offline/http failures emit a `{ server, tool, issue }` envelope for scripting.

```bash
mcporter call 'exa.web_search_exa(query: "...")' --output json
```

### Other argument styles (when the recommended one is awkward)

```bash
# Shell-friendly key/value pairs (good in scripts)
mcporter call linear.list_issues team=ENG limit:5

# Prebuilt JSON payload (or `--json -` to read from stdin)
mcporter call linear.create_issue --json '{"title":"Bug","team":"ENG"}'

# Read a long string argument from a file
mcporter call linear.create_comment issueId=LNR-123 body=@comment.md
```

## Ad-hoc servers (not in config)

Point `list`/`call` at an endpoint directly, no config edit needed:

```bash
mcporter list https://mcp.linear.app/mcp
mcporter call 'https://www.shadcn.io/api/mcp.getComponent(component: "vortex")'
mcporter list --stdio "bun run ./server.ts" --env TOKEN=xyz
```

Bare domains assume `https://`; plain `http://` needs `--allow-http`. Persist a tried endpoint with `--persist <config>` to make its short name reusable.

## Notes

- Pass secrets via environment variables, e.g. `LINEAR_API_KEY=… mcporter call 'linear.search_documentation(query: "automations")'`.
- Other commands exist (`auth`, `config`, `resource`, `daemon`, `generate-cli`, `emit-ts`); run `mcporter --help` when you need them.
