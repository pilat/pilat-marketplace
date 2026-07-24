---
description: "Use when asked to create, write, or build a Claude Code skill, or to prune and rework an existing SKILL.md."
---

# Skill Creator

A skill exists to bend a stochastic process toward the same shape every run — same process, not same output — and it earns its keep only where it changes what the model does versus what the model would do anyway. A skill is a delta. That's the trap in writing one: skill files grow by accumulation, because adding a line feels safe and deleting one feels risky, until the file is mostly restated documentation the model already knows, burying the three sentences that actually steer. Every line spends context and attention, so a bloated skill drowns its own load-bearing lines. The craft is less "write good instructions" than "find the delta and cut everything else."

The workflow: understand what's needed, draft, test the draft on a clean-context subagent, revise, ship. One instrument runs through all of it:

## The no-op test

For every sentence in a draft, ask: does this change the model's behavior versus its default? If it doesn't, it's a no-op — delete the sentence whole rather than trimming words from it. "Be thorough," "quality over quantity," "never ask what can be inferred": the model already does all of that, so those lines pay rent to say nothing. Restated platform documentation — frontmatter syntax, line-count guidance — fails the same way. Run the test sentence by sentence, not section by section; a mostly dead section can still hide one live line worth keeping. The other thing to hunt while you're in there is duplication: the same meaning stated in two places doubles the maintenance and inflates the idea's apparent importance — collapse it to one authoritative spot.

The test is model-relative, not taste-relative: if you and the user disagree about whether a line is a no-op, you disagree about what the default is, and you settle that by running the draft on a subagent, not by debate. And when the ask is to prune an existing skill rather than write a new one, this test *is* the job.

## Understand what's needed

Brainstormer rules: one question per message, an offered default with each ("X, unless you'd rather Y?"), stop when the next answer wouldn't change the draft. Two or three questions usually suffice, and the one that matters most is the delta question — **what does the model get wrong today, without this skill?** The answer is the skill. Triggers and hard boundaries are the only other things reliably worth asking about; tone and structure you can decide yourself.

Surface one design decision early, because it changes the frontmatter: who invokes this? A model-invoked skill keeps its description, and that description sits in the context window every turn of every session — it pays rent so the agent can reach the skill on its own. A user-invoked skill (`disable-model-invocation: true`) pays no rent, but the human becomes the index: they must remember it exists and type its name. If the skill would only ever fire by hand, make it user-invoked and pay nothing.

## Write the delta

**Persuade, don't command.** Name the trap and give the reason, instead of stacking "you MUST": a model that understands why can spot the exception a blanket rule would steamroll. Save hard rules for the seams — handoffs, destructive operations, places where a skipped step breaks something silently downstream.

**Phrase the target, not the ban.** "Don't think of an elephant" names the elephant: a prohibition drags the forbidden behavior into context, where it half-reads as an instruction. Write what to do — "one-line comments," not "never write verbose comments." A prohibition earns its place only as a hard guardrail you can't phrase positively, and even then, pair it with the positive move so attention lands on what to do.

**Hunt for leading words.** A leading word is a compact concept already in the model's pretraining — *tracer bullet*, *tight*, *red*, *fog of war* — that anchors a whole region of behavior in one token by recruiting priors the model holds for free. A sentence gesturing at one idea ("fast, deterministic, low-overhead") collapses into the word (*tight*). Wherever a draft passage sprawls, ask whether one strong word could replace it — and prefer pretrained words to coined ones, which cost you in definition what pretrained words give free. A leading word too weak to beat the default (*thorough*) is itself a no-op; the fix is a stronger word (*relentless*), not more sentences. A role is a leading word too — often the strongest: *a cautious senior DBA* recruits an entire prior of judgment in four words. When domain judgment is the skill's substance, give it one — mechanism over costume (years-of-experience backstories are no-ops), and light for factual work, where a confident voice resists saying "I don't know."

**Make done checkable.** A step that ends on "until understanding is reached" invites the model to declare victory early; one that ends on "every modified file accounted for" resists it. When the draft has steps, end each on a condition the model can check — done or not-done, no judgment call.

**Let structure follow branches.** What every run needs stays inline in SKILL.md; heavy reference that only some paths reach goes in a sibling file, pointed at from the line that needs it. The pointer's wording, not the file behind it, decides whether the model actually gets there.

## The description pays rent

A model-invoked description is loaded every turn of every session, so it earns harsher pruning than the body. Triggers only: front-load the word most likely to be in the air when the skill should fire, one trigger per genuinely distinct branch, no synonym padding, and never past 1024 characters — the platform truncates the rest. And keep the workflow out of it — a description that summarizes what the skill does tempts the model to act on the summary and never open the body, a failure mode confirmed in testing:

```yaml
# Summarizes — the model may follow this line instead of reading the skill
description: Reviews Go code by checking formatting, then patterns, then tests

# Triggers — the model opens the skill to find out how
description: Use when reviewing Go code or PRs, or when asked to check idioms
```

## Test on a clean subagent

You can't review your own skill. You hold the whole conversation — the requirements, the reasoning, everything that went without saying — while the future consumer holds only the file. A clean-context subagent *is* that consumer: it sees exactly what the skill says and nothing else, so the gaps invisible to you surface immediately. This seam gets the one hard rule here: a draft goes through a subagent before it ships.

Run one realistic scenario per genuinely distinct branch of the skill:

```
Task tool, general-purpose subagent, one per scenario:

  You have this skill loaded:
  [full draft, verbatim]

  A user asks: "[realistic request for this branch]"

  Work the task as the skill directs. Where the skill leaves you
  unsure what to do, say so explicitly rather than improvising.
```

Branches aren't the only scenarios worth running. Add two mean ones: a request that only half-matches the trigger (does the skill fire when it shouldn't?) and one where the user pushes against the skill's guidance (does it hold or fold?). Over-firing and folding are where skills fail in the wild, and neither is a branch you'd otherwise exercise.

Read the transcript as diagnosis. The subagent asks "what do I do here?" — an instruction is missing. It makes a choice you didn't intend — that guidance was a no-op or got buried. It skips a step — the emphasis or placement is wrong. Each gap becomes an edit or a question back to the user; after changes big enough to shift behavior, test again.

## Ship

Summarize the finished skill in three or four phrases and get a yes. Ask where it lives — this project (`.claude/skills/`) or personal (`~/.claude/skills/`, honoring `$CLAUDE_CONFIG_DIR` if set) — and write `<location>/<skill-name>/SKILL.md`; the directory name is the skill's name.

Before writing, hold the draft to its own standard one last time: a skill built on the no-op test that carries dead weight is a bug, not a style choice.
