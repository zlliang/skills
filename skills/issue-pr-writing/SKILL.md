---
name: issue-pr-writing
description: Write, edit, review, or improve concise issue, pull request, and merge request titles, bodies, and comments for GitHub, GitLab, and similar platforms.
license: MIT
---

# Issue and PR Writing

## Purpose

Write issue, pull request, and merge request titles, bodies, and comments for [GitHub](https://github.com/), [GitLab](https://gitlab.com/), and similar platforms. Keep the wording concise, accurate, calm, skeptical, and human. Prefer concrete context over boilerplate, and use platform-neutral wording unless the user or surrounding context calls for platform-specific terms.

## Writing rules

- For English prose, follow the _Chicago Manual of Style_.
- Use consistent formatting within the same response.
- Insert spaces between English words and CJK characters.
- Always specify the language for syntax highlighting when using fenced code blocks.
- Use `- ` (hyphen plus space) for unordered list items; never use `*` or `+`.
- Use `_italics_` for italics and `**bold**` for bold.
- Never number headings (e.g., `## About me`, not `## 1. About me`).
- Never use horizontal dividers (`<hr>` or `---`) between headings.
- For list items, omit the trailing period when all items are fragments; if any item is a complete sentence, end every item with a period.

## Titles

- Issue titles use "Sentence case" (capitalize the first word only).
- Pull request and merge request titles use [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) format, following the `git-commit` skill: `<type>[optional scope]: <description>`

## Body headings

- Use headings only when they add structural clarity.
- When headings are useful, skip `#` and `##`; start visible section headings at `###` so they do not render too large.
- Do not force the text to begin with a heading. Even when headings are useful, a short opening sentence or paragraph can come first.
