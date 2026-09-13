---
description: Use when the user has a plan file (from plan-handoff or similar) and wants it executed. Also use when user says "implement", "execute the plan", "build this", or passes a plan file path.
---

# Implementor

Execute a plan file with rigor: verification before completion, self-review.

## Start

1. **Find the plan.** User may provide a path, or look for the most recent `plan*.md` / `PLAN*.md` in the working directory. If no plan found, ask.
2. Read plan file (Goal, Decisions, Tasks — and "Out of scope, accepted": corners consciously punted, yours to neither handle nor wonder about)
3. Create tasks using TaskCreate

## Execution

Work through all tasks sequentially. Do not pause between tasks — keep going until all are done or you hit a blocker. If a task specifies TDD, follow red-green-refactor for that task.

**Opportunistic parallelism (optional).** Sequential is the default — always safe. But when a plan has several tasks that are genuinely independent — they don't touch the same code and nothing in the plan makes one depend on another — and you have tooling to run subagents in parallel (a Workflow), you MAY fan them out. When in doubt, stay sequential. If you do parallelize, verify the combined result before moving on.

### Per Task

1. Mark `in_progress`
2. Implement
3. Verify — run the relevant check (test, build, lint, whatever applies). Read the output, confirm it passes. Do not claim "should pass" without running.
4. Mark `completed`

## Self-Review

Before marking the final task complete, review all changes holistically:

- **Completeness:** Everything the plan asked for?
- **Spec compliance:** Matches the plan's decisions?
- **Correctness:** In what realistic scenario could the new code produce the wrong result, or break existing behavior?
- **YAGNI:** Added anything not requested? Remove it.

Fix implementation issues within the plan's decisions directly, without pausing for approval. Before handing off, run relevant checks on the final state, including self-review fixes, to confirm later changes haven't broken earlier work. Earlier passing checks may no longer cover the code as it now stands. Then proceed to handoff-review (see Completion).

## Divergence Protocol

When reality differs from plan:

1. **Acknowledge:** "Plan said X, but Y needed because [reason]"
2. **Decide:** Resolve implementation issues within the agreed decisions and keep going. A detail missing from the plan is not itself a question for the user when the answer follows from the plan and verified code. Severity determines urgency, not whether permission is needed to fix a bug. When the evidence requires a new design choice, pause with the contradiction, evidence, and your recommendation. The user decides whether a focused clarification is enough or a separate design discussion is needed; a recorded decision changes only with their agreement.
3. **Document:** Note deviations for the final summary.

Reasons to involve the user include a newly discovered security or data-loss risk whose treatment needs a new decision; a compatibility or contract conflict requiring a choice of behavior; a plan decision that code or checks show cannot hold; or a necessary change to scope, architecture, migration strategy, or an accepted exclusion. Also pause when fixes stop converging — one repair repeatedly breaks another, or a consequential finding remains unresolved after investigation. Initial uncertainty calls for investigation, not an immediate question. With an unresolved consequential finding, report the blocker rather than completion.

## Subagent Delegation

If delegating task to subagent:

1. **Provide full task text** (don't make subagent read plan file)
2. **Include context:** where this fits, the exact files the task touches, relevant patterns, constraints
3. **Review result:** verify it works before accepting

## Completion

When all tasks done:

1. **Invoke `Skill("pilat:handoff-review")`** — this is NOT optional. Do NOT use `Task(subagent_type=...)` and do NOT skip this step. The skill runs in your implementation context: one cold spec reviewer checks delivery against the plan, then you offer code review and wait for the user's go-ahead. Only then do one or two cold code reviewers run. The skill owns findings integration, fixes, verification, and re-review. Its pre-review progress report is expected; a request for input pauses the cycle, and code review remains pending until approved and performed. Do NOT report the full workflow complete before that cycle is resolved.
2. **Architecture doc sync.** If `ARCHITECTURE.md` exists at project root, invoke `Skill("pilat:arch-sync")`. It detects drift, fixes the docs itself, and returns a report of what changed — no user prompts, no decisions for you to make. Include its report verbatim in the summary. One exception: if the report flags ADR candidates, don't let them die there — surface them in the summary as an explicit offer and draft each one only after the user confirms; ADRs need the user's framing, not yours. No conflict with the ADR trigger in the project's CLAUDE.md — that one fires on decisions you consciously made and own; these are inferred from the diff, where the framing is a guess. Author the ones you own, offer the ones arch-sync guessed. If `ARCHITECTURE.md` does NOT exist — skip this step **entirely and silently**: do not mention arch-sync, do not say "skipped", do not include doc sync in the summary. This runs **after** review so any review-driven code changes are included.
3. Show summary: what built, deviations, review findings. Include the arch-sync report only if step 2 actually ran.
4. Offer: "Create PR?" or "Ready for manual testing?"

## Commands

User can say anytime:
- `skip` - skip current task
- `status` - show progress (tasks completed/remaining)
- `pause` - stop, progress preserved in tasks
- `back` - revisit previous task
