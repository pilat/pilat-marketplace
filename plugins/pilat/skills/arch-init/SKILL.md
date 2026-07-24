---
description: Use when initializing architecture documentation for a project. Triggers on "set up arch docs", "initialize ARCHITECTURE", "scaffold architecture documentation", "add architecture docs". Creates self-describing docs that contain their own maintenance rules.
---

# Arch-Init

Set up architecture documentation that knows how to maintain itself.

## Goal

Create docs (ARCHITECTURE.md, docs/coding-style.md, CLAUDE.md, docs/glossary.md, docs/adr/) that:
1. Describe the project's architecture accurately
2. Contain project-specific rules for how to keep them in sync with code
3. Work for any project structure — not just modular codebases

The docs you generate are self-describing: they tell future agents exactly how to verify and update them for THIS specific project.

## When NOT to use

- Single-file scripts, trivial utilities
- Config-only repos (dotfiles, IaC modules without app logic)
- Notebooks, ML exploration code
- Documentation-only repos
- **Monorepos with multiple services/apps.** Why this matters: arch-init produces ONE ARCHITECTURE.md, and that doc has to be accurate AND stay accurate. If a single doc tries to describe N independently-deployable services, it ends up lying about all of them — and arch-sync would hallucinate drift on every change. So when a repo is really N projects living together, we redirect instead of pretending.

  The question to ask yourself: **would someone deploy or ship JUST this sub-thing on its own?** If yes for two or more sub-directories, you're in a monorepo. Common signals that something is a multi-project workspace: `go.work`, root `package.json` with `workspaces`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`, Cargo `[workspace]`, Deno/Bun workspaces, Elixir umbrella (`apps_path`), `rebar.config` with apps. But the signal alone isn't the test — the test is whether multiple things actually ship independently.

  Internal `packages/*` that exist purely to share config/types/tooling within ONE deployable app (e.g., `packages/eslint-config`, `packages/tsconfig`, `packages/types`, an internal-only `packages/ui` consumed by the same app) do NOT make it a monorepo. One ship target = one project.

  When you do detect a real monorepo, tell the user what you found, recommend running arch-init inside each service directory, and stop. If they push back and ask for one root-level ARCHITECTURE.md anyway, hold the line — explain that the doc would be wrong on day one and arch-sync would punish them later. Don't capitulate to make the user happy; you'd be setting them up for worse pain.

If the project looks inappropriate — say so and stop. Don't scaffold just because asked.

## Two-Phase Research

### Phase 1: Structure Discovery

Spawn subagents to understand the project shape:
- What languages, frameworks, build systems?
- What's the project type — service, library, CLI, monorepo, data pipeline, SPA?
- Where are the boundaries — packages, modules, layers, features, or something else entirely?
- What's the public surface — HTTP API, library exports, CLI commands, none?
- What's generated vs authored?
- What are the entry points?

Phase 1 output: A map of "what matters" and "where to look deeper". Decisions about what structure makes sense for THIS project's docs.

### Phase 2: Deep Dive

Based on Phase 1 findings, spawn subagents to explore specific areas:
- If there's a domain layer → understand key entities and their relationships (these seed docs/glossary.md)
- If there's an API surface → document endpoints/exports and how to detect changes
- If there are architectural invariants → capture them (dependency rules, layer boundaries)
- If there's state management → understand what owns what
- If there are cross-cutting concerns → understand how they're handled

Phase 2 output: Enough understanding to write accurate docs with project-specific sync rules.

## What You Generate

File layout:
- `ARCHITECTURE.md` and `CLAUDE.md` at project root (top-level agent-facing docs; ALL_CAPS naming follows the established root-doc convention used across open-source)
- `docs/coding-style.md`, `docs/glossary.md`, and `docs/adr/` under `docs/` (lowercase-kebab; ADR folder follows the `docs/adr/` convention popularized by Martin Fowler and adr-tools)

Create the `docs/` directory if it doesn't exist.

**Never overwrite pre-existing files.** Treat anything already on disk as user content:
- `CLAUDE.md` exists → append-only, see collision rules in the CLAUDE.md section below.
- `ARCHITECTURE.md` / `docs/glossary.md` / `docs/coding-style.md` / any file under `docs/adr/` already exists → stop and ask the user before changing it. Default: leave it, propose updates as a diff.
- Other pre-existing files under `docs/` (e.g., hand-written `docs/api.md`, `docs/runbook.md`, `docs/setup.md`) → leave them untouched. You may reference them from ARCHITECTURE.md or CLAUDE.md read order, but never edit their content.

