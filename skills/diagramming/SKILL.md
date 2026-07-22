---
name: diagramming
description: Create compact monospaced Unicode diagrams. Use proactively whenever a spatial view would clarify an explanation, analysis, design, plan, comparison, or system description—especially when readers would otherwise reconstruct topology, order, branching, containment, interaction, state, data movement, dependencies, boundaries, or hierarchy from prose.
license: MIT
---

# Diagramming

Make structure visible, not decorative.

## Decide whether to draw

- Skip the diagram when it would merely restate a short list, a single linear fact, or prose that is already easier to scan.
- Use a Markdown table for attribute-by-attribute comparisons; use a diagram only when the comparison is spatial or structural.
- Treat the diagram as a complement to prose. State its key takeaway in prose so the answer remains useful to screen readers or if alignment breaks.

## Select the view

| Information to reveal | View |
| --- | --- |
| Components, calls, services, or tiers | Architecture |
| Processes, networks, trust zones, or deployment boundaries | Boundary map |
| Ordered work, decisions, parallel branches, or joins | Workflow |
| Sources, transformations, stores, sinks, or events | Data flow |
| Time-ordered messages among participants | Sequence |
| Modes and the events or conditions that change them | State machine |
| Ownership, prerequisites, imports, or parent-child structure | Hierarchy or dependency map |
| Before/after structure or competing topologies | Structural comparison |

[`references/patterns.md`](references/patterns.md) owns view-specific layout guidance; read the matching section. [`references/grammar.md`](references/grammar.md) owns drawing mechanics; read it before drawing.

## Construct the diagram

1. **Frame one message.** Write the single relationship or insight the diagram must reveal.
2. **Model before drawing.** Identify nodes, directed edges, groups, boundaries, branch conditions, and unknowns. Derive them from known facts; never invent a connection to improve symmetry.
3. **Reduce.** Keep only what serves the message. Group repetition and push monitoring, policy, security, or other cross-cutting concerns to a labeled side area.
4. **Lay out.** Adapt the selected pattern, keep one dominant direction, and group before connecting.
5. **Draw.** Apply the grammar; keep labels short and unambiguous.
6. **Validate.** Compare the finished diagram with the source facts, then complete the grammar's final inspection.

## Control scale

- Treat any of these as pressure to simplify, not a hard limit: 80–100 display columns, 12–15 meaningful nodes, or a third level of boundary nesting.
- When the view becomes dense, produce one overview and one focused detail instead of a wall-sized diagram.
- Prefer whitespace and alignment over ornamental borders, shadows, colors, or emoji.
- Mark uncertainty with `?`, `unknown`, or an external note rather than implying false precision.

## Output shape

Default to a fenced `diagram` block. Keep only nodes, edge labels, and essential annotations inside it. Do not use Mermaid syntax or generate an image unless explicitly requested.

```diagram
┌────────┐  request  ┌─────────┐
│ Client │──────────▶│ Server  │
└────────┘           └────┬────┘
                          │ job
                     ┌────▼────┐
                     │ Worker  │
                     └─────────┘
```
