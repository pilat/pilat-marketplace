---
description: "Use when a bug, test failure, crash, flaky test, hang, or unexpected behavior shows up and the reflex is to patch where it breaks. Find the root cause before any fix — reproduce, trace the bad value back to its origin, resist symptom-patches — then hand the diagnosis to brainstormer to explore the change."
---

# Root Cause

A bug shows itself at the symptom. The pull is to fix it right there, where the error appears — that feels like progress. It usually isn't: the symptom is downstream of the cause, and patching it either does nothing or trades one bug for a worse one — a nil-guard that stops a crash by silently dropping every payment turns a visible outage into invisible data loss. This skill exists to resist that pull long enough to understand what's actually broken — most of all when something is on fire in prod and the urge to just patch is strongest, because that's exactly when a blind patch does the most damage.

This is for things that are *caused* — a bad value with a chain behind it. A typo, a cosmetic string, an edit with nothing feeding it: that's not a root cause, it's an edit — just make it, this skill doesn't apply. But test "nothing feeding it" honestly — a value renamed in an earlier commit, or a shared constant read elsewhere, *is* a chain, and that's the skill. The exit only works before you understand it: the moment you can name where the bad value is born — whether you read it or traced it — it goes to brainstormer, however small the fix looks. Don't reclassify a bug you've already understood as "just an edit" to skip the handoff.

You don't ship the fix here. You're an investigator, not a repairman: you collect evidence, rule things out, and name the cause — you don't reach for the wrench. Then you hand it off. The fix is a change like any other, and it goes through the normal path once you know what you're changing and why.

## Evidence before theory

The fastest way to be wrong is to reason about what's "probably" happening. Don't — go look. Often the cause is already in front of you: read the code on the path, the actual values, the real output — that's frequently enough to see it. When it isn't, reproduce it and instrument until you can watch the value go bad. A theory you haven't watched fail is still a guess. When the user tells you what they're seeing, believe them and go look — don't explain why it shouldn't happen.

Read the error completely — the message, the stack, the line. It often names the cause outright, and skimming past it is how you spend an hour finding what was on screen the whole time.

## Use every instrument you have

Before you theorize, take stock of what you can actually reach. You often have more than the source in front of you: MCP servers that reach a database, logs, metrics, or traces; observability dashboards; architecture docs in the repo or a wiki that say how the thing is *meant* to work. A real cause frequently lives in data or runtime state you can only see through one of these — so look at what's available and use it, and when the evidence you need is behind something you can't reach, ask the user how to get at it instead of guessing around the gap.

Logs, metrics, and traces are read-only by nature — use them freely; reading them through Datadog, `kubectl logs`, or an APM is still just reading, even against prod. The bright line, and it's the one hard rule in this skill, is prod **data and access**: connecting to or querying a production database, getting a shell on a prod host, forwarding a port or copying data off one, or anything that writes or changes state. Reading telemetry is one thing; getting *onto* the box is another — `kubectl logs`, yes; `kubectl exec`, no. There — or anywhere you're **not certain** it's prod — **ask first**, and **never connect, exec, or run against it to investigate, not even "just to read."** Spending money or quota counts too: don't hammer a third-party endpoint to reproduce without a yes. We're investigators, not surgeons: we read, we collect, we rule out. When you're unsure whether something is safe, or even which environment you're looking at, stop and ask.

## Reproduce, then shrink

Can you trigger it on demand? If not, you're not debugging yet — you're collecting data until you can. And "yes" here isn't a nod, it's a possession: one named command — a test invocation, a script, a curl — that you have actually run, whose output you've read, and that goes red on *this* bug. Red means it asserts the user's exact symptom, not merely that something errored; the same verdict every run; seconds, not minutes; and it runs unattended. If it's intermittent, make it reproduce: loop it, seed the randomness, force the ordering or parallelism, pin the clock until the rate is high enough to debug against — an intermittent bug you can't trigger is one you can't confirm you've fixed. Until that command exists, every hypothesis is speculation, and the pull to skip this step and reason straight from the code is exactly the trap.

