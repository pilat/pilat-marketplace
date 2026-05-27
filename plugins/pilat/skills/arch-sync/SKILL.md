---
description: Use after implementing changes to verify architecture docs match code. Triggers on "sync docs", "check architecture drift", "verify ARCHITECTURE", "reconcile architecture", or when CLAUDE.md instructs post-implementation sync.
user-invocable: false
---

# Arch-Sync

Analyze recent code changes against architecture docs. Report drift, propose specific doc edits. **Analysis only — never modify files.** The caller (implementor or user) decides what to apply.

## Goal

Compare what the docs say vs what the code does. Report meaningful drift, propose concrete edits. You're an intelligent reviewer, not a mechanical checker — use judgment.

## How to Work

### 1. Read the docs

Start with CLAUDE.md → ARCHITECTURE.md (project root) → docs/coding-style.md → docs/adr/ if present. Look for sections that explain how to verify sync — arch-init puts these in. These are your starting point, not your entire scope.

### 2. Understand what changed

Look at the diff. Pick whichever scope fits:
- Feature branch vs main (`git diff main...HEAD`)
- On main: since the last tag (`git diff $(git describe --tags --abbrev=0)...HEAD`), or last 20 commits if no tags

What areas of code were touched? This focuses your review.

### 3. Check for drift

Using the sync guidance from the docs AND your own understanding:

- Do documented structures still match code?
- Are dependencies/relationships still accurate?
- Did public interfaces change without doc updates?
- Are there new components the docs don't mention?
- Did someone add something that violates stated invariants?
- Does the overall description still ring true?
- Is CLAUDE.md still accurate? (read order, non-negotiables, skill guidance)

### 4. Report findings + propose edits

Produce two parts (see Output below).

## Your Judgment Matters

The sync rules in the docs are guidance, not a script. You will notice things the rules don't cover:

- **Handler in unexpected location:** Rules said "handlers live in internal/api/" but you found one in internal/services/. Report it — structural drift is exactly what arch-sync catches.

- **Test dependency not in docs:** Someone added testify or jest. Ignore it — test tooling isn't architecture. **Borderline test tools** (testcontainers, MSW with cross-cutting fixtures, anything that runs Docker or affects runtime topology) — flag with "borderline, likely architectural".

- **Pattern erosion:** ARCHITECTURE describes Repository pattern but you see direct DB calls creeping in. Report it with nuance — "either the pattern is obsolete (update docs) or this is tech debt (track it)."

- **Alignment toward documented style:** If a change moves code TOWARD the documented pattern (e.g., refactor from nested error handling to flat IIFE that matches docs/coding-style.md), note it as `in sync` with a positive line. Don't omit silently — confirming alignment is a signal the docs are working.

- **Rules vs your judgment:** When the doc's sync rule contradicts what you observe (e.g., rule says "check ls internal/" but the project no longer uses internal/), **your judgment wins.** Flag the rule itself as stale and propose updating it.

- **Granularity tiebreaker:** New top-level package or module → always flag. Single helper file inside an existing documented area → ignore unless it introduces new responsibility. When in doubt, flag — false positives are cheap, missed drift is expensive.

You're a smart colleague doing a review, not a linter running a checklist.

## ADR Candidates

If you see decisions that look significant or hard-to-reverse, flag them as potential ADR candidates:

- New major dependency (especially with runtime/cross-cutting impact)
- Architectural pattern change
- Migration from one approach to another
- New cross-cutting concern (email, telemetry, auth, queueing)

Threshold: if reverting the choice would require touching 3+ unrelated files OR creates an external dependency the team will have to maintain — ADR candidate. Don't write the ADR content — just note "consider ADR for X".

## What You Don't Do

- Don't modify any files — docs or code. You're analysis-only.
- Don't verify every line mechanically — use judgment about what matters.
- Don't report formatting differences or trivial mismatches.
- Don't suggest code changes — sync direction is docs ← code.
- Don't write ADR content — only flag candidates.
- Don't refuse to work if docs have no sync rules — do your best.

## Output

Two sections, in this order. Skip preamble.

### 1. Drift Report

Actionable list, one line per issue, grouped by area. If alignment was confirmed, include it as a positive.

```
ARCHITECTURE.md drift:

internal/notifications/ — new package (email sending), not documented
internal/domain/users/service.go — new Delete method not in documented API
go.mod — github.com/redis/go-redis added, not mentioned

docs/coding-style.md: in sync. Refactor in internal/api/orders.go aligned with documented IIFE pattern.

ADR candidates:
- Email delivery via SendGrid (new outbound dependency)

Suggested: 3 ARCHITECTURE updates, 0 docs/coding-style updates, 1 ADR to consider.
```

If nothing's wrong:
```
Docs are in sync with code (checked N files across M areas).
```

### 2. Suggested Edits

For each drift item that needs a doc change, propose a concrete edit. Show what to add/change in the doc, not in code. Format:

```
Edit: ARCHITECTURE.md
Where: "## Modules" section, after the storage entry
Add:
  ### notifications
  Email delivery via SendGrid. Outbound only.
  Public surface: `NewMailer(cfg) Mailer`, `Mailer.Send(ctx, msg)`.
```

Group edits logically. The caller decides whether to apply. **You never edit files yourself** — including .md files. Only propose.

## When Docs Have No Sync Rules

If ARCHITECTURE.md exists but has no explicit sync guidance (wasn't created by arch-init), do your best:

1. Compare documented structures against actual code
2. Check for obvious drift (new directories, changed interfaces, missing components)
3. Note that the docs lack sync guidance and propose adding it as one of the suggested edits

Don't refuse to work — partial docs are better than no review.
