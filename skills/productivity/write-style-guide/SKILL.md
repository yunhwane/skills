---
name: write-style-guide
description: Create a STYLE.md at the repo root that codifies coding standards for the review skill's Standards axis. Detects language, framework, build tooling, and existing standards sources, then interviews the user on contested choices before writing. Use when the user wants to create or scaffold a style guide, write STYLE.md, document coding standards, or asks what the project's conventions are.
---

# Write a Style Guide

Generate a `STYLE.md` at the repo root that documents the project's **opinionated** coding standards — the kind a reviewer (human or the [`review`](../../engineering/review/SKILL.md) skill) cites when calling out a violation.

A good `STYLE.md`:

- Says **what to do and what not to do**, with reasons when non-obvious.
- Skips anything a linter, formatter, or compiler already enforces.
- Defers vocabulary to [`CONTEXT.md`](../../../CONTEXT.md) and decisions to `docs/adr/`.
- Reads like a checklist, not an essay.

## Process

### 1. Detect the stack

Read the repo to figure out what kind of project this is. Don't ask the user things the filesystem already answers.

Look at:

- **Build manifest** — `build.gradle{,.kts}`, `pom.xml`, `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc. — to identify the language, build tool, and dependency surface.
- **Language version** — Java toolchain block, `node` field, `python_requires`, `rust-toolchain`, `go.mod` directive.
- **Framework signals** — Spring Boot starters, React/Next/Vue, Django/FastAPI, Rails, Axum, etc.
- **ORM / persistence** — JPA, MyBatis, Prisma, SQLAlchemy, sqlx, etc.
- **Test stack** — JUnit/AssertJ/Mockito, Vitest/Jest, pytest, go test, etc. Note whether Testcontainers, fixtures, or property tests are present.
- **Formatter / linter config** — Spotless, Prettier, Biome, Black, gofmt, ESLint, Checkstyle, PMD. These are out of scope for `STYLE.md`; record what they cover so we don't duplicate.

### 2. Read existing standards

Don't overwrite documented rules. Read:

- `AGENTS.md`, `CLAUDE.md`
- `CONTRIBUTING.md`
- `CONTEXT.md`, `CONTEXT-MAP.md`
- `docs/adr/`
- Existing `STYLE.md` / `STANDARDS.md` / `STYLEGUIDE.md` (if any — update in place, don't replace)
- `docs/agents/*.md` if [`setup-myskills`](../../engineering/setup-myskills/SKILL.md) has already run

### 3. Survey the codebase

Spend a few reads sampling actual code to learn the project's real conventions — they often disagree with what people say they do.

- How are classes/modules organised? (layer-first vs feature-first vs hexagonal)
- How is dependency injection done? (constructor, field, manual, framework)
- How are aggregates / records / value objects shaped?
- How are tests organised? Test slices, integration containers, fixtures, tags?
- How is error handling expressed? (exceptions, `Result`, sentinel values)
- How is logging done? (which library, structured or string)
- What's already consistent? What's all over the place?

Capture inconsistencies — these are the strongest candidates for a `STYLE.md` rule.

### 4. Draft the section list

Pick the sections the project actually needs. A small project does not need a 20-section guide. Typical pool:

1. Language baseline (version, allowed features)
2. Project layout
3. Naming
4. Dependency injection / wiring
5. Domain model / data shape conventions
6. Persistence
7. Web / interface layer
8. Error handling
9. Null / option handling
10. Logging
11. Concurrency / async
12. Tests
13. Build, formatting, static analysis
14. What this guide does not cover

Drop sections that don't apply (a CLI tool has no web layer; a stateless API may not need a concurrency section).

### 5. Interview the user

Walk the section list **one section at a time**. For each section:

- Show what the codebase already does.
- Show the decisions that look contested or undocumented.
- Propose a default (cite a reasonable source — Effective Java, Google Style, framework docs).
- Ask the user to confirm, override, or skip.

Do **not** dump the whole survey at once. The user picks one section's worth of decisions, then moves on. If they're AFK, default to the inferred conventions and mark unconfirmed choices with `<!-- TODO: confirm -->`.

When the user uses a domain term that isn't in `CONTEXT.md`, mention it — that's a [`grill-with-docs`](../../engineering/grill-with-docs/SKILL.md) candidate, not a `STYLE.md` rule.

### 6. Show the draft

Render the assembled `STYLE.md`. Let the user edit before writing. Confirm the path (default: repo root).

### 7. Write

Write `STYLE.md` at the repo root. End it with the **"What this guide does not cover"** section that explicitly delegates:

- Whitespace, brace placement, import order → formatter (`<tool name>`).
- Method length, complexity → static analysis (`<tool name>`).
- Branch naming, commit format, PR templates → `CONTRIBUTING.md`.
- Architecture decisions for this project → `docs/adr/`.
- Domain vocabulary → `CONTEXT.md`.

Linking out keeps `STYLE.md` small and stops it from drifting into territory better held elsewhere.

## Rules for the output

- **Opinionated, not exhaustive.** Pick one way and call it the way. Style guides that list "or" are worthless to reviewers.
- **Cite the codebase**, not abstract principles. "All adapters live in `infrastructure/` (see `infrastructure/repository/`)" beats "follow ports and adapters."
- **Hard rules use imperative voice** (`Constructor injection only.`). Judgement calls are flagged (`Prefer X unless Y.`).
- **No machine-enforceable rules.** If a linter can catch it, don't write it down.
- **Stay under ~150 lines.** Bigger guides are unread guides. If you need more, split into `STYLE.md` + `docs/adr/`.

## Anti-patterns

- **Cargo-culted advice.** Don't paste rules from another stack ("prefer records over classes" in a Python repo). Match the language.
- **Bikeshedding.** Tabs vs spaces, semicolons, trailing commas — these go in `.editorconfig`, not `STYLE.md`.
- **Aspirational guides.** Don't write rules the codebase currently violates everywhere. Either fix the code first or capture the migration target as an ADR.
- **Comment-shaped guides.** "Write good code" is not a rule.

## When done

Mention which engineering skills consume `STYLE.md`:

- [`review`](../../engineering/review/SKILL.md) — Standards axis reads `STYLE.md` as one of its sources.
- [`improve-codebase-architecture`](../../engineering/improve-codebase-architecture/SKILL.md) — respects `STYLE.md` when proposing deepenings.

Suggest the user open a follow-up PR with the file alone so it can be reviewed and discussed before changes start landing against it.
