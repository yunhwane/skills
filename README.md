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
- **[improve-codebase-architecture](./skills/engineering/improve-codebase-architecture/SKILL.md)** — Find deepening opportunities in a codebase, informed by `CONTEXT.md` and `docs/adr/`. Renders the review as a self-contained HTML report with before/after diagrams.
- **[prototype](./skills/engineering/prototype/SKILL.md)** — Build a throwaway prototype to flesh out a design — either a runnable terminal app for state/business-logic questions, or several radically different UI variations toggleable from one route.
- **[setup-myskills](./skills/engineering/setup-myskills/SKILL.md)** — Scaffold the per-repo config (issue tracker, triage label vocabulary, domain doc layout) that the other engineering skills consume. Run once per repo before using `to-issues`, `to-prd`, `triage`, `diagnose`, `tdd`, `improve-codebase-architecture`, or `zoom-out`.
- **[tdd](./skills/engineering/tdd/SKILL.md)** — Test-driven development with a red-green-refactor loop. Java/Spring Boot examples grounded in DDD, OOP, and Clean Code.
- **[to-issues](./skills/engineering/to-issues/SKILL.md)** — Break any plan, spec, or PRD into independently-grabbable issues on the project issue tracker using tracer-bullet vertical slices.
- **[to-prd](./skills/engineering/to-prd/SKILL.md)** — Turn the current conversation context into a PRD and publish it to the project issue tracker. No interview — synthesizes what was already discussed.
- **[triage](./skills/engineering/triage/SKILL.md)** — Triage issues through a state machine of triage roles (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`).
- **[zoom-out](./skills/engineering/zoom-out/SKILL.md)** — Tell the agent to zoom out and give broader context or a higher-level perspective on an unfamiliar section of code.

### Productivity

General workflow tools, not code-specific.

- **[caveman](./skills/productivity/caveman/SKILL.md)** — Ultra-compressed communication mode. Cuts token usage ~75% by dropping filler while keeping full technical accuracy.
- **[grill-me](./skills/productivity/grill-me/SKILL.md)** — Get relentlessly interviewed about a plan or design until every branch of the decision tree is resolved.
- **[handoff](./skills/productivity/handoff/SKILL.md)** — Compact the current conversation into a handoff document so another agent can continue the work.
- **[write-a-skill](./skills/productivity/write-a-skill/SKILL.md)** — Create new skills with proper structure, progressive disclosure, and bundled resources.

### Misc

Tools kept around but rarely used.

- **[git-guardrails-claude-code](./skills/misc/git-guardrails-claude-code/SKILL.md)** — Set up Claude Code hooks to block dangerous git commands (push, reset --hard, clean, branch -D, etc.) before they execute.

## Attribution

Some skills are adapted from other open-source skill collections. See [`ATTRIBUTIONS.md`](./ATTRIBUTIONS.md) for sources and licenses.
