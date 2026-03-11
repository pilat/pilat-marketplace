---
description: "Turn exploration into an implementation plan that a fresh session can execute independently."
user-invocable: false
---

# Plan Writer

The exploration is done. The decisions are made. What remains is a plan that a separate implementation session can execute without asking the original human.

## The Plan

Save to the project's existing plan or task folder if one exists. Otherwise, save in the working directory. Filename: `plan-YYYY-MM-DD-<topic-slug>.md`

The audience is a **fresh session that cannot ask the original human**:

- **Goal**: what we're trying to achieve and why, in one paragraph.
- **Decisions**: what was chosen, why, and what was traded away — so the implementer understands the reasoning and doesn't try to reverse it.
- **Constraints**: explicit rules where it matters — what the implementer must or must not do.
- **Affected code**: actual files that will change and why, grounded in what was explored.
- **Tasks**: broken into steps with acceptance criteria. Each task should be completable in a single focused session. Consider how each task should be verified — this might mean test cases, but could also be manual checks, metrics, or integration tests depending on what the project already does.
- Describe what to build. Reference existing patterns instead of writing code. When no patterns exist, describe the approach — a brief example can anchor understanding, but the plan is not the place to write the implementation.

Not every exploration leads to a multi-task plan. A single task with the diagnosis and the fix is a valid plan. Match the ceremony to the problem.

## Principles

- Everything discovered during exploration is context. The contradictions, complications, and surprises matter most.
- Decisions that are hard to reverse deserve careful articulation. The rest can move quickly.
- If it's not needed for the goal, leave it out. When a task is inherently cross-cutting, break it into phases and be explicit about what's in this plan vs what's deferred.
- The output is a plan, not code.

## Before You're Done

You have something the implementer won't: the full exploration context. Every decision, every complication, every rejected alternative. The plan is a lossy compression of all that — and the implementer will build from the compressed version. What gets lost here becomes a wrong assumption there.

### Self-check

**Enumerate first.** Before looking at the plan again, go through the exploration and list every decision made, every constraint stated, every edge case raised, every rejected alternative as a numbered list. Commit to what SHOULD be in the plan before checking what IS.

**Verify.** For each item, note where in the plan it appears or mark it MISSING. Add every MISSING item to the appropriate section.

**Scan for implicit knowledge.** Search the plan for hedging language: "handle appropriately," "as discussed," "the usual approach," "relevant files," "etc." These are fingerprints of things you know but didn't write down — places where the implementer will have to guess. Replace each one with specifics.

### Fresh-eyes check

Launch review subagents (Task) to read the plan cold. Provide ONLY the plan file path — do not summarize the exploration, do not add "helpful context," do not explain what the plan is about. The subagent must experience exactly what the implementer will: the plan file and the codebase, nothing else.

**One subagent (straightforward plan):** "You are about to implement this plan in a fresh session with no other context. Read it and identify every point where you would need to stop and ask a question, make an assumption, or guess. For each, say what's missing and what you'd need to know to proceed."

**Two subagents (multi-task or cross-cutting plan):** The first is the builder — same brief as above, walks through each step, finds where they'd get stuck. The second is the reviewer: "Someone implemented this plan. You're reviewing their work. Based ONLY on what the plan says, how would you verify the implementation is correct? Where could two reasonable implementers produce meaningfully different results from these same instructions?"

### Integrate findings

Collect findings from self-check and subagents. For gaps — missing information needed for implementation — fix the plan directly. If a finding contradicts a decision made during exploration, surface it to the user rather than overriding it.

## When done

Point the user to `pilat:implementor` for execution.
