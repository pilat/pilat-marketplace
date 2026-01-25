---
name: implement
description: Execute a plan through rigorous task-by-task implementation with TDD discipline and verification
---

# Implement

Execute a plan file with rigor: TDD discipline, verification before completion, self-review.

## Start

1. Read plan file (Goal, Decisions, Tasks)
2. Create tasks using TaskCreate
3. Ask once: **"TDD or implementation-first?"**

## Execution: Batches of 3

Work in batches of 3 tasks, then checkpoint with user.

### Per Task

```
Mark in_progress
      ↓
[TDD mode?]
├── YES → TDD Cycle (see below)
└── NO  → Implement → Verify → Self-review
      ↓
Mark completed (only after verification passes)
```

## TDD Discipline

**Rule:** No production code without a failing test first.

### Red-Green-Refactor

1. **RED:** Write ONE failing test
2. **VERIFY RED:** Run test, watch it fail. If it passes → test is wrong, fix it.
3. **GREEN:** Write MINIMAL code to pass (no extras)
4. **VERIFY GREEN:** Run test, watch it pass.
5. **REFACTOR:** Clean up, keep tests green.
6. **COMMIT:** After each green.

**Red flags:**
- Test passes immediately → didn't test the right thing
- Writing code before test → stop, write test first
- "Just this once" → no exceptions

## Verification Before Completion

**Rule:** No completion claims without fresh evidence.

Before marking ANY task completed:

1. **RUN** the verification command (test, build, lint)
2. **READ** full output
3. **CONFIRM** it actually passes (0 failures, exit code 0)
4. **ONLY THEN** mark completed

**Never:**
- Say "should pass" without running
- Trust previous run (run fresh)
- Mark done if tests are red

## Self-Review Before Handoff

After implementing, before marking complete, ask yourself:

- **Completeness:** Did I implement everything? Miss edge cases?
- **Spec compliance:** Does it match what the plan asked for?
- **Quality:** Is this my best work? Clear names? Clean code?
- **YAGNI:** Did I add anything not requested? Remove it.

Fix issues found during self-review.

## Divergence Protocol

When reality differs from plan:

1. **Acknowledge:** "Plan said X, but Y needed because [reason]"
2. **Ask:**
   - Update plan file?
   - Note as deviation and continue?
   - Discuss approach?
3. **Document:** If continuing with deviation, note it for final summary

## After Each Batch (3 tasks)

Pause and report:
- What was implemented
- Test results (paste output)
- Any deviations
- "Ready for next batch?"

## Subagent Delegation

If delegating task to subagent:

1. **Provide full task text** (don't make subagent read plan file)
2. **Include context:** where this fits, relevant patterns, constraints
3. **Require:** subagent follows same TDD/verification rules
4. **Review result:** verify tests pass before accepting

## Completion

When all tasks done:
1. Run full test suite
2. Show summary: what built, deviations, test results
3. Offer: "Create PR?" or "Ready for manual testing?"

## Commands

User can say anytime:
- `skip` - skip current task
- `status` - show progress (tasks completed/remaining)
- `pause` - stop, progress preserved in tasks
- `back` - revisit previous task
