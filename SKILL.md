---
name: decision-log
description: "Capture already-made decisions as individual markdown files with YAML frontmatter in a decisions/ directory, indexed by an append-only decisions/_index.jsonl. Works for any domain — engineering, research, health, creative, financial, personal, scientific, legal, or anything else. Use ONLY when the user explicitly asks to log, record, or capture a decision, update the decision log, review logged decisions ('what decisions have we made'), look up a specific DEC-XXXX, or invokes /DL, /DecisionLog, or /decision-log. Handles decision extraction from conversations, options-considered tables, auto-generated index, status tracking, and supersession chains. Do NOT use for general note-taking, meeting minutes, journaling, brainstorming, or helping the user make a decision that has not been settled yet."
license: MIT
metadata:
  author: Blake Graham
  version: "1.3.0"
  argument-hint: "decision description or empty to scan"
---

# Decision Log Skill

Maintain a `decisions/` directory where each decision is a standalone markdown file with YAML frontmatter. An append-only `_index.jsonl` manifest is updated incrementally — one JSON record per decision — so logging never requires re-reading every decision file.

## Critical Behavior Rule

**This skill is fully autonomous.** When invoked:
- Do NOT ask the user which decisions to log. Log all unlogged decisions found.
- Do NOT present candidates for confirmation. Just write them.
- Do NOT narrate your phases or telegraph your process. Execute silently, then show results.
- The only output the user should see is the final result: the logged entries themselves.

## Directory Structure

```
decisions/
  _index.jsonl     # Append-only manifest, one JSON record per decision — never hand-edit
  DEC-0001.md      # One file per decision (source of truth)
  DEC-0002.md
```

The markdown `DEC-*.md` files are the source of truth. `_index.jsonl` is a fast cache derived from their frontmatter: it is updated incrementally (append on create, in-place line rewrite on status change) and can always be rebuilt from the decision files if missing or corrupt.

## Instructions

### Step 1 — Assess State

1. Check if `decisions/` directory exists. If not, create it.
2. List existing `DEC-*.md` files to determine the next ID (highest number + 1, zero-padded to 4 digits).
3. Read `_index.jsonl` if it exists to know which decisions are already logged. Do **not** read every `DEC-*.md` file — the JSONL records carry the id/status/domain needed to assess state. Only read full decision files when the user asks to look one up, or when rebuilding the index (see Step 4).
4. Scan the current conversation for any decisions not already captured.

