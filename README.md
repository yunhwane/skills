# myskills

A personal collection of agent skills (slash commands and behaviors) loaded by Claude Code.

Inspired by [mattpocock/skills](https://github.com/mattpocock/skills) and tailored toward Java + Spring Boot backend work.

## Structure

Skills live under `skills/`, grouped into buckets:

- **engineering/** — daily code work
- **productivity/** — daily non-code workflow tools
- **personal/** — tied to my own setup, not promoted
- **in-progress/** — drafts not yet ready to ship
- **deprecated/** — no longer used

See [`CLAUDE.md`](./CLAUDE.md) for the rules each bucket follows.

## Reference

### Engineering

Skills used daily for code work.

- **[grill-with-docs](./skills/engineering/grill-with-docs/SKILL.md)** — Grilling session that challenges your plan against the existing domain model, sharpens terminology, and updates `CONTEXT.md` and ADRs inline as decisions crystallise.

### Productivity

General workflow tools, not code-specific.

- **[grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the decision tree is resolved.

## Attribution

Some skills are adapted from other open-source skill collections. See [`ATTRIBUTIONS.md`](./ATTRIBUTIONS.md) for sources and licenses.
