---
name: gh-cli
description: Uses GitHub CLI (gh) for GitHub repositories, issues, pull requests, actions, releases, gists, and API calls. Use when a task requires GitHub operations from the command line.
license: MIT
---

# GitHub CLI (`gh`)

Use `gh` as the first choice for GitHub work that can be done from the command line: repository inspection, issue and pull request management, actions runs, releases, gists, and authenticated GitHub API calls.

For exact flags and current behavior, run `gh --help`, `gh <command> --help`, or read the official [GitHub CLI manual](https://cli.github.com/manual/).

## Operating principles

- Prefer `gh` over ad hoc browser steps for GitHub operations, especially when output must be copied, filtered, or scripted.
- Inspect before mutating. Run a read-only command first when context, repository, branch, or account is uncertain.
- Be explicit about repository scope with `--repo OWNER/REPO` when not operating from the target repository checkout.
- Use `--json` with `--jq` for structured output instead of parsing human-oriented tables.
- Avoid interactive prompts in automation. Use flags such as `--body-file`, `--title`, `--label`, `--assignee`, `--confirm`, or `--yes` only after checking the command help and confirming the operation is safe.
- Treat destructive operations as high risk: delete, close, merge, archive, release deletion, secret updates, workflow cancellation, and permission changes require extra verification.

## Initial checks

Before relying on `gh`, check the basics:

```bash
gh --version
gh auth status
git remote -v
gh repo view --json nameWithOwner,defaultBranchRef,url
```

Useful context commands:

```bash
gh status
gh repo view --web
gh browse --no-browser
gh config list
```

For GitHub Enterprise or non-current repositories, use the relevant hostname and repository flags documented by `gh --help`.

## Common workflows

### Repositories

```bash
gh repo view OWNER/REPO
gh repo clone OWNER/REPO
gh repo fork OWNER/REPO --clone=false
gh repo sync OWNER/REPO --branch BRANCH
gh repo list OWNER --limit 100 --json name,visibility,url
```

### Issues

```bash
gh issue list --state open --limit 50
gh issue view NUMBER --comments
gh issue create --title "Title" --body-file body.md
gh issue comment NUMBER --body-file comment.md
gh issue edit NUMBER --add-label label --add-assignee @me
```

### Pull requests

```bash
gh pr status
gh pr list --state open --limit 50
gh pr view NUMBER --comments --files
gh pr diff NUMBER
gh pr checkout NUMBER
gh pr create --fill
gh pr review NUMBER --comment --body-file review.md
gh pr checks NUMBER --watch
```

Before creating or updating a PR, inspect local git state with `git status --short`, `git branch --show-current`, and `git log --oneline --decorate -n 10`.

### Actions and workflows

```bash
gh run list --limit 20
gh run view RUN_ID --log
gh run watch RUN_ID
gh workflow list
gh workflow run WORKFLOW --ref BRANCH
```

Use workflow-dispatch commands only after confirming the target branch, workflow name, and required inputs.

### Releases

```bash
gh release list
gh release view TAG
gh release create TAG --notes-file notes.md
gh release upload TAG path/to/asset
```

Confirm tags, generated notes, and assets before publishing or editing releases.

### GitHub API

Use `gh api` when `gh` has no dedicated command or when a task needs a GraphQL or REST endpoint directly.

```bash
gh api repos/OWNER/REPO
gh api graphql -f query='query { viewer { login } }'
gh api --paginate repos/OWNER/REPO/issues
```

Prefer `--jq` for filtering and `--paginate` only when the endpoint may return multiple pages.

## Output handling

- For scripts, prefer stable machine output:

    ```bash
    gh pr view NUMBER --json title,author,state,mergeable --jq '{title, author: .author.login, state, mergeable}'
    ```

- For markdown bodies, write content to a temporary or existing file and pass `--body-file` / `--notes-file`. Avoid complex shell quoting for long text.
- If output may contain secrets, tokens, private URLs, or user data, summarize instead of pasting it verbatim.

## Authentication and permissions

Use `gh auth status` to confirm the active account and host. If a command fails with insufficient scopes, inspect the error, then use the official help for the specific scope requirement. Common remedies include `gh auth refresh` for interactive use or setting `GH_TOKEN` in automation.

Never print tokens with `gh auth token` unless the task explicitly requires it and the output will not be exposed.

## When not to use `gh`

- Use local `git` for commit history, blame, branch manipulation, staging, rebasing, and local diffs.
- Use direct file reads or repository search for local code understanding.
- Use the browser only when the task requires a UI-only feature, visual inspection, or an authenticated flow not covered by `gh`.
