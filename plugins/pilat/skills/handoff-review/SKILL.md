---
description: "Check implementation against its plan, then offer user-approved cold code review. Invoked by implementor and by brainstormer's mechanical path."
user-invocable: false
---

# Code Review

The code is implemented, self-reviewed, and verified with relevant checks. But you wrote it, and self-review has a blind spot: you see what you meant, not what's there. First check delivery against the plan; then let the user inspect the code before inviting code reviewers.

This skill stays in the implementation context and owns the review-and-fix cycle. The review subagents run cold; their findings return here for investigation and correction.

You and the reviewers are an autonomous implementation team entrusted with a carefully developed plan, or an agreed mechanical change. Own the technical follow-through: when a fix is clear or follows from the agreed decisions, inspected code and its neighbors, or verified project architecture, make it and verify it. The authors need not supervise those choices. Bring the user concrete, evidence-backed concerns the team cannot resolve within the agreed decisions, under the escalation rules below.

## Review Material

Each reviewer investigates read-only, without further delegation, from the review material rather than the implementation conversation:

- **The goal** — the task or plan that drove this implementation (paste the relevant section into each reviewer's prompt, don't summarize it away); on the mechanical path, the agreed change description stands in for the plan
- **What changed** — the actual modifications (staged, unstaged, or committed — whatever reflects the current work)
- **Key decisions** — especially the non-obvious ones and their reasoning
- **Where to look** — the changed files plus relevant neighbors: callers, tests, and existing patterns. These are starting points, not boundaries.
- **What was consciously punted** — if the plan carried an "Out of scope, accepted" list, hand it to each reviewer: those corners are settled by a conscious call, not missed, so a reviewer hunting for gaps should not flag them. Evidence that an exclusion undermines the goal or is plainly unsafe warrants a finding, not permission to change the decision.

Without this, review degenerates into surface-level linting. Reviewers report findings with file paths and specific concerns, not general commentary.

## 1. Cold Spec Review

For a planned implementation, launch one cold reviewer with the full plan and review material. Its question is whether the code fulfills the plan: decisions, constraints, accepted scope, task coverage, and acceptance criteria. This is a review of implementation against the plan, not another critique of the plan's design. Report concrete mismatches with the plan section, code evidence, and consequence, or give a clean verdict. Do not launch code reviewers alongside it.

Investigate its findings under the rules below and make one consolidated round of corrections. Run relevant checks, then ask the same reviewer to verify the corrections and any evidence-based refutations, using the continuation procedure below. If a consequential mismatch remains, pause for the user rather than starting another autonomous spec-fix round. A clean initial review needs no correction round. On the mechanical path there is no plan to audit; skip this stage rather than inventing one.

## 2. User Gate

After the spec review passes — on the mechanical path, once the change's checks pass — briefly report what changed and the verification results; for a planned change, confirm implementation matches the plan. This is readiness for the user's inspection, not a claim that code review has passed.

Ask "Run code review?" and end the turn. Wait for an explicit go-ahead before launching cold code reviewers; permission to implement or run the spec review is not that go-ahead. If the user requests changes, handle them first, rerun relevant checks, recheck affected plan requirements with the spec reviewer where applicable, and offer code review for the updated code. If they decline or defer, leave code review pending and stop. Once approved, routine fixes and re-reviews belong to the autonomous cycle below, without another approval checkpoint.

## 3. Cold Code Review

After approval, launch one code reviewer for a small change or two in parallel for a more complex one — at most two code-review roles, in addition to the one spec reviewer. Choose complementary angles from the plan and actual diff: behavior and contracts, state and concurrency, security, compatibility, or migrations as relevant. Several angles can belong to one reviewer; an angle does not require its own agent. Choose a model capable of the review's risk and complexity.

Give each reviewer this brief alongside its angle and review material:

> Review the actual code for defects, not task completion. Read the diff and relevant surrounding code; trace changed contracts through callers and consumers. For removed or replaced behavior, identify the guarantee it provided and where that guarantee now lives. Look for realistic inputs, states, and event orderings that cause incorrect behavior. The plan explains intent, not correctness. Your angle guides depth without excusing a consequential bug you notice elsewhere.
>
> Orient via the project's instructions and `docs/coding-style.md` and `docs/glossary.md` where they exist, then the surrounding code. Where conventions are undocumented, inspect surrounding code and tests; distinguish explicit requirements from inferred conventions and personal preferences. For a rule violation, cite the scoped rule and offending code, explaining the consequence rather than treating every difference as a defect.

### Findings and Triage

Use this priority scale, most serious first:

- **P0:** Critical defect requiring immediate attention before normal progress continues.
- **P1:** Serious failure of behavior, security, or data integrity.
- **P2:** A concrete defect of smaller impact that still needs correction.
- **Nit:** Non-blocking polish, not a disguised correctness issue.

For each P0–P2 finding, give file and line, the failure mechanism, realistic triggering conditions, and consequences. Keep severity separate from confidence: a real mechanism with an uncertain trigger is a candidate to investigate, not automatically a nit or a false positive. State the uncertainty. For nits, explain the concrete improvement. A clean review is valid; there is no minimum finding count.

## Fixing Findings

Fix all confirmed P0–P2 defects within the agreed decisions before completion. Fix nits when the change is simple, safe, and useful; leave subjective preferences with the author and avoid substantial refactoring for optional polish. Cheap describes the fix, not the reviewer's reasoning effort.

Check findings against the code and agreed behavior, combining duplicates. Fix confirmed defects; close refuted findings with a brief evidence-based reason. Neither a reviewer's assertion nor "the plan says so" settles correctness, and uncertainty alone does not refute a finding.

Investigate technical uncertainty and fix within the agreed decisions without a user checkpoint, even for a critical bug. Pause when the evidence requires a new choice: treatment of a new security or data-loss risk, incompatible behavior or contracts, a broken plan decision, or a change to scope, architecture, migration strategy, or an accepted exclusion. Show the contradiction, evidence, and recommendation; the user chooses whether a focused clarification or a separate design discussion is needed. On the mechanical path, existing behavior and the agreed change provide the boundary even without a plan.

After fixes, run relevant checks on the updated code, including regression checks where later fixes could break earlier work. For re-review, resume the existing reviewers by their agent/session identifiers if the harness supports continuation, preserving their conversations rather than launching the same roles again. Tell each which findings were addressed, what changed, and what verification ran; provide the fix diff or where to read it, plus any user-approved decision updates. Ask them to verify the fixes against their findings and check affected dependencies; broaden the pass when changes have wider consequences. If a code-review fix affects plan compliance, have the spec reviewer check the affected requirements too. If continuation is unavailable, give fresh reviewers the relevant review material, earlier findings, and concise change and verification summaries, keeping the same follow-up scope. The spec stage has its one-correction-round limit; during code review, if fixes stop converging or a consequential finding remains unresolved after investigation, surface the blocker instead of continuing to patch or reporting completion. New optional polish alone is not a reason to extend the cycle.

A clean review is a good outcome — it means the implementation was solid. Report it as such.

## Report

What was found, what was fixed or refuted (with a brief reason), what needs user input — findings numbered, so the reply can point at one; the numbers just appear, unremarked. Brevity over ceremony.
