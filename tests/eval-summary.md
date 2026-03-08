# Eval Summary — decision-log v1.2.0

**Date:** 2026-03-07
**Method:** Manual eval via `claude -p --dangerously-skip-permissions` with and without `--disable-slash-commands`

## Results

| Test | Prompt | Result | Domain Inferred |
|------|--------|--------|-----------------|
| Explicit decision (with skill) | "log this decision: we're using PostgreSQL..." | PASS | Architecture |
| Explicit decision (baseline) | Same prompt, skills disabled | PASS* | Architecture |
| Health decision (with skill) | "log this decision: after consulting with my doctor..." | PASS | Health |
| Creative decision (with skill) | "record decision: for the album... analog tape..." | PASS | Music Production |
| Non-trigger (with skill) | "take notes from today's standup meeting..." | PASS | N/A (correctly did not activate) |

*Baseline output matched skill format exactly — `--disable-slash-commands` may not fully unload skill influence from `~/.claude/skills/`. Baseline comparison inconclusive.

## Key Findings

1. **Description fix was critical.** v1.1.0 said "project decisions" which caused Claude to gatekeep non-engineering domains (health decision was rejected as "engineering only"). Changing to "decisions" with explicit multi-domain examples fixed this.

2. **Open domain works.** Claude freely created `Music Production` as a domain without being told to. The guidance "any domain is valid" plus diverse examples is sufficient.

3. **Method inference is accurate.** Health decision correctly got `stakeholder-input` (doctor consultation). Engineering decision got `analysis`. Both inferred without explicit user input.

4. **Non-trigger boundary holds.** Standup meeting notes correctly did NOT activate the skill.

5. **Critical Behavior Rule works.** All skill-activated outputs were autonomous — no confirmation prompts, no process narration, just results.

## Changes Made During Eval

- Removed "project" from description (v1.1.0 → v1.2.0)
- Added explicit multi-domain signal: "Works for any domain — engineering, research, health, creative, financial, personal, scientific, legal, or anything else"
- Added Security section: redact credentials, omit secrets from decision files

## File Artifacts

- `eval-results/with-skill/explicit-decision.md` — PostgreSQL decision output
- `eval-results/with-skill/health-decision.md` — Health domain output
- `eval-results/with-skill/creative-decision.md` — Music production output
- `eval-results/with-skill/non-trigger.md` — Standup notes (correctly not a decision)
- `eval-results/baseline/explicit-decision.md` — Baseline comparison
