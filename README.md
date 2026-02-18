# Decision Log

Agent-native decision tracking for Claude Code. Captures decisions with full context, queryable YAML frontmatter, and auto-generated indexing.

## Install

```bash
npx skills add bjg4/decision-log-skill
```

Or manually:

```bash
git clone https://github.com/bjg4/decision-log-skill.git /tmp/decision-log-skill
cp -r /tmp/decision-log-skill/.claude/skills/decision-log ~/.claude/skills/decision-log
```

## Usage

Invoke in any project:

```
/decision-log
```

The skill scans the current conversation, extracts decisions, and writes each one as a standalone markdown file in `decisions/` with YAML frontmatter:

```
decisions/
  _index.md        # Auto-generated manifest
  DEC-0001.md      # One file per decision
  DEC-0002.md
```

Each decision file captures: what was decided, options considered, rationale, expected outcome, method, and full conversation context.

### Triggers

- `/decision-log` or `/DL`
- "log this decision", "capture this decision", "record decision"
- "what decisions have we made" (reads the index)
- "tell me more about DEC-0003" (reads a specific decision)

### Querying

Decision files use YAML frontmatter, so they're greppable:

```bash
# Find all architecture decisions
grep -l "domain: Architecture" decisions/DEC-*.md

# Find superseded decisions
grep -l "status: Superseded" decisions/DEC-*.md

# Find decisions by tag
grep -l "yaml-frontmatter" decisions/DEC-*.md
```

## License

MIT — Created by [Blake Graham](https://github.com/bjg4)