**Watch for functional duplicates at different paths.** If you spot an existing doc that serves the same purpose as one you're about to generate (e.g., a hand-written `docs/architecture.md`, `docs/design.md`, or `ARCHITECTURE_OVERVIEW.md` while you're about to create `ARCHITECTURE.md` at root; or a hand-written `docs/style.md` vs your `docs/coding-style.md`; or a root `GLOSSARY.md` / `CONTEXT.md` vs your `docs/glossary.md`), stop and ask before generating. Don't ship two competing docs in the same project — that's how teams end up with three sources of truth and zero accurate ones. Offer the user a clear choice: (a) skip generation, the existing doc is authoritative; (b) generate yours, mark the existing as legacy in the read order; (c) migrate content from the old into the new and remove the old. Their call, not yours.

### ARCHITECTURE.md

Architecture description. Structure adapts to the project. Examples:
- Go service with `internal/` → "Modules" section
- React/Vue app with `features/` → "Features" section
- Python/Ruby/PHP web app with layers → "Layers" or "Components"
- Rust workspace → "Crates" section
- Java/Kotlin Maven/Gradle → "Packages" or "Modules"
- C#/.NET solution → "Projects" or "Assemblies"
- CLI or library → "Public API" or "Commands"
- Something else → figure out what fits

Don't force structure. Describe what exists. The skill must work for any language — if your stack isn't in the list above, infer the right grouping from how the code is actually organized.

Include a section explaining how to verify this doc stays accurate for THIS project — what to check, where to look, what commands to run.

### docs/coding-style.md

Code and architecture style rules. Two parts:
- **Code-level:** formatting, naming, idioms, error handling patterns
- **Architecture-level:** allowed dependencies, layer rules, patterns in use

**Source priority — codify, don't invent:**
1. **If linter/formatter configs exist** (e.g., `.eslintrc`, `.prettier*`, `.ruff.toml`, `pyproject.toml [tool.ruff]`, `.rustfmt.toml`, `.golangci.yml`, `.editorconfig`): extract the actual rules and document them. These are the team's real conventions.
2. **If no config exists:** scan the actual code for de-facto patterns (error handling, naming, import order, file size, where business logic lives). Then look up the dominant style for the language (gofmt + golangci-lint for Go, ruff + black for Python, rustfmt + clippy for Rust, eslint + prettier for JS/TS, rubocop for Ruby, php-cs-fixer for PHP, etc.) and propose those as the baseline. Ask the user ONE question: "No style config detected. Use [language-standard tool] defaults as the baseline, or do you have other conventions in mind?" Codify their answer. Don't invent rules from scratch — pick from established practice and let them confirm.

### docs/glossary.md

Before writing it, check for an existing glossary (a root `GLOSSARY.md` or `CONTEXT.md`) — that's a functional duplicate: stop and ask, per the duplicates rule above.

Project glossary — the shared vocabulary. It pays off three ways: variables, functions, and files get named consistently; the codebase gets easier to navigate; and agents stop spending twenty words paraphrasing a concept the project names in one.

Entry format:

```md
**Order**:
A confirmed request to buy, created at checkout.
_Avoid_: purchase, transaction
```

Definitions are one tight paragraph — what the thing IS, not what it does. The `_Avoid_` line is the load-bearing part: a glossary that names the winner but not the banned synonyms is decorative. Omit `_Avoid_` when a term has no tempting synonym — a manufactured one weakens the real bans. Close the file with a `## Flagged ambiguities` section for naming conflicts, open and resolved.

Seed it from what the code already reveals — the entities and terms Phase 2 surfaced. Project-specific concepts only; general programming vocabulary (handlers, retries, timeouts) doesn't belong, however often it appears. When code and prose disagree on a name (README says cart, the type is `basket`), pick one winner, put the loser on `_Avoid_`, and log the conflict in `## Flagged ambiguities` — a silent pick reads as authoritative and buries a discussion nobody had.

Like the other docs, it carries its own maintenance rule. Write it in near the top: entries are added in the same turn an ambiguity surfaces — someone had to ask what a term means, or noticed two names for one thing, or one name for two. Never batch-extracted at session end: that's a chore that gets skipped, after the context that made the ambiguity obvious is gone. Name arch-sync as the sanctioned backstop: its diff sweep catches what live sessions missed.

### CLAUDE.md

Entry point for agents. Contains:
- Read order (README → docs/glossary.md → ARCHITECTURE.md → docs/coding-style.md → docs/adr/) — glossary first, it's the vocabulary everything else is written in
- Non-negotiables for this project
- Project-specific guidance for skills (implementor, brainstormer, etc.)

**If CLAUDE.md already exists** — its content belongs to the user, treat it as sacred. Your job is to add the architecture-docs glue (read order, `/pilat:arch-sync` pointer, "keep docs accurate" reminder) without disturbing anything that's already there.

