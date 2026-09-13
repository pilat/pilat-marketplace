---
description: "Turn exploration into an implementation plan that a fresh session can execute independently."
user-invocable: false
---

# Plan Writer

The exploration is done. The decisions are made. What remains is a plan that a separate implementation session can execute without asking the original human.

## Unresolved Decisions

By this point every decision should be made. If you hit one that isn't — and the answer isn't obvious from the exploration or the codebase — ask the user now, while they're here. Use AskUserQuestion for a clean choice between options; ask directly for anything open-ended.

**Never defer a question into the plan.** No "open question for the human," no "ask before implementing," no "resolve before task N." The implementer cannot ask the original human — a plan containing a question is a plan that blocks. Every decision lands in the plan as a decision, with its reasoning.

If a fact cannot be established and it affects the design, bring that uncertainty to the user now. A fallback or reduced scope is itself a design choice for the user, not an automatic way to finish the plan. Deployment assumptions deserve the same treatment as assumptions about the code.

## The Plan

Save to the project's existing plan or task folder if one exists. Otherwise, save in the working directory. Filename: `plan-YYYY-MM-DD-<topic-slug>.md`

The audience is a **fresh session that cannot ask the original human**:

- **Goal**: what we're trying to achieve and why, in one paragraph.
- **Decisions**: what was chosen, why, and what was traded away — so the implementer understands the reasoning and doesn't try to reverse it.
- **Constraints**: explicit rules where it matters — what the implementer must or must not do.
- **Out of scope, accepted**: corner cases raised and consciously not handled — each named, with the one-line why. Recorded so the implementer neither handles them nor wonders, and review doesn't flag them as missed.
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

Do this in the same context that explored the problem and wrote the draft. That context holds both the spoken decisions and the assumptions behind them; a separate reviewer would have to reconstruct them.

**Enumerate first.** Before looking at the plan again, go through the exploration and list every decision made, every constraint stated, every edge case raised, every rejected alternative as a numbered list. Commit to what SHOULD be in the plan before checking what IS.

**Verify.** For each item, note where in the plan it appears or mark it MISSING. Add every MISSING item to the appropriate section.

**Scan for implicit knowledge.** Search the plan for hedging language: "handle appropriately," "as discussed," "the usual approach," "relevant files," "etc." These are fingerprints of things you know but didn't write down — places where the implementer will have to guess. Replace each one with specifics.

**Check your assumptions.** Consider what went unsaid, where your confidence is weakest, and what would most plausibly break after shipping. For each new component or boundary, check why it needs to be separate and whether the codebase already does its job. Use evidence from the exploration while it remains current; inspect the code, documentation, or available data where evidence is missing. A confident recollection alone does not establish a fact.

Resolve these findings using the rules below before inviting the cold critic. Incorporate the facts and reasoning the implementer needs into the plan's decisions, constraints, and tasks. Questions that change the design go to the user now, and their answers shape the draft. The cold critic reads the resulting plan without the author's self-check notes, so missing context stays visible.

### Fresh-eyes check

Launch one review subagent (Task) to read the plan cold. Give it the plan file path and the review brief below, without conversation history, a summary of the exploration, or self-check notes. Its evidence is the same material the implementer will have: the plan and the repository. This is one read-only review, without further delegation.

Repo artifacts are not context contamination. The critic can orient via `ARCHITECTURE.md` and `docs/glossary.md` if they exist, then follow relevant code and tests. Docs are starting points; claims about behavior or existing implementations need evidence from the actual code.

Give the critic this brief:

> You are about to implement this plan in a fresh session. Can you build the intended result and verify it without guessing at behavior or design? Identify consequential gaps, contradictions, or ambiguities, then check the design against the code: existing invariants, interactions, and plausible failure scenarios. For components the plan creates, look for existing implementations that already do the same job; a shared primitive or a partial match alone is not duplication. Treat consciously accepted exclusions as settled unless evidence shows they undermine the goal or are plainly unsafe. Report numbered findings with the relevant plan section, code evidence where applicable, and the consequence. A clean review is a valid result.
>
> Trace each planned behavior through the relevant existing code, including where proposed additions would connect. Follow the affected contracts across callers, state changes, and consumers; check that tasks agree where they meet. For behavior being replaced or removed, identify what the existing code guarantees and where that guarantee will live afterward. Use this tracing to ground findings, not to write the implementation or prescribe every coding step. Spend depth on consequential boundaries rather than unrelated code.

### Integrate findings

These rules apply to both the author's self-check and the cold review. Check each finding against the plan and its evidence, and combine duplicates before deciding what it changes.

- Fill technical gaps directly when the answer follows from the agreed decisions and verified facts. This restores missing information without introducing a new design choice.
- Close factually refuted findings with a brief evidence-based rationale in your working notes. Routine dismissals need no user response. Uncertainty alone is not a refutation.
- Bring findings that change behavior, scope, accepted risk, or a recorded decision to the user, with your recommendation. A new mechanism, component, or migration is a design addition even when a critic suggests it. Explain the discovery and the specific choice it affects; preserve the rest of the discussion. Number the findings you surface so the user can refer to them directly.

The user settles new design choices before they enter the plan. An unhandled corner case gets a decided treatment or an explicitly accepted exclusion with its reason. Resolve any remaining design uncertainty with the user before completing the plan; a later implementation session is not where the missing discussion happens.

After cold-review changes, resume the existing critic by its agent/session identifier if the harness supports continuation, preserving its conversation rather than launching the same role again. Tell it which findings were addressed and what changed, including decisions settled with the user and now recorded in the plan; point it to the updated sections. Ask it to verify the changes against its findings and check affected dependencies. A broader pass is warranted when the change alters the design throughout the plan. If continuation is unavailable, give a fresh critic the review brief, updated plan path, earlier findings, and a concise change summary, without the exploration history, so it can review the update without repeating the whole investigation.

## Decision records

With findings integrated the decisions are final — this is where an ADR is written, if the project keeps `docs/adr/` and the exploration settled a choice that's significant and hard to reverse: the tradeoff someone questions in six months. Next free number, project's TEMPLATE, Context and Alternatives straight from the conversation context this session still holds — the plan transports the outcome, not the reasoning, and no later session can reconstruct the why. Writing one starts with reading what's already there: if the dialogue reversed a decision an existing ADR records, the new ADR supersedes it — the old file stays, its Status becomes `Superseded by ADR-NNNN`, its number is never reused, and the new one links back. The trail is the point: a reversal that erases its predecessor reads, six months later, as if the first option was never considered — and invites someone to propose it again. Easily-reversed choices don't earn one.

## When done

Point the user to `pilat:implementor` for execution.
