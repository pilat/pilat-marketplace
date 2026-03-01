# pilat-marketplace

Skills for thoughtful software development with Claude Code.

## Philosophy

Most AI coding tools optimize for speed: "just do it." These skills optimize for **understanding first**.

The pipeline:
1. **brainstormer** — explore the problem through dialogue, resist jumping to solutions
2. **handoff-plan** — capture understanding as a plan (called automatically by brainstormer)
3. **implementor** — execute the plan with TDD discipline

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

### handoff-plan

Turn exploration into a plan. Decisions, constraints, tasks — everything a fresh session needs. Not user-invocable — called automatically by brainstormer when exploration is complete.

### implementor

Execute with discipline. TDD, verification, incremental progress.

```
/pilat:implementor
```

### skill-creator

Meta-skill for creating Claude Code skills.

```
/pilat:skill-creator I need a skill for code review
```

## Agents

### humanizer

Rewrites text to sound natural. Kills AI-isms, varies rhythm, adds human texture. Used proactively by other agents when writing user-facing text.

## When to use what

| Situation | Use |
|-----------|-----|
| "I have an idea but not sure how to approach it" | brainstormer (skill) |
| "We discussed it, now need a plan" | handoff-plan (auto, via brainstormer) |
| "Plan is ready, let's build" | implementor (skill) |
| "I want to create a new skill" | skill-creator (skill) |
| Writing PR comments, docs, commits | humanizer (agent, auto) |

## License

MIT