**Index consistency check (cheap):** if `_index.jsonl` is missing, or its record count does not match the number of `DEC-*.md` files on disk, rebuild it once from the decision files (read each file's frontmatter, write one JSON line per decision). Otherwise trust the incremental index.

### Step 2 — Extract and Classify

For each unlogged decision, determine:

| Field | Values |
|-------|--------|
| domain | A short descriptive label for the area this decision belongs to. Infer from context. Examples: `Architecture`, `Product`, `Design`, `Operations`, `Strategy`, `Data Engineering`, `Security`, `Research`, `Finance`, `Health`, `Creative`, `Legal`, `Education`, `Workflow`, `Relationships`, `Career`, `Music`, `Science`, `Writing` — any domain is valid. Use whatever fits the decision. Prefer reusing domains already present in existing decisions for consistency, but create new ones freely when the context demands it. |
| status | `Active`, `Proposed`, `Superseded`, `Revisited`, `Deprecated` |
| method | `analysis`, `discussion`, `constraint`, `default`, `research`, `prototype`, `stakeholder-input` |

**Method definitions:**
- `analysis` — decided by weighing data, benchmarks, or structured comparison
- `discussion` — emerged from back-and-forth conversation
- `constraint` — forced by a technical, budget, or time limitation
- `default` — accepted a sensible default without active deliberation
- `research` — decided after investigating documentation, articles, or prior art
- `prototype` — decided after building and evaluating a proof of concept
- `stakeholder-input` — directed by a stakeholder, client, or external party

When the method is ambiguous, pick the one that best describes the *primary* driver. Default to `discussion` if genuinely unclear.

Also capture: decision statement, options considered (with pros/cons), rationale, expected outcome, related DEC-XXXX IDs, 2-5 lowercase tags, and context (summary, conversation highlights, actions performed, artifacts).

### Step 3 — Write Decision Files

Create a file per decision using the schema in [references/templates.md](references/templates.md). File naming: `decisions/DEC-XXXX.md` (zero-padded to 4 digits). Write all files in a single pass.

### Step 4 — Update Index

Update `decisions/_index.jsonl` incrementally using the JSONL record schema in [references/templates.md](references/templates.md). Do not re-read existing decision files — operate on the records you already have:

- **New decision:** append one JSON record line per newly written decision. No rescan.
- **Status change / supersession:** rewrite only the affected line(s) in `_index.jsonl` (read the small JSONL file, replace the records whose `id` changed, write it back). The other lines are untouched.
- **Rebuild (fallback only):** if the consistency check in Step 1 failed, read every `DEC-*.md` frontmatter and write a fresh `_index.jsonl`. This is the only path that reads all decision files, and it should be rare.

Records are append-ordered by id; consumers can sort if needed. Never hand-edit `_index.jsonl`.

### Step 5 — Show Results

Present a clean summary table:

**Logged 3 decisions:**

| ID | Domain | Decision | Status |
|----|--------|----------|--------|
| DEC-0001 | Architecture | Per-file decision tracking with YAML frontmatter | Active |
| DEC-0002 | Research | Use longitudinal study design over cross-sectional | Active |
| DEC-0003 | Creative | Analog synth palette for album intro tracks | Active |

Updated: `./decisions/`

## Edge Cases

**Explicit single decision** ("log this decision: X"): Extract only that decision. Do not scan for others.

**Supersession** ("we changed our mind on DEC-0003"): Create new file with `supersedes: DEC-XXXX`. Edit prior file's frontmatter to `status: Superseded`. Append the new record and rewrite the superseded record's line in `_index.jsonl`. Output:

**Logged 1 decision:**

| ID | Domain | Decision | Status |
|----|--------|----------|--------|
| DEC-0004 | Creative | Switch to FM synthesis for album intro tracks | Active |

Superseded: DEC-0003 → `Superseded`

Updated: `./decisions/`

**Review request** ("what decisions have we made"): Read `_index.jsonl`, render the records as a table in your reply (do not write a file). Use the table format shown in Step 5.

**Context lookup** ("tell me more about DEC-0003"): Read `decisions/DEC-0003.md`, present it.

**Status update** ("mark DEC-0003 as Deprecated"): Edit the decision file's frontmatter, then rewrite that record's line in `_index.jsonl`.

## Security

Decision files capture conversation context. When extracting decisions, omit credentials, API keys, tokens, passwords, and other secrets that may have appeared in the conversation. If a decision references a secret (e.g., "we chose AWS as our provider"), log the decision but redact the actual credential values. Frontmatter tags should never contain sensitive identifiers.

## Error Handling

**No decisions found in conversation:** Tell the user: "No unlogged decisions found in this conversation." Do not create empty files or fabricate decisions.

**Malformed existing DEC file:** If a `DEC-*.md` file has broken frontmatter, skip it during an index rebuild and warn: "Skipped DEC-XXXX.md — invalid frontmatter." Do not delete or overwrite it. (A malformed file is also why a consistency check may fail; skipping it keeps the count off by design — note this rather than looping on rebuilds.)

**Ambiguous decision boundary:** When it's unclear whether something is one decision or two, prefer fewer, broader decisions. One well-scoped decision is better than two overlapping ones.

**Referenced DEC file missing:** If a supersession or related reference points to a non-existent file, write the reference anyway but add an open question: "Referenced DEC-XXXX not found — may have been removed or renamed."
