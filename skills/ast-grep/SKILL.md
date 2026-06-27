---
name: ast-grep
description: Search, lint, and rewrite code structurally with ast-grep (sg). Use for any syntax-aware task — outlining a file or directory, finding code by AST structure, enforcing code constraints, writing lint rules, and running codemods/rewrites across a codebase. Prefer over text tools (grep/ripgrep) whenever the query depends on code structure rather than raw text.
license: MIT
---

# ast-grep (`sg`)

[ast-grep](https://ast-grep.github.io/) is a fast, polyglot tool that treats code as syntax trees instead of text. Reach for it whenever a task depends on **code structure**: mapping a file, finding a call shape, banning a pattern, or rewriting code at scale. It is a hybrid of `grep`, `eslint`, and `codemod`, backed by [tree-sitter](https://tree-sitter.github.io/).

Prefer the short alias **`sg`** for every invocation, exactly as `rg` stands in for `ripgrep`; `ast-grep` is the same binary if `sg` is shadowed on a host. For exact flags, run `sg <command> --help`. When a rule is non-trivial, read [references/rule-reference.md](references/rule-reference.md) for the full rule object, transforms, and ESQuery selectors.

## The four commands

```diagram
╭─────────╮  map structure first   ╭──────────────────────────────────╮
│ outline │ ─────────────────────▶ │ functions, classes, imports,     │
╰─────────╯                        │ exports, members + source ranges │
╭─────────╮  ad-hoc search/rewrite ╰──────────────────────────────────╯
│ run     │ ─── -p '<pattern>' [--rewrite '<fix>'] ───▶ one-off codemod
╰─────────╯
╭─────────╮  rule-driven scan      ╭──────────────────────────────────╮
│ scan    │ ─────────────────────▶ │ YAML rules: lint, constrain, fix │
╰─────────╯                        ╰──────────────────────────────────╯
╭─────────╮  validate rules        ╭──────────────────────────────────╮
│ test    │ ─────────────────────▶ │ valid/invalid cases + snapshots  │
╰─────────╯                        ╰──────────────────────────────────╯
```

- `sg outline` — cheap, syntax-aware table of contents. Use **before** reading whole files.
- `sg run` — one-shot pattern search and rewrite from the CLI.
- `sg scan` — run YAML rules for linting, constraints, and complex codemods.
- `sg test` — test rules against `valid`/`invalid` cases with snapshots.

## Outline first (`outline`)

Before opening or reading a large file or directory, get its shape. `outline` parses on demand (no index), stays local, and is far cheaper in tokens than reading source.

```bash
sg outline src/parser.ts                                 # One file: local structure (digest view)
sg outline src                                           # A directory: exported surface
sg outline src/parser.ts --match Parser --view expanded  # Zoom into one symbol
sg outline src/parser.ts --items imports                 # Just dependencies
sg outline src/parser.ts --json=compact                  # Machine-readable
```

Useful flags: `--items {structure|exports|imports|all}`, `--view {names|signatures|digest|expanded}`, `--type class,enum`, `--match <regex>` (matches top-level item names/signatures only, **not** members), `--pub-members`, `-l <lang>` (required for stdin).

It is a navigation primitive, not a language server: no cross-file analysis, no type resolution. Use it to decide what to read next, then `run`/`scan` or read the precise slice.

## Search with patterns (`run`)

A pattern is real code with **meta variables**. ast-grep parses it and matches the tree, so it ignores formatting and matches nested occurrences.

```bash
sg run -p 'console.log($ARG)' -l js         # Single-arg console.log anywhere
sg -p 'await $_'              -l ts src/    # Run is the default subcommand
echo 'foo(1)' | sg run -p '$F($A)' -l js --stdin
```

Meta variable rules:

- `$VAR` matches one named node; reusing a name back-references it (`$A == $A` matches `a == a`, not `a == b`).
- `$$$ARGS` matches zero or more nodes (arguments, params, statements).
- `$_` / `$_FOO` are non-capturing (each occurrence may differ).
- `$$VAR` captures unnamed nodes (operators, punctuation).
- Names are `$` + uppercase/`_`/digits. `$abc`, `$KEBAB-CASE` are invalid. A meta variable must be the whole node text (`"hello $X"`, `on$EVENT` do not work).

Language is inferred from file extensions; pass `-l/--lang` for stdin or to force it. Match by node type with `-k/--kind`, which accepts [ESQuery-style selectors](references/rule-reference.md#esquery-style-kind) like `call_expression > identifier`.

When a pattern is ambiguous or won't parse, inspect the tree to find the right `kind`:

```bash
sg run -p 'class A { a = 1 }' -l js --debug-query=ast   # Formats: ast | cst | pattern | sexp
```

For a target that only parses in context, narrow with `--selector <kind>` on the CLI, or a `pattern: { context, selector }` object in a rule.

## Rules: when patterns aren't enough (`scan`)

Patterns match one node by shape. **Rules** are composable, CSS-selector-like filters that add context and logic. A rule object combines three categories (a node must satisfy *all* fields present):

- **Atomic** — `pattern`, `kind`, `regex`, `nthChild`, `range`.
- **Relational** — `inside`, `has`, `follows`, `precedes`. Each takes a sub-rule matching the *surrounding* node. Add `stopBy: end` to search to the end of that relation's direction (default is immediate neighbor only); `inside`/`has` also accept `field:` to match by semantic role.
- **Composite** — `all`, `any`, `not`, `matches` (reference a reusable `utils` rule). `all`/`any` apply to a single node, not multiple nodes.

Run a rule without project scaffolding via a file or inline:

```bash
sg scan -r rule.yml path/                     # Single rule file
sg scan --inline-rules 'id: x
language: ts
rule: { pattern: await $P, inside: { kind: for_in_statement, stopBy: end } }' src/
echo 'await x' | sg scan --stdin -r rule.yml  # Scan reads language from the rule
```

Minimal rule (find `await` inside a `for-in` or `while` loop):

```yaml
id: no-await-in-loop
language: TypeScript
rule:
  pattern: await $_
  inside:
    any: [{ kind: for_in_statement }, { kind: while_statement }]
    stopBy: end
```

The full rule object, relational tuning, `nthChild`, utility/recursive rules, and the ESQuery selector grammar are in [references/rule-reference.md](references/rule-reference.md).

## Constrain & lint a project

A lint rule extends the rule object with reporting and filtering fields. The workflow is **Find → Patch**: `rule` finds nodes, `constraints` filters meta variables, `transform` + `fix` rewrite.

```yaml
id: no-console-log
language: JavaScript
severity: error                           # Levels: error | warning | info | hint | off
message: Avoid console.log in production.
note: Use the structured logger instead.  # Supports Markdown
rule:
  pattern: console.log($GREET)
constraints:
  GREET: { kind: identifier }             # Only when arg is an identifier; ignored for $$$ vars
files: ['src/**/*.js']                    # Paths relative to project root, no leading ./
```

Project setup and CI:

```bash
sg new                    # Scaffold sgconfig.yml + rules/ + rule-tests/ + utils/
sg scan                   # Run every rule; an error-severity hit exits non-zero (CI gate)
sg scan --format github   # Or --format sarif for code-scanning
```

Suppress with a comment on/above the line: `// ast-grep-ignore` (all rules) or `// ast-grep-ignore: rule-id`. Override severity per run with `--error=<rule-id>` / `--warning=<rule-id>` / `--off=<rule-id>` (the `=` is required; bare `--error` etc. affect all rules). The official [GitHub Action](https://github.com/ast-grep/action) runs `sg scan` on push.

## Rewrite & codemod

Quick rewrites use `--rewrite`; meta variables from the pattern are substituted into the fix. ast-grep shows a diff and applies nothing until confirmed.

```bash
sg run -p 'var $X = $Y' -r 'let $X = $Y' -l js          # Preview diff
sg run -p 'foo' -r 'bar' -l py -i                       # Interactive edit: y/n/e/q
sg run -p 'foo' -r 'bar' -l py -U                       # Apply all without prompting
```

For real refactors use `fix` in a rule, optionally with `transform` (rename/case/regex/substring) and `rewriters` (apply sub-rules to each child of a `$$$` capture). `fix` is textual and indentation-sensitive; expand the replaced range with `FixConfig` (`expandStart`/`expandEnd`) to absorb a trailing comma, etc.

```yaml
id: debug-to-release
language: js
rule: { pattern: $FN($$$ARGS) }
constraints: { FN: { regex: '^debug' } }
transform:
  NEW: replace($FN, replace='debug(?<R>.*)', by='release$R')   # String-style, sg 0.38.3+
fix: $NEW($$$ARGS)
```

Transform operations and rewriters are detailed in [references/rule-reference.md](references/rule-reference.md#transformations).

## JSON output

`outline`, `run`, and `scan` accept `--json` for structured, scriptable output — pipe it into `jq`. Use `--json=stream` for large result sets or `--json=compact` for minimal size (the `=` is required).

```bash
sg run -p 'import $X from "$P"' -l ts --json | jq '.[].metaVariables.single.P.text'
```

## References

- **CLI usage** — run `sg <command> --help` (`outline`, `run`, `scan`, `test`, `new`) for exact flags, options, and current behavior.
- **Rules & rewriting** — read [references/rule-reference.md](references/rule-reference.md) for the full rule object, relational tuning, transforms, rewriters, ESQuery selectors, and the workflow for developing a hard rule.
- **Full docs** — the official site <https://ast-grep.github.io/>, the [playground](https://ast-grep.github.io/playground.html), and the single-file <https://ast-grep.github.io/llms-full.txt> for authoritative details.