Approach by content-type, not by heading name. If you're appending list items (file paths in a read order), find an existing list-shaped section (`## Read Order`, `## Docs`, etc.) and extend it — append-only, never reorder or remove existing entries even if they look stale (that's the user's call). If you're adding prose (sync pointer, warnings), find a prose section about docs or create a new `## Architecture Docs` section. If your payload has both shapes, split it across the matching targets — don't force everything into one place.

If nothing relevant exists, append a new `## Architecture Docs` section at the end with read order + sync pointer + reminder.

Show the user a unified diff before saving. They may accept everything, reject everything, or accept only some hunks — re-render with only accepted hunks before writing. If they reject everything, warn them that without read-order links in CLAUDE.md, future agents may miss the new docs.

### docs/adr/

Decision log directory:
- `README.md` explaining the ADR pattern (what an ADR is, when to write one, the filename convention used here)
- `TEMPLATE.md` for new decisions

**Filename convention:** `NNNN-kebab-case-title.md` with a zero-padded sequential number (e.g., `0001-postgres-over-mysql.md`, `0002-use-event-sourcing.md`). This is the standard ADR convention per Martin Fowler and adr-tools — easy to reference ("see ADR-7"), trivially sortable. Document this convention in the `README.md` you create.

**TEMPLATE.md structure:** Status (proposed / accepted / superseded), Context, Decision, Consequences, Alternatives Considered. ADRs are immutable once accepted — supersede by writing a new one that links back to the old.

## Sync Rules

Every ARCHITECTURE.md you create must have a section that explains how to keep it accurate. This section should cover:

- **What to check:** specific files, directories, patterns relevant to this project
- **How to detect drift:** commands, comparisons that make sense here
- **What counts as drift:** changes that require doc updates vs normal churn
- **How to update:** what to do when drift is found

These rules are project-specific. A Go service has different sync rules than a React app than a Python CLI. You figure out what makes sense during research.

Example for a Go service:
```markdown
## Keeping This Document Accurate

After implementation changes, verify:
- `ls internal/` matches the module list in this doc
- Each module's exported functions match its documented API
- Dependencies in go.mod align with what's documented
- New fx.Module registrations are reflected in wiring descriptions

Run /pilat:arch-sync to check automatically.
```

Example for a React app:
```markdown
## Keeping This Document Accurate

After implementation changes, verify:
- Feature directories match the Features section
- Route definitions match documented routes
- New API hooks are reflected in Data Fetching section
- Shared components haven't diverged significantly

Run /pilat:arch-sync to check automatically.
```

Example for a Python service:
```markdown
## Keeping This Document Accurate

After implementation changes, verify:
- Package directories under `src/` match the documented Modules
- Public API in `src/<pkg>/__init__.py` matches documented exports
- `pyproject.toml` deps reflect any new architectural dependency
- Pydantic/SQLAlchemy model files match the Entities table

Run /pilat:arch-sync to check automatically.
```

The skill must work for any language — if your stack isn't shown above, write sync rules that fit how the code is actually organized.

## One Focused Question

Before scaffolding, ask ONE question that unblocks the most. Match to what you detected:

| Detected | Ask |
|----------|-----|
| Backend + frontend together | "Cover both in one ARCHITECTURE.md, or split them?" |
| Existing partial docs | "Build on existing docs or start fresh?" |
| Clear single project | "Anything specific about the architecture I should know?" |

Don't bundle questions. One focused question, then proceed with sensible defaults.

The anti-pattern is interrogation, not gathering necessary info. Some narrow sub-questions are legitimate inside a single-issue scope — e.g., if no style config exists you also need a style-baseline confirmation. Batch all genuine clarifications into ONE turn; don't ping-pong.

## After Generation

1. **Re-read what you wrote.** Spot-check that the entities, modules, and dependencies in ARCHITECTURE.md actually match the code. If you find inaccuracies, fix them before reporting done.

2. **Tell the user:**
   - What was created (full paths)
   - Next step: run /pilat:arch-sync after future implementation cycles
   - Suggest a language-specific architecture testing tool if one fits the stack:
     - Go: goguard, go-cleanarch
     - Java/Kotlin: ArchUnit
     - TypeScript/JavaScript: dependency-cruiser, eslint-plugin-boundaries
     - Python: import-linter, deptry
     - Rust: cargo-deny, cargo-modules
     - C#/.NET: NetArchTest
     - Ruby: rubocop with custom architectural cops
     - PHP: deptrac
     - Any other language: arch-sync covers the gap.

## Anti-patterns

- Don't scaffold on trivial projects
- Don't force modular structure on non-modular projects
- Don't write generic rules — every sync rule should be specific to THIS project
- Don't over-document — capture what matters, skip what's obvious
- Don't invent conventions — codify what exists
- Don't ask multiple questions — one focused question, then defaults
