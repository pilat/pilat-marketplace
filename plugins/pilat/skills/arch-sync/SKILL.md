---
description: Use after implementing changes to keep architecture docs synced with code. Triggers on "sync docs", "check architecture drift", "verify ARCHITECTURE", "reconcile architecture", or when CLAUDE.md instructs post-implementation sync.
user-invocable: false
---

# Arch-Sync

Find drift between architecture docs and code, fix the `.md` files directly, report what was changed. Never edit source code. Never prompt the user.

## Goal

Keep docs aligned with reality. Read the docs, look at recent code changes, fix drift in the doc files yourself. You're a smart colleague doing the upkeep, not a linter generating a report someone else has to act on.

## Scope of Writes

You may edit only these files (when they exist at the project root):
- `ARCHITECTURE.md`
- `CLAUDE.md`
- `docs/coding-style.md`
- Any file under `docs/adr/`

**Never** edit source code, configs, or any other file. If a code change would be the right fix, flag it in the report — don't make it.

## How to Work

### 1. Read the docs

Start with CLAUDE.md → ARCHITECTURE.md (project root) → docs/coding-style.md → docs/adr/ if present. Look for sections that explain how to verify sync — arch-init puts these in. These are your starting point, not your entire scope.

### 2. Understand what changed

Look at the diff. Pick whichever scope fits:
- Feature branch vs main (`git diff main...HEAD`)
- On main: since the last tag (`git diff $(git describe --tags --abbrev=0)...HEAD`), or last 20 commits if no tags

What areas of code were touched? This focuses your review.

### 3. Detect drift

Using the sync guidance from the docs AND your own understanding:

- Do documented structures still match code?
- Are dependencies/relationships still accurate?
- Did public interfaces change without doc updates?
- Are there new components the docs don't mention?
- Did someone add something that violates stated invariants?
- Does the overall description still ring true?
- Is CLAUDE.md still accurate? (read order, non-negotiables, skill guidance)

### 4. Fix the docs

For each drift item that calls for a doc change, edit the relevant `.md` file directly with the Edit tool. No "suggested edits" output, no "should I apply?" prompt — fix it.

If a fix is ambiguous (two equally valid wordings, unclear which doc owns it) — pick the reasonable one and note the choice in the report.

### 5. Report what was done

Output the report (see Output below). Past tense — describe what you changed, not what you propose. No questions, no prompts.

## Your Judgment Matters

The sync rules in the docs are guidance, not a script. You will notice things the rules don't cover:

- **Handler in unexpected location:** Rules said "handlers live in internal/api/" but you found one in internal/services/. Structural drift is exactly what arch-sync catches — if the code change was intentional, update the doc to match. If it looks like tech debt, flag it in the report and leave the doc alone.

- **Test dependency not in docs:** Someone added testify or jest. Ignore it — test tooling isn't architecture. **Borderline test tools** (testcontainers, MSW with cross-cutting fixtures, anything that runs Docker or affects runtime topology) — update the docs, they're architectural.

- **Pattern erosion:** ARCHITECTURE describes Repository pattern but you see direct DB calls creeping in. Don't quietly update the doc to legitimize tech debt — flag it as "code violates documented pattern" in the report and leave the doc alone. Sync direction is docs ← code only when the code change was deliberate.

- **Alignment toward documented style:** If a change moves code TOWARD the documented pattern (e.g., refactor from nested error handling to flat IIFE that matches docs/coding-style.md), note it as `in sync` with a positive line. Confirming alignment is a signal the docs are working.

- **Rules vs your judgment:** When the doc's sync rule contradicts what you observe (e.g., rule says "check ls internal/" but the project no longer uses internal/), **your judgment wins.** Update the rule itself.

- **Granularity tiebreaker:** New top-level package or module → always update docs. Single helper file inside an existing documented area → ignore unless it introduces new responsibility. When in doubt, update — false positives in docs are cheap, missed drift is expensive.

You're a smart colleague doing a review and fixing what you find, not a linter running a checklist.

## ADR Candidates

If you see decisions that look significant or hard-to-reverse, flag them as potential ADR candidates in the report. Do NOT write ADR content yourself — ADRs need explicit human framing of the decision:

- New major dependency (especially with runtime/cross-cutting impact)
- Architectural pattern change
- Migration from one approach to another
- New cross-cutting concern (email, telemetry, auth, queueing)

Threshold: if reverting the choice would require touching 3+ unrelated files OR creates an external dependency the team will have to maintain — ADR candidate. Note "consider ADR for X" in the report.

## What You Don't Do

- Don't edit source code or any non-`.md` file.
- Don't write ADR content — only flag candidates.
- Don't prompt the user for confirmation. You either apply or you don't.
- Don't quietly legitimize tech debt — flag pattern erosion as code drift, don't paper it over.
- Don't verify every line mechanically — use judgment about what matters.
- Don't report formatting differences or trivial mismatches.
- Don't refuse to work if docs have no sync rules — do your best.

## Output

Single section, no preamble. Past tense — what was fixed, what was flagged, what's still in sync.

```
Synced ARCHITECTURE.md:
- §5.10: vm.detach_sg now lists 3 steps (was 2), added assertSGNotTemplate guard.
- §5.2m: removed ErrSGIsTemplate, ErrTemplateRefBroken from securitygroups sentinels; added ErrInvalidIcmpFields.

docs/coding-style.md: in sync. Refactor in internal/api/orders.go aligned with documented IIFE pattern.

Flagged (no doc change made):
- internal/services/notifier.go uses direct DB calls — violates Repository pattern in §3. Code drift, not doc drift.

ADR candidates:
- Email delivery via SendGrid (new outbound dependency)
```

If nothing was wrong:
```
Docs in sync with code (checked N files across M areas).
```

## When Docs Have No Sync Rules

If ARCHITECTURE.md exists but has no explicit sync guidance (wasn't created by arch-init), do your best:

1. Compare documented structures against actual code
2. Fix obvious drift (new directories, changed interfaces, missing components)
3. Add a "Sync rules" stub to ARCHITECTURE.md describing how future syncs should verify alignment

Don't refuse to work — partial docs are better than no review.
