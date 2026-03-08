# Decision Log

Decision tracking for Claude Code. Captures decisions from any domain with full context, queryable YAML frontmatter, and auto-generated indexing.

Works for engineering, research, health, creative, financial, personal, scientific, legal, or any other context where you want to track decisions over time.

## Install

```bash
npx skills add bjg4/decision-log-skill
```

Or manually:

```bash
git clone https://github.com/bjg4/decision-log-skill.git ~/.claude/skills/decision-log
```

## Update

```bash
npx skills add bjg4/decision-log-skill
```

If you installed via git clone:

```bash
cd ~/.claude/skills/decision-log && git pull
```

## Usage

Invoke in any conversation:

```
/decision-log
```

The skill scans the current conversation, extracts decisions, and writes each one as a standalone markdown file in `decisions/` with YAML frontmatter.

```
decisions/
  _index.md        # Auto-generated manifest
  DEC-0001.md      # One file per decision
  DEC-0002.md
```

Each decision captures: what was decided, options considered, rationale, expected outcome, method, and full conversation context.

### Triggers

- `/decision-log` or `/DL`
- "log this decision", "capture this decision", "record decision"
- "what decisions have we made" (reads the index)
- "tell me more about DEC-0003" (reads a specific decision)

### Example Output

```
Logged 2 decisions:

| ID | Domain | Decision | Status |
|----|--------|----------|--------|
| DEC-0001 | Architecture | Use PostgreSQL as the primary database | Active |
| DEC-0002 | Music Production | Analog tape recording for drum tracks | Active |

Updated: ./decisions/
```

### Querying

Decision files use YAML frontmatter, so they're greppable:

```bash
# Find all architecture decisions
grep -rl "domain: Architecture" decisions/

# Find superseded decisions
grep -rl "status: Superseded" decisions/

# Find decisions by tag
grep -rl "postgresql" decisions/
```

## Skill Structure

```
SKILL.md              # Main skill definition
references/
  templates.md        # Decision file and index schemas
tests/
  prompts.json        # Trigger and non-trigger test prompts
  eval-prompts.json   # Eval pipeline prompts with assertions
  eval-summary.md     # Latest evaluation results
  eval-results/       # Raw eval outputs
```

## License

MIT — Created by [Blake Graham](https://github.com/bjg4)
