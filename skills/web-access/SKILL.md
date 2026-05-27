---
name: web-access
description: Route agents to the right web access method only when built-in web access tools are unavailable or insufficient for the task. Use for public search/fetch, browser interaction, authenticated browsing, screenshots, web app testing, or Electron app control when built-in tools cannot handle the requirement.
license: MIT
---

# Web Access

## Purpose

Use this skill when an agent needs web access and built-in web access tools are unavailable or not capable of the given task. Its job is to choose the access route, not to explain each tool in detail.

After choosing a route, load the dedicated skill for that tool.

## Choose the route

| Need | Preferred approach |
| --- | --- |
| Search the public web | Exa MCP |
| Fetch, extract, or summarize a public page | Exa MCP |
| Read a known simple public URL, such as raw text, JSON, Markdown, or simple HTML | Direct shell fetch, such as `curl`; use Exa MCP if extraction is needed |
| Interact with a website: click, type, navigate, submit forms, or inspect rendered state | agent-browser |
| Use existing credentials, cookies, sessions, screenshots, visual inspection, or JavaScript-heavy pages | agent-browser |
| Test a web app, reproduce browser bugs, or control an Electron app | agent-browser |
| Access private service data | Prefer a configured service-specific MCP if available; otherwise use agent-browser |

## Minimal workflow

1. Classify the task using the table above.
2. Pick the lightest sufficient route:
   - Simple known public URL: direct shell fetch may be enough.
   - Public search or page extraction: use Exa MCP.
   - Interaction, authentication, visual/rendered state, testing, or Electron control: use agent-browser.
3. Load the dedicated skill for the selected route: `mcporter` or `agent-browser`.
4. (Optional) Verify important web-derived claims from fetched page content, not only search results.

## Source quality

- Prefer primary sources: official docs, standards, source repositories, vendor pages, changelogs, and authoritative publications.
- Use search results to discover sources; use fetched page content to support claims.
- Cite source URLs in final answers when web-derived facts materially affect the answer.
- If sources conflict, are stale, or access is incomplete, say so.

## Safety

- Do not ask users to paste passwords, one-time codes, API keys, cookies, tokens, or session data unless they explicitly choose that path and there is no safer alternative.
- Do not store secrets in files, logs, shell history, transcripts, or skill content.
- Do not bypass access controls, paywalls, or terms of service.
- Before submitting forms, sending messages, making purchases, deleting data, changing settings, or taking actions that affect other users or systems, state the intended action and get explicit approval unless the user already requested that exact action.
- If no available tool can access the resource, report the limitation and ask for the smallest user action that would unblock the task.
