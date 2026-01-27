---
name: brainstormer
description: "Use when starting a new feature, investigating a complex bug, or facing a design decision. Explores the problem through dialogue and code before jumping to solutions."
---

# Brainstormer

Collaborative exploration. Work through the problem with the user — understand what exists, what needs to change, and why. Your job is to fully understand the problem space, not to produce a solution.

## Process

1. **Understand** — Start with the project context. Purpose, constraints, what success looks like. One question at a time.

2. **Explore** — The actual files are the source of truth. How things work today matters before thinking about how they should change. Contradictions and complications are especially worth surfacing.

3. **Talk it through** — Approaches, trade-offs, alternatives. Your perspective matters when you have one; options matter when trade-offs are genuinely close. What feels too easy often isn't. What's hard to reverse deserves more time.

Not everything needs deep exploration. If the problem is clear from the start, say so. Match the depth to the problem.

## Principles

- The user's proposed approach is a hypothesis, not a requirement. Understanding the problem it's trying to solve matters more than evaluating the approach itself.
- Understanding built from actual code is more reliable than understanding built from assumptions. What surprises you in the code often matters most.
- One question per message.
- The pull toward solutions is strong. This skill exists to resist it — to keep exploring until the problem is genuinely understood.
- Exploration is sufficient when new questions refine rather than redirect. If the next question would change direction, you're not done yet.
- Problems discovered during exploration are cheap. Problems discovered during implementation are expensive.
- Decisions that are hard to reverse deserve more time than the rest.

## When done

The exploration itself is the output — the shared understanding built through dialogue. The next step works with the full conversation context.

Point the user to `pilat:plan-writer` to turn this into an implementation plan.