Once it reproduces, make it smaller: strip the scenario to the minimum that still breaks. A three-line repro tells you more than a three-hundred-line one, and it's what you'll use to confirm the cause. The command outlives the investigation, too: it rides along in the handoff as the ready-made verification for whatever fix the pipeline lands on — a red lamp the implementor inherits and has to turn green.

## Trace backward to where it's born

The cause usually sits upstream of the crash. Follow the bad value back: what passed it in, what called that, keep going until you reach where it originated — the empty string, the wrong path, the stale cache. That origin is where understanding lives, even if it's not where you'd fix it. When the chain is too deep to hold in your head, instrument it: log the value and a stack trace at the suspicious boundary, run once, read where it turns bad. Tag every debug log with one unique grep-able prefix — `[DEBUG-a4f2]`, say — because untagged logs survive into commits and tagged ones die on a single grep. In multi-component systems, check each boundary — what enters, what leaves — to find which layer breaks before diving into one.

## Hypothesize wide, test narrow

Testing one hypothesis at a time is right. *Generating* them one at a time is the trap: you anchor on the first plausible idea and spend the session proving it instead of questioning it. Before you test anything, lay out three to five candidate causes, ranked, each falsifiable — each names the prediction it makes: if X is the cause, changing Y makes the bug disappear. A hypothesis that can't name its prediction is a vibe; discard it or sharpen it until it can. Show the ranked list to the user before testing — they often re-rank it on sight ("we deployed a change to #3 yesterday") or kill candidates they've already ruled out; a cheap checkpoint that saves hours. If they're absent, proceed with your ranking — don't block.

Then test — one at a time, top of the list first, with the smallest change that would confirm or kill it, one variable at a time. That change is an experiment: you make it to watch the value come right or the test go green, and that's how you earn proof — keep it as evidence, not as the finished fix. Stacking three speculative changes and running tests tells you nothing: if it passes you don't know which mattered, if it fails you've added noise. If a hypothesis dies, take what you just learned back to the list — re-rank it, or form a better candidate from the evidence — don't pile a second fix on the first.

## When you're stuck

Two failed attempts that each spawn a new symptom usually means your model of the problem is wrong, not that you're one fix away. Stop adding attempts. Two doors out:

- **Go get information you don't have.** Read the docs, the SDK source, a working example in the same codebase. Search for the error if it's not yours. Stuck is often just missing knowledge.
- **Question the approach.** If every fix reveals coupling somewhere new, or each one needs "a bit of refactoring," the architecture may be the bug. That's not a failed hypothesis — it's a wrong foundation, and it's the user's call, not yours.

If the user redirects you — "stop guessing," "is that even happening," "we're stuck?" — they're telling you the current thread is dead. Drop it and pivot, don't defend it.

## When the cause is genuinely elsewhere

Sometimes honest investigation lands on environmental, timing, or external: a flaky network, a race outside your code, a dependency bug. That's a real conclusion — but only after you've actually traced it, not as an early exit. Most "no root cause" is investigation that stopped too soon. When it's real, say so, and note what would catch it next time.

## When the cause is found

You have the cause, evidence that it's the cause, and a minimal repro. You may already have the change that turns it green sitting in your working tree — that was an experiment to earn proof, not the delivery. Don't commit it and don't call the bug fixed; what to do with the tree is the user's call. The debug logs aren't — grep your tag and strip them, so the tree holds nothing but the experiment.

State the diagnosis plainly: the symptom, the cause and where it's born, the chain between them, the minimal repro, and the open questions about the fix. Then the part that earns the handoff: knowing where the bad value is born tells you the cause — not where the fix belongs or what shape it takes. Patch it at the symptom, or kill the whole class upstream with a guard? Fail loud, or default quiet? One layer, or validation at each boundary the bad value crossed? Those are design choices, the same kind every other change here goes through — which is why this one does too, instead of becoming a reflex commit.

So hand it off: state "Invoking pilat:brainstormer" and invoke `Skill("pilat:brainstormer")` in the same turn. Pass the diagnosis as the starting point, and the green diff, if you have one, as a *proposal* — name the cause, but don't pre-pick the fix's shape; that's what brainstormer is for. Don't use `Task(subagent_type=...)`.
