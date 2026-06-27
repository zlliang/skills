---
name: building-skills
description: Create, edit, or improve agent skills (SKILL.md). Use whenever the task is to author a new skill, refactor or shorten an existing one, fix its frontmatter or triggering description, or split it for progressive disclosure. Load this before researching other skills or writing any SKILL.md.
license: MIT
---

# Building Skills

An [agent skill](https://agentskills.io/) is a folder with a `SKILL.md` file that teaches an agent a task on demand. The format is an open standard ([spec](https://agentskills.io/specification)) supported across many agents, so write skills that are portable, not tied to one product.

The goal every time: make the skill **smaller, clearer, and more useful** than before. A skill is read thousands of times but written once — every token must earn its place.

## How skills load (progressive disclosure)

```diagram
╭──────────────╮ always loaded  ╭───────────────────────────────────────╮
│ 1. metadata  │ ─────────────▶ │ name + description (~100 tokens)      │
╰──────────────╯                │ the only thing that decides triggering│
╭──────────────╮ on trigger     ╰───────────────────────────────────────╯
│ 2. SKILL.md  │ ─────────────▶ body instructions (aim < 500 lines)
╰──────────────╯
╭──────────────╮ as needed      ╭───────────────────────────────────────╮
│ 3. resources │ ─────────────▶ │ scripts/, references/, assets/        │
╰──────────────╯                ╰───────────────────────────────────────╯
```

Put each thing at the cheapest level that works: triggering cues in the description, the core workflow in the body, and bulky detail in resources the agent reads only when needed.

## Frontmatter

YAML frontmatter, then Markdown. Only `name` and `description` are required.

| Field | Required | Constraints |
| --- | --- | --- |
| `name` | yes | ≤ 64 chars; lowercase `a-z`, `0-9`, hyphens; no leading/trailing or consecutive hyphens; must equal the folder name |
| `description` | yes | ≤ 1024 chars (keep it far shorter); says what it does **and** when to use it |
| `license` | no | License name or file reference (this repo uses `MIT`) |
| `compatibility` | no | ≤ 500 chars; only if the skill needs specific tools/runtime/network |
| `metadata` | no | arbitrary string key-value map |
| `allowed-tools` | no | space-separated pre-approved tools (experimental) |

Most skills need nothing beyond `name`, `description`, and `license`. Don't add fields speculatively.

### Name

Pick the form from what the skill is about:

- **A specific tool** → name it after the tool: `agent-browser`, `gh-cli`, `mcporter`. Namespace by tool when it sharpens triggering (`gh-address-comments`).
- **A repeatable task, workflow, or broad action** → use the gerund form (verb + -ing): `building-skills`, `writing-issues-and-prs`, `managing-deployments`.

Either way keep it short and hyphenated, and avoid vague names like `helper`, `utils`, `tools`.

### Description — the trigger

This is the only text the agent sees when deciding to use the skill, so it carries all the "when to use" information; never put that in the body. Write it in third person, state **what** the skill does and the concrete **contexts/phrases** that should invoke it, and use specific keywords for discovery. Quote the value if it contains colons or other YAML-special characters.

- Good: `Search, lint, and rewrite code structurally with ast-grep (sg). Use for any syntax-aware task: outlining a file, finding code by AST structure, enforcing constraints, or running codemods.`
- Poor: `Helps with code.` (vague, no trigger)

If a skill under-triggers, make the description a bit more assertive about its triggering contexts rather than adding rules to the body.

## Core principles

- **Concise is key.** Assume the agent is already smart. Add only what it doesn't already know — non-obvious procedure, domain facts, project conventions. Challenge every line: does it earn its tokens? Cut introductions, summaries, and explanations of common knowledge.
- **Explain the why, not just the what.** A short rationale lets a capable agent generalize to cases you didn't foresee. Reframe heavy-handed `ALWAYS`/`NEVER` rules as the reasoning behind them; reserve hard constraints for genuinely fragile, error-prone steps.
- **Match freedom to fragility.** Open-ended task with many valid paths → high-level prose. Preferred pattern → pseudocode or a parameterized example. Fragile sequence where mistakes are costly → a precise script with few choices.
- **Generalize, don't overfit.** The skill must work across many future prompts, not just the example in front of you. Prefer durable patterns over special-casing.
- **Write imperatively.** Direct instructions to the agent ("Run X", "Read Y first"), not narration.

## Anatomy

```
skill-name/
├── SKILL.md      # Required: frontmatter + instructions
├── scripts/      # Executable code for deterministic or repeated work
├── references/   # Docs the agent reads on demand (schemas, APIs, deep guides)
└── assets/       # Files used in output (templates, icons, fonts)
```

- **scripts/** — when the agent would otherwise rewrite the same code each run, or when a step needs deterministic reliability. Reference with intent: "Run `scripts/validate.py` to check the frontmatter." Test scripts by actually running them.
- **references/** — move detailed material here to keep SKILL.md lean. Reference each file from SKILL.md and say when to read it. Keep references one level deep; add a table of contents to files over ~100 lines.
- **assets/** — resources copied or used in the output, not loaded into context.

Do **not** add `README.md`, `CHANGELOG.md`, install guides, or notes about how the skill was made. A skill contains only what the agent needs to do the job.

## Workflow

1. **Understand the task.** Get concrete examples: what should it enable, what phrases should trigger it, what's the expected output? When turning a finished conversation into a skill, mine the steps, tools, and corrections that already happened.
2. **Plan resources.** For each example, ask what script, reference, or asset would help an agent do it repeatedly. List them before writing.
3. **Draft SKILL.md.** Write the frontmatter (nail the description), then the body: a one-line summary, the workflow, and concrete examples. Build resources first when they exist.
4. **Split for disclosure.** If the body grows past ~500 lines or covers multiple variants/domains, keep the core workflow and selection logic in SKILL.md and move the rest into `references/` (one file per variant).
5. **Validate.** Check the frontmatter and naming rules. Then re-read the whole skill for consistency: uniform terminology, heading style, and formatting; and no content repeated across sections or between SKILL.md and references — each fact should live in exactly one place.
6. **Iterate on real usage.** Use the skill, watch where the agent struggles or wastes steps, and tighten. If every run reinvents the same helper, bundle it as a script. Re-read the draft with fresh eyes and cut.

## When editing an existing skill

Read it fully first and preserve its `name` and folder. Make it shorter and clearer: remove dead weight, replace rigid rules with reasoning, lift the description's triggering, and push detail into references. Match the surrounding house style. Prefer the smallest change that improves it — leave the rest alone.
