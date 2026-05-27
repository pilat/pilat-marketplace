# pilat-marketplace

Skills for thoughtful software development with Claude Code.

## Philosophy

Most AI coding tools optimize for speed: "just do it." These skills optimize for **understanding first**.

```
  /pilat:brainstormer          You describe what you need
         │
         ▼
  ┌─ brainstormer ──┐          Dialogue. Questions. Understanding.
  │  one question    │          No solutions yet.
  │  at a time       │
  └────────┬─────────┘
           │ auto
           ▼
  ┌─ handoff-plan ──┐          Decisions, constraints, tasks.
  │  writes plan.md  │          Everything a fresh session needs.
  └────────┬─────────┘
           │
           ▼
      plan.md saved              You review the plan.

  /pilat:implementor            New session. Fresh context.
         │
         ▼
  ┌─ implementor ───┐          TDD. Verification. Batches.
  │  executes plan   │          Checkpoints with you.
  └────────┬─────────┘
           │ auto
           ▼
  ┌─ handoff-review ┐          Parallel Sonnet subagents.
  │  fresh eyes on   │          Find what the author missed.
  │  the code        │          Auto-fix. Report.
  └────────┬─────────┘
           │
           ▼
      Ready for PR
```

Each phase has its own artifact. Each artifact is a checkpoint. Mistakes caught early are cheap.

## Installation

```bash
/plugin marketplace add pilat/pilat-marketplace
/plugin install pilat@pilat-marketplace
```

## Skills

### brainstormer

Explore before you build. One question at a time. Understanding > solutions.

```
/pilat:brainstormer I want to add caching to our API
```

### handoff-plan (auto)

Turn exploration into a plan. Decisions, constraints, tasks — everything a fresh session needs. Not user-invocable — called automatically by brainstormer when exploration is complete.

### implementor

Execute with discipline. TDD, verification, incremental progress.

```
/pilat:implementor
```

### handoff-review (auto)

External code review by parallel subagents after implementation. Not user-invocable — called automatically by implementor. Fresh eyes find what the author can't see.

### skill-creator

Meta-skill for creating Claude Code skills.

```
/pilat:skill-creator I need a skill for code review
```

### arch-init

Set up architecture documentation that knows how to maintain itself. Generates `ARCHITECTURE.md` and `CLAUDE.md` at project root, `docs/coding-style.md` and `docs/adr/` under `docs/` — all with project-specific sync rules embedded. Works for any language, not just modular codebases. Skips monorepos cleanly.

```
/pilat:arch-init
```

### arch-sync (auto)

Intelligent reviewer that compares docs vs code. Uses sync guidance from docs as a starting point, but applies judgment — catches drift the rules don't cover, ignores noise like test dependencies. Analysis-only: returns a drift report plus suggested doc edits; the caller (implementor) decides what to apply. Not user-invocable — called automatically by implementor after handoff-review.

## Agents

### humanizer

Rewrites text to sound natural. Kills AI-isms, varies rhythm, adds human texture. Used proactively by other agents when writing user-facing text.

## When to use what

| Situation | Use |
|-----------|-----|
| "I have an idea but not sure how to approach it" | brainstormer (skill) |
| "We discussed it, now need a plan" | handoff-plan (auto, via brainstormer) |
| "Plan is ready, let's build" | implementor (skill) |
| "Code is done, need review" | handoff-review (auto, via implementor) |
| "I want to create a new skill" | skill-creator (skill) |
| "Set up architecture docs for this project" | arch-init (skill) |
| "Did the implementation drift from the docs?" | arch-sync (auto, via implementor) |
| Writing PR comments, docs, commits | humanizer (agent, auto) |

## License

MIT
