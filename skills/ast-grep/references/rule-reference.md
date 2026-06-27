# ast-grep Rule Reference

Authoritative-but-condensed reference for the ast-grep rule object, lint fields, transforms, rewriters, and ESQuery selectors. For the complete docs in one file, fetch <https://ast-grep.github.io/llms-full.txt>; the web [playground](https://ast-grep.github.io/playground.html) visualizes the AST.

A node matches a rule object when it satisfies **every** field present (an implicit, *unordered* `all`). Use an explicit `all:` array when later fields depend on meta variables captured earlier.

## Rule object

```yaml
rule:
  # Atomic
  pattern: 'console.log($ARG)'                                       # String, or object { context, selector, strictness }
  kind: 'call_expression'                                            # Tree-sitter node kind; also accepts ESQuery selectors
  regex: '^foo'                                                      # Rust regex over node text; combine with kind/pattern
  nthChild: 2                                                        # 1-based index among named siblings; or An+B / object
  range: { start: {line: 0, column: 0}, end: {line: 0, column: 5} }  # 0-based
  # Relational (each takes a sub-rule + optional stopBy; field on inside/has only)
  inside:   { kind: function_declaration, stopBy: end }
  has:      { kind: return_statement, field: body, stopBy: end }
  follows:  { pattern: 'let x = 1' }
  precedes: { pattern: 'return $V' }
  # Composite
  all: [ {kind: number}, {pattern: $A} ]
  any: [ {pattern: 'let $X = $Y'}, {pattern: 'const $X = $Y'} ]
  not: { pattern: 'console.log($_)' }
  matches: 'is-literal'                                              # Reference a utils/ rule by id
```

### Atomic rules

- **pattern** — code with meta variables. Use the object form to disambiguate:
  ```yaml
  pattern:
    context: 'class A { $FIELD = $INIT }'   # Full parseable snippet
    selector: field_definition              # The sub-node kind to actually match
    strictness: smart                       # Strictness: cst | smart(default) | ast | relaxed | signature
  ```
  `strictness` controls how unnamed/trivia nodes are compared (`cst` strictest → `signature` loosest). Note `kind` + `pattern` as sibling fields do **not** reparse the pattern; only `context`/`selector` does.
