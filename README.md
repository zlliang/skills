# Skills

A small collection of [agent skills](https://docs.anthropic.com/en/docs/claude-code/skills).

## Available skills

| Skill | Description |
| --- | --- |
| [ast-grep](skills/ast-grep/SKILL.md) | Search, lint, and rewrite code structurally with ast-grep (sg). Use for any syntax-aware task: outlining a file or directory, finding code by AST structure, enforcing code constraints, writing lint rules, and running codemods/rewrites across a codebase. Prefer over text tools (grep/ripgrep) whenever the query depends on code structure rather than raw text. |
| [gh-cli](skills/gh-cli/SKILL.md) | Use GitHub CLI (gh) for GitHub repositories, issues, pull requests, actions, releases, gists, and API calls. Use when a task requires GitHub operations from the command line. |
| [mcporter](skills/mcporter/SKILL.md) | Inspect and call MCP server tools from the command line with MCPorter. Use when a task needs an MCP tool (search, docs, integrations) or when checking which MCP servers and tools are available. |
| [git-commit](skills/git-commit/SKILL.md) | Create Git commits with Conventional Commits analysis, safe staging, and concise message generation. Use when the user asks to commit changes or create a git commit. |
| [writing-issues-and-prs](skills/writing-issues-and-prs/SKILL.md) | Write, edit, review, or improve concise issue, pull request, and merge request titles, bodies, and comments for GitHub, GitLab, and similar platforms. |
