# myskills

A personal collection of agent skills (slash commands and behaviors) loaded by Claude Code.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) and tailored toward Java + Spring Boot backend work.

## Structure

Skills live under `skills/`, grouped into buckets:

- **engineering/** — daily code work
- **productivity/** — daily non-code workflow tools
- **misc/** — kept around but rarely used
- **personal/** — tied to my own setup, not promoted
- **in-progress/** — drafts not yet ready to ship
- **deprecated/** — no longer used

See [`CLAUDE.md`](./CLAUDE.md) for the rules each bucket follows.

## Reference

### Engineering

Skills used daily for code work.

- **[diagnose](./skills/engineering/diagnose/SKILL.md)** — Disciplined diagnosis loop for hard bugs and performance regressions: reproduce → minimise → hypothesise → instrument → fix → regression-test.
- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates `CONTEXT.md` and ADRs inline as decisions crystallise.
- **[tdd](./skills/engineering/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Java/Spring Boot examples grounded in DDD, OOP, and Clean Code.
- **[zoom-out](./skills/engineering/zoom-out/SKILL.md)** — Tell the agent to zoom out and give broader context or a higher-level perspective on an unfamiliar section of code.

### Productivity

General workflow tools, not code-specific.

- **[grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the decision tree is resolved.

### Misc

Tools kept around but rarely used.

- **[git-guardrails-claude-code](./skills/misc/git-guardrails-claude-code/SKILL.md)** — Set up Claude Code hooks to block dangerous git commands (push, reset --hard, clean, branch -D, etc.) before they execute.

## Attribution

Some skills are adapted from other open-source skill collections. See [`ATTRIBUTIONS.md`](./ATTRIBUTIONS.md) for sources and licenses.