- **kind** — the tree-sitter node name (find it via `sg run -p '<code>' -l <lang> --debug-query=ast`). Use when a pattern is ambiguous (`{}`), too verbose to enumerate (class declarations), or only valid in context.
- **regex** — Rust regex ([no look-around/backrefs](https://docs.rs/regex/latest/regex/)); inline flags like `(?i)`. Always pair with `kind`/`pattern`; it can't be AST-optimized alone.
- **nthChild** — `3`, `2n+1`, or `{ position, reverse, ofRule }`. 1-based, named siblings only.
- **range** — for integrating external tools (compilers, type checkers) that supply positions. 0-based, start inclusive, end exclusive.

### Relational rules

Read as *target* `relation` *surrounding*: the **other** rule (e.g. `pattern`) matches the target node; the relational sub-rule matches the context.

- `inside` — target is inside an ancestor matching the sub-rule.
- `has` — target has a descendant matching the sub-rule.
- `follows` — target appears after the sub-rule node.
- `precedes` — target appears before the sub-rule node.

Tuning:

- **stopBy** — `neighbor` (default, one level), `end` (traverse to root/leaf/first-last sibling), or a rule object (stop, inclusively, when it matches). Default to `stopBy: end` unless you specifically want immediate neighbors.
- **field** (`inside`/`has` only) — match by semantic role, e.g. a `pair` whose `key` field is `prototype`:
  ```yaml
  kind: pair
  has: { field: key, regex: 'prototype' }
  ```

### Composite rules

- `all` (AND, ordered), `any` (OR), `not` (single sub-rule), `matches` (utility rule by id).
- `all`/`any` test **one** node against the list of rules — they do not range over multiple nodes. `all: [{kind: number}, {kind: string}]` matches nothing. To require both a number child and a string child: `all: [{has: {kind: number}}, {has: {kind: string}}]`.

## Meta variables

- `$VAR` — one named node; same name back-references (`$A == $A`).
- `$$$ARGS` — zero or more nodes (args, params, statements).
- `$_` / `$_FOO` — non-capturing (faster; each occurrence independent).
- `$$VAR` — capture an unnamed node (operator, punctuation).
- Valid: `$META`, `$META_VAR1`, `$_`. Invalid: `$lower`, `$1`, `$KEBAB-CASE`. A meta variable must be the whole node's text — `"hi $X"`, `on$EVENT`, `a $OP b` do not parse as meta variables.

## Lint & config fields

Beyond `id`, `language`, `rule`:

```yaml
constraints: { ARG: { kind: string } }               # Filter single meta vars; runs after rule; not inside `not`; ignores $$$ vars
transform:                                           # Derive new vars for fix (see below)
  NEW: replace($ARG, replace='x', by='y')
fix: 'logger.log($ARG)'                              # Textual, indentation-sensitive; or FixConfig object
rewriters: [ ... ]                                   # Sub-rules applied to children of a capture
message: 'Avoid $ARG.'                               # One-line diagnostic; rule meta vars usable
note: 'Use the logger instead.'                      # Detailed, Markdown
severity: warning                                    # Levels: error | warning | info | hint | off
labels: { ARG: { style: primary, message: '...' } }  # Custom highlight spans (vars from rule/constraints only)
files:   ['src/**/*.js']                             # Globs, relative to project root, no leading ./
ignores: ['test/**']                                 # Applied before files
url: 'https://...'
metadata: { author: '...' }
```

`severity: error` makes `sg scan` exit non-zero (CI gate). Multiple rules per file are separated by `---`.

### Utility (reusable) rules

```yaml
# Local: in the same file, referenced via matches
utils:
  is-literal:
    any: [ {kind: 'true'}, {kind: 'false'}, {kind: number}, {kind: string}, {kind: 'null'} ]
rule:
  any:
    - matches: is-literal
    - { kind: array, has: { matches: is-literal } }
```

Global utility rules live in files under `utilDirs` (declared in `sgconfig.yml`) and need their own `id` + `language`. Utilities may recurse through relational rules (`has`/`inside`); a direct `matches` dependency cycle is forbidden.

```yaml
# Recursive: match number even when wrapped in parens, e.g. (((123)))
utils:
  is-number:
    any:
      - kind: number
      - { kind: parenthesized_expression, has: { matches: is-number } }
rule: { matches: is-number }
```

## Transformations

`transform` maps a **new variable name** (no `$`) to an operation on an existing meta variable. Later transforms can consume earlier ones. Object form and string form (sg 0.38.3+) are equivalent:

```yaml
transform:
  LIST:  substring($GEN, startChar=1, endChar=-1)                # Cut leading/trailing chars
  NAME:  replace($OLD, replace='debug(?<R>.*)', by='release$R')  # Rust regex capture in `by`
  SNAKE: convert($VAR, toCase=snakeCase)                         # Cases: camelCase/snakeCase/kebabCase/pascalCase/...
  NEW:   rewrite($$$ARGS, rewriters=[my-rewriter])               # Apply rewriters to a sub-AST
```

- `replace` is the only place regex capture groups work, and only the same op's `by` can reference them.
- Conditional text (DasSurma trick): `MAYBE_COMMA: replace($$$ARGS, replace='^.+', by=', ')` yields `, ` only when `$$$ARGS` matched something.
- You cannot concatenate `$VAR` + a capitalized literal (`$VARName` parses as one var); use `replace`/`convert`.

## Rewriters

Apply different fixes to each child node of a `$$$` capture, then join the results.

```yaml
rewriters:
  - id: kw-to-pair  # Id + rule + fix required; no severity/message
    rule:
      kind: keyword_argument
      all: [ {has: {field: name, pattern: $K}}, {has: {field: value, pattern: $V}} ]
    fix: "'$K': $V"
rule: { pattern: 'dict($$$ARGS)' }
transform:
  BODY: { rewrite: { rewriters: [kw-to-pair], source: $$$ARGS, joinBy: ', ' } }
fix: '{ $BODY }'    # Result: dict(a=1, b=2) -> { 'a': 1, 'b': 2 }
```

If multiple rewriters match one node, the first listed wins.

## FixConfig (expand the replaced range)

`fix` replaces exactly the matched node. Expand the edit to absorb surrounding trivia:

```yaml
fix:
  template: ''
  expandEnd: { regex: ',' }   # Also delete the trailing comma (expandStart for the other side)
```

## ESQuery-style `kind`

`kind` accepts a subset of [ESQuery](https://ast-grep.github.io/reference/rule/esquery.html) selectors, compiled to rule objects. Also usable with `sg run -k/--kind`.

| Selector | Equivalent |
| --- | --- |
| `a > b` | `b` `inside: {kind: a}` (direct child) |
| `a b` | `b` `inside: {kind: a, stopBy: end}` (descendant) |
| `a + b` | `b` `follows: {kind: a}` (next sibling) |
| `a ~ b` | `b` `follows: {kind: a, stopBy: end}` |
| `a, b` | `any: [{kind: a}, {kind: b}]` |
| `a:has(b)` | `a` `has: {kind: b, stopBy: end}`; `:has(> b)` for direct child |
| `a:not(b)` | `a` `not: {kind: b}` |
| `:is(a, b)` | `any: [...]` (the only pseudo-class taking a comma list) |
| `a:nth-child(2n+1)` | `a` + `nthChild: 2n+1`; supports `1 of kind` and `:nth-last-child` |

Limitations: class selectors (`.body`) are rejected; only `has`, `not`, `is`, `nth-child`, `nth-last-child` pseudo-classes are supported; identifiers may not start with a digit.

## Workflow for a hard rule

1. Write a tiny example file (or use `--stdin`) showing exactly what should and should not match.
2. Inspect the tree with `sg run -p '<code>' -l <lang> --debug-query=ast` (or `cst`) to learn the real `kind` names.
3. Start from one atomic rule that selects the target node, then add `inside`/`has`/`not` to refine.
4. Build incrementally; if a relational rule misses, add `stopBy: end`. If a rule object behaves oddly, switch to an ordered `all:` (rule objects are unordered and matching can depend on order).
5. Iterate with `sg scan --inline-rules '...'` against the example; add `valid`/`invalid` cases and run `sg test` once it works.
6. The web [playground](https://ast-grep.github.io/playground.html) visualizes the AST and shares rules via URL.

## Suppression

Suppress in source: `// ast-grep-ignore` (all rules, next/same line) or `// ast-grep-ignore: rule-a, rule-b`. A first-line comment followed by a blank line suppresses the whole file.
