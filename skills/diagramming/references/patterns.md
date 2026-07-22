# Patterns

Use these as topological skeletons, not fixed templates. Preserve the selected view's meaning while adapting labels, orientation, and detail.

## Architecture

Use boxes for independently meaningful components. Arrows may represent calls, events, or data, so label them whenever the relationship is not obvious.

```diagram
┌────────┐  request  ┌────────┐  query  ┌──────────┐
│ Client │──────────▶│ API    │────────▶│ Database │
└────────┘           └───┬────┘         └──────────┘
                         │ job
                     ┌───▼────┐
                     │ Worker │
                     └────────┘
```

An event bus is an architecture variant, not a separate diagram type. Put the bus on the primary axis and reuse the workflow branch pattern for fan-out.

## Workflow: branch and merge

Use a question inside a normal box rather than a text-art diamond when the diamond would complicate alignment. Keep split and merge branches symmetric.

```diagram
       ┌───────────┐
       │ Cached?   │
       └─────┬─────┘
             │
      ┌──────┴──────┐
  yes │             │ no
┌─────▼─────┐ ┌─────▼─────┐
│ Read hit  │ │ Fetch     │
└─────┬─────┘ └─────┬─────┘
      │             │
      └──────┬──────┘
             │
       ┌─────▼─────┐
       │ Respond   │
       └───────────┘
```

Parallel fan-out/fan-in uses the same topology. Label lanes only when their work differs; otherwise use `Task × N` instead of drawing many identical branches.

## Data flow

Use nouns for sources, payloads, stores, and sinks; use verbs for transformations. Prefer an explicitly labeled store over a fragile imitation of a cylinder.

```diagram
┌────────┐  events  ┌───────────┐  records  ┌───────────┐
│ Source │─────────▶│ Normalize │──────────▶│ Event log │
└────────┘          └─────┬─────┘           └───────────┘
                          │ invalid
                     ┌────▼────┐
                     │ Rejects │
                     └─────────┘
```

Separate control flow from data flow when putting both in one picture would make arrow semantics ambiguous.

## Sequence

Place participants left-to-right and time top-to-bottom. Keep messages horizontal and preserve call order. Include returns only when they clarify the interaction.

```diagram
Client              API                 Database
  │                  │                      │
  │──── request ────▶│                      │
  │                  │─────── query ───────▶│
  │                  │◀────── result ───────│
  │◀─── response ────│                      │
  │                  │                      │
  ▼                  ▼                      ▼
```

For optional or repeated exchanges, add a short note beside the relevant messages rather than constructing a large frame around the sequence.

## State machine and lifecycle

Use boxes only for states. Put events, conditions, or outcomes on arrows so actions are not mistaken for states. A linear timeline is simply a state machine without back edges.

```diagram
┌──────┐  start  ┌─────────┐  success  ┌──────┐
│ Idle │────────▶│ Running │──────────▶│ Done │
└───▲──┘         └────┬────┘           └──────┘
    │                 │ failure
    │            ┌────▼────┐
    │            │ Failed  │
    │            └────┬────┘
    │      retry      │
    └─────────────────┘
```

Route a retry or feedback edge around the outside of the primary flow. Never cross it through unrelated states.

## Hierarchy and dependencies

Use a tree for true parent-child, ownership, or containment relationships.

```diagram
Application
├── Web
│   ├── Routes
│   └── Components
├── API
│   ├── Handlers
│   └── Services
└── Storage
    ├── Database
    └── Cache
```

A dependency graph with shared prerequisites is not a tree. Switch to boxes and arrows rather than threading cross-edges through a tree.

For a small relationship model, label edges directly, such as `Customer 1 ─── N Order`. Use a table or a specialized notation when many attributes or cardinalities matter.

## Boundaries, layers, and deployment

Use containing boxes for process, network, deployment, ownership, or trust boundaries. Label the boundary and place external actors outside it. A connector may cross a boundary border only when that crossing is part of the message.

```diagram
┌──────────────── Trusted network ────────────────┐
│                                                 │
│  ┌────────┐       ┌─────────┐       ┌───────┐   │
│  │ API    │──────▶│ Service │──────▶│ Store │   │
│  └───▲────┘       └─────────┘       └───────┘   │
│      │                                          │
└──────┼──────────────────────────────────────────┘
       │ authenticated request
┌──────┴─────┐
│ Public app │
└────────────┘
```

For layers, stack full-width compartments within one labeled boundary.

## Structural comparison

Align comparable nodes and keep the same reading direction on both sides.

```diagram
         Before                           After

┌────────┐  sync  ┌─────┐      ┌────────┐  async  ┌───────┐
│ Client │───────▶│ API │      │ Client │────────▶│ Queue │
└────────┘        └─────┘      └────────┘         └───┬───┘
                                                      │
                                                 ┌────▼────┐
                                                 │ Worker  │
                                                 └─────────┘
```
