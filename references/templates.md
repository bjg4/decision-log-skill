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
domain: [descriptive label — any domain is valid, e.g. Architecture, Product, Health, Creative, Finance, Research, etc.]
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

The index at `decisions/_index.jsonl` is an append-only manifest with **one JSON object per line**, one line per decision. It is updated incrementally (append on create, in-place line rewrite on status change) and is never hand-edited. The `DEC-*.md` files are the source of truth; the JSONL is a fast cache that can be rebuilt from them.

Each line is a compact mirror of that decision's frontmatter:

```jsonl
{"id":"DEC-0001","date":"YYYY-MM-DDTHH:MM","domain":"Architecture","title":"Short Decision Title","status":"Active","method":"analysis","supersedes":null,"related":[],"tags":["tag1","tag2"]}
{"id":"DEC-0002","date":"YYYY-MM-DDTHH:MM","domain":"Health","title":"Another Decision","status":"Superseded","method":"stakeholder-input","supersedes":null,"related":["DEC-0003"],"tags":["sleep","routine"]}
```

Field rules:

- One record per `DEC-*.md` file. `id` is unique and is the key used for in-place rewrites.
- `supersedes` is a `DEC-XXXX` string or `null`. `related` and `tags` are JSON arrays (may be empty).
- Records are written in id order; consumers that need another order can sort.

Because every record carries `id`, `domain`, `status`, and `tags`, an agent can understand the whole decision landscape — counts, domains, what's superseded — by reading only this one file, without opening any individual decision. Aggregate counts (per-domain totals, decision count) are computed on demand from the lines rather than stored.

### Rendering a review table

When the user asks to review decisions, read `_index.jsonl` and render the records as a markdown table in the reply (do not persist a file):

```markdown
| ID | Date | Domain | Title | Status | Tags |
|----|------|--------|-------|--------|------|
| [DEC-XXXX](./DEC-XXXX.md) | YYYY-MM-DD | [domain] | [title] | [status] | [tags] |
```
