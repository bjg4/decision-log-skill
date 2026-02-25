# Decision Log Templates

Read this file when creating new decision files or regenerating the index.

---

## Decision File Schema

Each decision is a standalone file at `decisions/DEC-XXXX.md`:

```markdown
---
id: DEC-XXXX
title: "Short Decision Title"
date: YYYY-MM-DDTHH:MM
domain: [Product | Data Engineering | Design | Architecture | Operations | Strategy]
status: [Active | Proposed | Superseded | Revisited | Deprecated]
method: [analysis | discussion | constraint | default | research | prototype | stakeholder-input]
supersedes: null
related: []
tags: [tag1, tag2, tag3]
---

## Decision

[One to three sentences stating what was decided.]

## Options Considered

| # | Option | Pros | Cons |
|---|--------|------|------|
| 1 | [option] | [pros] | [cons] |
| 2 | [option] | [pros] | [cons] |
| 3 | [option] | [pros] | [cons] |

## Rationale

[Why this option was selected. Reference constraints, goals, trade-offs.]

## Expected Outcome

[What we expect to happen as a result of this decision.]

## Context

### Summary

[2-5 sentence narrative of the discussion that led to this decision.]

### Conversation Highlights

> [Direct quotes or paraphrased key points from the conversation.]

- [Highlight 1]
- [Highlight 2]

### What Was Performed

- [Action taken, analysis run, code written, prototype built, etc.]
- [Include file paths, command outputs, or tool invocations where relevant.]

### Artifacts

| Artifact | Type | Location |
|----------|------|----------|
| [name] | [code / doc / config / diagram / data] | [path or description] |

### Open Questions

- [Anything unresolved, risks identified, follow-up items.]
```

---

## Index File Schema

The index at `decisions/_index.md` is regenerated from disk on every skill invocation. Never hand-edit it.

```markdown
---
schema_version: 1
last_updated: YYYY-MM-DDTHH:MM
decision_count: N
domains:
  Architecture: N
  Product: N
  Design: N
  Data Engineering: N
  Operations: N
  Strategy: N
---

# Decision Index

| ID | Date | Domain | Title | Status | Tags |
|----|------|--------|-------|--------|------|
| [DEC-XXXX](./DEC-XXXX.md) | YYYY-MM-DD | [domain] | [title] | [status] | [tags] |
```

The index frontmatter provides aggregate counts so an agent can understand the decision landscape without reading any individual files.
