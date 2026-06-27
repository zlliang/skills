---
name: git-commit
description: Create Git commits with Conventional Commits analysis, safe staging, and concise message generation. Use when the user asks to commit changes or create a git commit.
license: MIT
---

# Git Commit with Conventional Commits

Create standardized, semantic git commits using the [Conventional Commits](https://www.conventionalcommits.org/) specification. Analyze the actual diff to determine appropriate type, scope, and message.

## Commit format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit types

| Type       | Purpose                        |
| ---------- | ------------------------------ |
| `feat`     | New feature                    |
| `fix`      | Bug fix                        |
| `docs`     | Documentation only             |
| `style`    | Formatting/style (no logic)    |
| `refactor` | Code refactor (no feature/fix) |
| `perf`     | Performance improvement        |
| `test`     | Add/update tests               |
| `build`    | Build system/dependencies      |
| `ci`       | CI/config changes              |
| `chore`    | Maintenance/misc               |
| `revert`   | Revert commit                  |

## Breaking changes

Exclamation mark after type/scope:

```
feat!: remove deprecated endpoint
```

And/or: `BREAKING CHANGE` footer

```
feat: allow config to extend other configs

BREAKING CHANGE: `extends` key behavior changed
```

## Workflow

### 1. Inspect status

```bash
git status --porcelain
```

Default behavior:

- Prefer already staged changes when present.
- If nothing is staged, stage only the coherent logical change.
- Leave unrelated worktree changes untouched.

### 2. Analyze diff

```bash
# If files are staged, use staged diff.
git diff --staged

# If nothing is staged, use working tree diff.
git diff
```

When staged and unstaged changes coexist, commit only the staged changes by default. If the staged changes appear incomplete, inspect the unstaged diff and decide whether to stage closely related files or ask the user.

### 3. Stage files (if needed)

If nothing is staged or you want to group changes differently:

```bash
# Stage specific files
git add path/to/file1 path/to/file2

# Stage by pattern
git add *.test.*
git add src/components/*
```

**Never commit secrets** (.env, credentials.json, private keys).

### 4. Generate commit message

Analyze the diff to determine:

- **Type**: What kind of change is this?
- **Scope**: What area/module is affected?
- **Description**: One-line summary of what changed. (present tense, imperative mood, <72 chars)

Use the smallest accurate type and scope. Prefer short, single-line commits with no body. Add a body only when the commit is large or nuanced enough that it cannot be summarized clearly in one short sentence. Do not add an explanatory body just because extra context is available.

Use breaking-change syntax only when the diff truly changes a public contract.

### 5. Execute commit

```bash
# Single line
git commit -m "<type>[scope]: <description>"

# Multi-line with body/footer
git commit -m "$(cat <<'EOF'
<type>[scope]: <description>

<optional body>

<optional footer>
EOF
)"
```

## Best practices

- One logical change per commit.
- Present tense: "add" not "added".
- Imperative mood: "fix bug" not "fixes bug".
- Keep description under 72 characters.
- Reference issues with trailers such as `Closes #123` or `Refs #456` only when the diff or user request provides that context. If there is no body, append after the description instead: `feat: add login, closes #123`.

## Git safety protocol

- Never update git config.
- Never run destructive commands (--force, hard reset) without explicit request.
- Never skip hooks (--no-verify) unless user asks.
- Never force push to main/master.
- If hooks fail before the commit is created, fix the issue and retry
- If a commit was created and later validation fails, do not amend unless requested

## Final verification

After committing:

```bash
git status --short
git log -1 --oneline
```

Report the new commit hash and subject. Mention any remaining unstaged or untracked changes.

## References

- [Conventional Commits 1.0.0](references/conventional-commits.md)
- Forked from [github/awesome-copilot/git-commit](https://github.com/github/awesome-copilot/blob/main/skills/git-commit/SKILL.md)
