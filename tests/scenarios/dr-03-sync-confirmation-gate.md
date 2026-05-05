---
skill: "doc-routing"
behavior: "Sync confirmation gate — plan shown and execution blocked until user confirms"
when: "Any SYNC workflow triggered by git range or uncommitted changes"
---

# Test dr-03 — Sync confirmation gate

## Behavior

> The SYNC workflow must show a full UPDATE / CREATE / SKIP plan and stop. No file is written until the user explicitly confirms.
> **When:** Any SYNC workflow — regardless of how many topics are in scope.

## Why this test exists

Silent writes during sync are hard to reverse and can overwrite docs the user didn't intend to touch. The confirmation gate is a hard rule — skipping it even once (e.g. "only one file changed, seems safe") is a failure.

## Input

Run the doc-routing skill with this prompt:

```
sync docs for the last 3 commits
```

Assume the git log contains:
- `auth/token.ts` — modified
- `auth/refresh.ts` — modified
- `proxy/retry.ts` — modified (no existing doc)
- `package-lock.json` — modified

## What to observe

Expected grouping and classification:

| Files | Topic | Classification |
|---|---|---|
| auth/token.ts + auth/refresh.ts | `auth` | UPDATE (doc exists) |
| proxy/retry.ts | `proxy-retry` | CREATE (no doc exists) |
| package-lock.json | — | SKIP |

The skill must output this plan and wait. No doc-writing call may happen before the user replies.

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Files grouped by topic | `auth/token.ts` and `auth/refresh.ts` appear as one `auth` topic — not two separate entries |
| C2 | Correct classifications | auth → UPDATE, proxy-retry → CREATE, package-lock.json → SKIP |
| C3 | Plan shown before any write | Skill outputs the full UPDATES / CREATES / SKIPPED list before calling doc-writing |
| C4 | Execution blocked on confirmation | Skill ends its turn after showing the plan — does not proceed until user replies |
| C5 | SKIP applied to lock file | `package-lock.json` appears under SKIPPED — not classified as UPDATE or CREATE |

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Files grouped by topic | `auth/token.ts` and `auth/refresh.ts` appear as one `auth` topic — not two separate entries |
| C2 | Correct classifications | auth → UPDATE, proxy-retry → CREATE, package-lock.json → SKIP |
| C3 | Plan shown before any write | Skill outputs the full UPDATES / CREATES / SKIPPED list before calling doc-writing |
| C4 | Turn ends after plan | Skill ends its turn immediately after the plan — no edit tool called in the same turn |
| C5 | SKIP applied to lock file | `package-lock.json` appears under SKIPPED — not classified as UPDATE or CREATE |
| C6 | No new rules added from sync | SYNC never adds new content that wasn't referenced in the changed files — only updates existing wording |

## Fail signals

- Skill writes any file before showing the plan
- Skill calls an edit tool in the same turn as the plan output
- Files not grouped — each changed file becomes its own topic entry
- `package-lock.json` classified as UPDATE or CREATE
- Skill adds a new rule or section not referenced in the changed files
- Plan shown but execution continues in the same turn without waiting for a reply
