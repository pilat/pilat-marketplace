---
description: "External code review by fresh-eyes subagents after implementation. Invoked by implementor and by brainstormer's mechanical path."
user-invocable: false
---

# Code Review

The implementation is complete — self-reviewed, tests passing, build clean. But you wrote this code, and self-review has a blind spot: you see what you meant, not what's there. Your job now: bring in fresh eyes to find what you missed.

## The Review

Launch parallel review subagents (Task) to examine the implementation from independent angles. Sonnet is a natural fit — fast, cost-effective, genuinely different pattern-matching from the implementing model. Scale the number of reviewers to the scope: a single-file fix needs one or two; a multi-service feature needs more.

Give each reviewer a distinct angle — "correctness and edge cases", "consistency with existing codebase patterns", "what's missing." Don't prescribe checklists; frame the angle and let the reviewer decide what matters.

What makes these reviews useful is context. Each reviewer needs:

- **The goal** — the task or plan that drove this implementation (paste the relevant section into each reviewer's prompt, don't summarize it away)
- **What changed** — the actual modifications (staged, unstaged, or committed — whatever reflects the current work)
- **Key decisions** — especially the non-obvious ones and their reasoning
- **Where to look** — the changed files plus the neighbors you already know matter: callers, the tests that cover them, the existing pattern the code imitates. Name them as starting points, not boundaries — the "what's missing" angle in particular must look beyond them.
- **What was consciously punted** — if the plan carried an "Out of scope, accepted" list, hand it to each reviewer: those corners are settled by a conscious call, not missed, and that binds the "what's missing" angle too. Only a plainly unsafe punt is worth reopening.

Without this, review degenerates into surface-level linting. Reviewers report findings with file paths and specific concerns, not general commentary.

## Fixing Findings

Fix what's clearly better — missing error handling, off-by-one errors, dead code, test gaps. For style questions, respect the author's choice. But when the style conflicts with existing codebase patterns, consistency wins — fix it.

Pause for the user when the right call isn't obvious, especially if a fix would change the design direction.

After fixing, re-run the same verification the implementor used. If fixes are cascading, surface the situation to the user rather than looping.

A clean review is a good outcome — it means the implementation was solid. Report it as such.

## Report

What was found, what was fixed, what needs user input. Brevity over ceremony.
