# Scenario Outlines

Starting points for common writing scenarios.

These are examples, not fixed templates: treat them as a starting point and shape the writing to the actual situation — reorder, drop, add, merge, or invent sections as the case demands. Prefer a repo's or tracker's own template whenever one exists.

- GitHub and GitLab: the repo's issue templates, pull request templates, and contributing guidelines
- Jira, Linear, and similar: the team's configured issue types, fields, and workflows

Before writing, skim the project's recent issues, PRs, and work items to learn its conventions.

## Bug report (issue)

- Title: what failed and where, specific enough to recognize months later — e.g. `Login button does nothing on Safari 17`, not `Login broken`.
- Summary: one or two sentences on the problem and its impact.
- Steps to reproduce: numbered, minimal, deterministic.
- Expected vs. actual: what should happen, then what does.
- Environment: OS, browser or runtime, version or commit, relevant config.
- Evidence: logs, screenshots, stack traces, or a minimal reproduction link.

## Feature request or enhancement (issue)

- Title: the outcome wanted, not the mechanism.
- Problem or motivation: the need or pain and who feels it. Lead with why.
- Proposed solution: what you suggest, at a high level.
- Alternatives considered: other options and why they fall short.
- Additional context: mockups, links, constraints, scope notes.

## Pull request or merge request

- Title: Conventional Commits format, `<type>[scope]: <description>`.
- What: a short overview of the change.
- Why: the context and reasoning; link the tracking issue (e.g., `Closes #123`).
- How it was tested: manual steps, new or updated tests, and their results.
- Screenshots: before and after for user-facing changes.
- Breaking changes or migration: call these out separately, with upgrade steps.
- Review guidance: where to start and what feedback you want.

## Work item or user story (Jira, Linear, and similar)

Prefer short, plain-language issues that describe a concrete task or problem with a clear outcome. Follow the team's convention.

- Title: a short, scannable statement of the task or outcome, in sentence case.
- Description: only as much as needed to do the work and give context; keep it brief and optional. When passing along a bug report or feature request, quote the original feedback directly rather than summarizing it, and link to the source.
- Notes or out of scope: constraints, dependencies, links, and what this explicitly does not cover.
- Acceptance criteria and deliverables (when useful): what marks the work done — testable conditions written as scenarios or a checklist of rules, and any concrete artifacts the work must produce.
