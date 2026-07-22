# Grammar

Apply these mechanics to every diagram; simplify the layout rather than improvising ambiguous notation.

## Core palette

```text
Boxes       ┌ ┐ └ ┘ ─ │
Junctions   ├ ┤ ┬ ┴ ┼
Arrows      ▶ ◀ ▲ ▼
Trees       ├── └── │
Secondary   ┄ ┆
```

## Junction topology

Choose a glyph by the sides that actually connect. A junction visually asserts every limb it contains.

| Glyph | Connected sides | Typical use |
| --- | --- | --- |
| `─` | left, right | Horizontal edge |
| `│` | up, down | Vertical edge |
| `┌` | right, down | Top-left corner |
| `┐` | left, down | Top-right corner |
| `└` | right, up | Bottom-left corner |
| `┘` | left, up | Bottom-right corner |
| `├` | up, down, right | Branch to the right |
| `┤` | up, down, left | Branch to the left |
| `┬` | left, right, down | Merge from sides, then descend |
| `┴` | left, right, up | Arrive from above, then split sideways |
| `┼` | left, right, up, down | Four-way join or boundary pass-through |

Scan each junction by column and inspect its immediate neighbors. If a claimed limb meets whitespace, use a different glyph or add the missing segment.

If two flow edges would cross, reorder or reroute the nodes, or split the diagram; never represent the crossing with `┼`.

## Connection recipes

- Make arrow direction represent the actual relation, not merely the reading direction.
- Run horizontal arrows directly from the source border to the target border.
- For vertical arrows, replace one target-border segment without changing its width; align the stem, arrowhead, and target-box center in one column.
- In sequence diagrams, aligned terminal `▼` markers indicate downward time and have no target.

A split followed by a merge combines the junctions as follows:

```diagram
       │
  ┌────┴────┐
  │         │
┌─▼─┐     ┌─▼─┐
│ A │     │ B │
└─┬─┘     └─┬─┘
  │         │
  └────┬────┘
  ┌────▼────┐
  │ Next    │
  └─────────┘
```

## Box sizing

- Compute width in display cells, not bytes or code points.
- Set the interior width to the widest label plus left and right padding.
- Give every content row exactly that interior width; top and bottom borders must match it.
- Keep peer boxes equal in width when the equality communicates comparable roles; otherwise size independently.

For `Worker` with a display width of six cells and one cell of padding on each side, the content row is `│ Worker │`; both borders therefore contain eight horizontal segments.

## Label alignment

- Give box labels at least one cell of padding on each side. Left-align labels within peer boxes unless centered text carries meaning.
- Center a horizontal edge label over the full connector span between borders.
- Put a vertical edge label beside its stem without shifting the stem.
- Center feedback or retry labels between their parallel stems.
- Align branch conditions symmetrically beside the branches they govern.
- When a boundary label interrupts its border, preserve the total width and leave at least one border segment on each side.

## Line semantics

Use solid `─` and `│` for the primary relationship. If one secondary relationship matters, use `┄` and `┆` for exactly one declared meaning, such as asynchronous or optional flow:

Add a legend only when a secondary line style or unfamiliar marker needs an explicit meaning.

```diagram
┌────────┐  events  ┌────────┐
│ Source │┄┄┄┄┄┄┄┄┄▶│ Sink   │
└────────┘          └────────┘

Legend: ┄┄▶ asynchronous
```

Dashed corners and junctions do not have a consistently supported matching family, so they become solid at turns and joins. If that creates ambiguity, use a solid line with an edge label instead.

For an emphasized outer boundary, a self-contained double-line box is available:

```text
╔ ═ ╗   ║   ╚ ═ ╝
```

Do not junction a double-line boundary with light lines. Prefer a labeled light-line boundary when any connector must pass through its border.

Optional point markers include `● ○ ◆ ◇`. Use them only when their meaning is obvious or stated, and keep them away from alignment-critical rows when renderer behavior is unknown.

## Character-family discipline

- Use the core palette by default. Keep each shape to one border weight, use one arrowhead family throughout, and do not substitute rounded corners.
- Use spaces, never tabs.
- Remove common leading indentation so the diagram's leftmost content starts in the first column.
- Preserve intentional internal padding, but remove accidental trailing whitespace.

## Display width and CJK

Unicode text has no renderer-independent column width:

- CJK wide and full-width characters commonly occupy two display cells.
- Combining marks may occupy zero cells.
- Emoji sequences and fallback fonts may occupy one or more cells unpredictably.
- Box-drawing glyphs, arrows, and triangle arrowheads are East Asian Ambiguous. Most developer tools render them as one cell, but CJK-oriented configurations may render them as two.
- `▶` and `◀` may receive emoji-like treatment in some renderers. Do not add variation selectors.

Never claim universal alignment. Inspect the target terminal, editor, browser, or chat renderer when CJK or other variable-width text appears. Use short ASCII labels when strict cross-renderer alignment matters.

For difficult layouts, add a temporary ASCII ruler while drafting and remove it from the final result:

```text
         1         2         3
123456789012345678901234567890
```

Do not rely on language string length. Use a `wcwidth`/`wcswidth`-compatible display-cell calculation when available, then visually inspect the actual destination.

## Pure ASCII fallback

When the destination cannot align Unicode reliably, redraw the whole diagram with one ASCII family:

```text
Corners and joins  +
Lines              - |
Arrows             > < ^ v
```

Do not mix the ASCII fallback with Unicode borders in the same diagram. Simplify the topology if `+` makes junction semantics unclear.

## Final inspection

1. Compare every connector and junction with **Junction topology** and **Connection recipes**.
2. Compare every box and label with **Box sizing** and **Label alignment**.
3. Check every glyph against **Line semantics** and **Character-family discipline**.
4. Inspect the actual rendered width, using **Display width and CJK** or **Pure ASCII fallback** when needed.
