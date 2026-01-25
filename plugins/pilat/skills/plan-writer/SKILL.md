---
name: plan-writer
description: "Turn exploration into an implementation plan that a fresh session can execute independently."
---

# Plan Writer

The exploration is done. The decisions are made. What remains is a plan that a separate implementation session can execute without asking the original human.

## The Plan

Save to the project's existing plan or task folder if one exists. Otherwise, suggest a location. Filename: `YYYY-MM-DD-<topic-slug>.md`

The audience is a **fresh session that cannot ask the original human**:

- **Decisions**: what was chosen, why, and what was traded away — so the implementer understands the reasoning and doesn't try to reverse it.
- **Constraints**: explicit rules where it matters — what the implementer must or must not do.
- **Affected code**: actual files that will change and why, grounded in what was explored.
- **Tasks**: broken into steps with acceptance criteria. Each task should be completable in a single focused session. Consider how each task should be verified — this might mean test cases, but could also be manual checks, metrics, or integration tests depending on what the project already does.
- Describe what to build. Reference existing patterns instead of writing code. When no patterns exist, describe the approach — a brief example can anchor understanding, but the plan is not the place to write the implementation.

Not every exploration leads to a multi-task plan. A single task with the diagnosis and the fix is a valid plan. Match the ceremony to the problem.

## Principles

- Everything discovered during exploration is context. The contradictions, complications, and surprises matter most.
- Flaws in the plan become bugs in the implementation.
- Decisions that are hard to reverse deserve careful articulation. The rest can move quickly.
- If it's not needed for the goal, leave it out. When a task is inherently cross-cutting, break it into phases and be explicit about what's in this plan vs what's deferred.
- The output is a plan, not code.

## When done

Point the user to `pilat:implement` for execution.
