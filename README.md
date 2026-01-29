# pilat-marketplace

Skills for thoughtful software development with Claude Code.

## Philosophy

Most AI coding tools optimize for speed: "just do it." These skills optimize for **understanding first**.

The pipeline:
1. **brainstormer** — explore the problem through dialogue, resist jumping to solutions
2. **plan-writer** — capture understanding as a plan that a fresh session can execute
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

### plan-writer

Turn exploration into a plan. Decisions, constraints, tasks — everything a fresh session needs.

```
/pilat:plan-writer
```

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

## When to use what

| Situation | Skill |
|-----------|-------|
| "I have an idea but not sure how to approach it" | brainstormer |
| "We discussed it, now need a plan" | plan-writer |
| "Plan is ready, let's build" | implementor |
| "I want to create a new skill" | skill-creator |

## License

MIT
