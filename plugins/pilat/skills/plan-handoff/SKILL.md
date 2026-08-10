---
description: "Turn exploration into an implementation plan that a fresh session can execute independently."
user-invocable: false
---

# Plan Writer

The exploration is done. The decisions are made. What remains is a plan that a separate implementation session can execute without asking the original human.

## Unresolved Decisions

By this point every decision should be made. If you hit one that isn't — and the answer isn't obvious from the exploration or the codebase — ask the user now, while they're here. Use AskUserQuestion for a clean choice between options; ask directly for anything open-ended.

**Never defer a question into the plan.** No "open question for the human," no "ask before implementing," no "resolve before task N." The implementer cannot ask the original human — a plan containing a question is a plan that blocks. Every decision lands in the plan as a decision, with its reasoning.

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
- **Warm-context notes** (draft only — stripped before the plan is final, see Integrate findings): the unsaid half of the exploration, as numbered statements each carrying a confidence level — assumptions never stated aloud, the spots you're least confident about, the most likely three-month failure. Statements, not questions — a question here is a decision that escaped. One subsection is mandatory: for every new package, component, or boundary this plan creates, one line on why it must be separate from everything else the plan creates and from what already exists, with confidence — free recall is blind to confidently-held structural assumptions, and separateness asserted without a reason is how duplication ships. This section's audience is the critics, never the implementor and never the user.

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

Repo artifacts are not context contamination. Tell each critic except Reuse to orient via `ARCHITECTURE.md` and `docs/glossary.md` if they exist, before grepping — the implementer will have those too, so reading them keeps the simulation faithful and saves the re-discovery cost. Reuse gets no pointer: its job is exhaustive search of the actual code, and a doc index invites stopping at what's documented.

Two families of angles — transport (can a stranger run this?) and design (does what it builds hold up?):

- **Builder** (always): "You are about to implement this plan in a fresh session with no other context. Read it and identify every point where you would need to stop and ask a question, make an assumption, or guess. For each, say what's missing and what you'd need to know to proceed."
- **Reviewer** (multi-task or cross-cutting plan): "Someone implemented this plan. You're reviewing their work. Based ONLY on what the plan says, how would you verify the implementation is correct? Where could two reasonable implementers produce meaningfully different results from these same instructions?"
- **Premortem** (any plan that carries design decisions): "This plan shipped and broke in production. What most plausibly broke? Hunt corner cases the plan doesn't handle, invariants materialized in types, tests, or assertions that it ignores, interactions with existing behavior — and check each claim against the actual code before reporting it. Treat what the plan marks out-of-scope-accepted as settled unless it's plainly unsafe. If nothing plausibly breaks, say so — don't manufacture findings."
- **Reuse** (when the plan builds new components): "For each component this plan builds new, search the codebase for an existing implementation. Reinvention means something existing already does the same job — not a shared primitive it builds on, not a partial or differently-scoped match. If nothing genuinely equivalent exists, say so."

Every critic except Reuse carries one added duty: verdict each warm-context note against the actual code — CONFIRMED with evidence, REFUTED with evidence, or UNVERIFIABLE. A verdict without evidence is a rubber stamp, and a rubber stamp here launders a doubt into a fact. Reuse never sees the section: warm-context notes are author context in its purest form, and Reuse's no-pointer rule exists precisely to keep author context out of its search.

Scale by trigger, not fixed buckets: run each critic whose trigger the plan meets — builder always, premortem wherever design decisions ride, reuse wherever new components do, reviewer for multi-task or cross-cutting work. A small plan can hand one subagent two angles — except Reuse, which never folds: its job is exhaustive code search, and pairing it with a doc-fed angle reopens the exact shortcut its no-pointer rule closes. Fold premortem into the builder or reviewer instead.

### Integrate findings

Collect findings from self-check and subagents. For gaps — missing information needed for implementation — fix the plan directly. If a finding contradicts a decision made during exploration, surface it to the user and wait for their answer — do not override the decision yourself, and do not park it in the plan as an open question. The same discipline covers additions: a finding that introduces a mechanism, component, or migration the exploration never discussed is new design, not a gap-fix — however sound it looks, it goes to the user with the critic's reasoning and your own read, and enters the plan only on their yes. Critics exist to see what the dialogue missed; deciding still happens warm. A plan that quietly gained machinery nobody discussed reads, to the person who approved a different design, as the author's invention — and costs the trust the whole pipeline runs on. A corner case the plan doesn't handle gets one of exactly two endings: handled — the plan gains the task or constraint — or consciously punted with the user's say-so and recorded under out-of-scope, accepted. If it was never discussed during exploration, it goes to the user now, while they're still here. Dismissing a critic's finding as noise also happens aloud, in one line — the user can't veto a dismissal they never saw. Everything surfaced to the user here goes as one numbered list, dismissals included — the answer comes back by number; the numbers just appear, unremarked. The plan is not done while a blocking question stands unanswered.

Warm-context notes resolve here too, each into exactly one ending. Confirmed with evidence — dropped, no trace needed. Refuted by code evidence — the plan changes, and gains a one-line record under `Overturned in review`: the implementor should know the earlier idea was considered and struck, not wonder whether it was missed. Refuted as critic opinion, or touching a decision the user made aloud in dialogue — a numbered question to the user; the rule above still wins, spoken decisions are never overturned silently. Unverifiable because it's a property of the deployment environment — recorded under out-of-scope, accepted with the assumption named and a fail-loud constraint added, not parked as a question to a user who may already be gone; unverifiable at the design level goes to the user by number. Then the section is stripped: the final plan carries decisions, constraints, and tasks — never doubts. A hedge that survives to the implementor is a decision that escaped twice.

## Decision records

With findings integrated the decisions are final — this is where an ADR is written, if the project keeps `docs/adr/` and the exploration settled a choice that's significant and hard to reverse: the tradeoff someone questions in six months. Next free number, project's TEMPLATE, Context and Alternatives straight from the conversation context this session still holds — the plan transports the outcome, not the reasoning, and no later session can reconstruct the why. Writing one starts with reading what's already there: if the dialogue reversed a decision an existing ADR records, the new ADR supersedes it — the old file stays, its Status becomes `Superseded by ADR-NNNN`, its number is never reused, and the new one links back. The trail is the point: a reversal that erases its predecessor reads, six months later, as if the first option was never considered — and invites someone to propose it again. Easily-reversed choices don't earn one.

## When done

Point the user to `pilat:implementor` for execution.
