---
skill: "doc-routing"
behavior: "SYNC — correct grouping, classification, and delegation from git changes"
when: "Any SYNC workflow triggered by git range or uncommitted changes"
---

# Test dr-03 — Sync classification and delegation

## Behavior

> The SYNC workflow scans changed files, groups them by topic, classifies each as UPDATE / CREATE / SKIP, then hands each entry to the correct workflow.
> **When:** User triggers SYNC with a git range or after code changes.

## Why this test exists

Agents run SYNC autonomously after commits. The skill must group related files correctly, classify intent accurately (including creating new docs for undocumented patterns), and delegate without mixing workflows.

## Input

Run the doc-routing skill with:

```
sync docs after the last 3 commits
```

Assume the git log contains:
- `auth/token.ts` — modified
- `auth/refresh.ts` — modified
- `proxy/retry.ts` — added (new file, no existing doc)
- `proxy/circuit-breaker.ts` — added (new file, no existing doc)
- `package-lock.json` — modified

## What to observe

Expected grouping and classification:

| Files | Topic | Classification | Reason |
|---|---|---|---|
| auth/token.ts + auth/refresh.ts | `auth` | UPDATE | doc exists, pattern changed |
| proxy/retry.ts + proxy/circuit-breaker.ts | `proxy-resilience` | CREATE | new pattern, no doc exists |
| package-lock.json | — | SKIP | not a behavior change |

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Related files grouped into one topic | auth/token.ts and auth/refresh.ts appear as one `auth` entry — not two |
| C2 | New files trigger CREATE | proxy/retry.ts and proxy/circuit-breaker.ts grouped and classified as CREATE |
| C3 | Lock file skipped | package-lock.json appears under SKIP — never UPDATE or CREATE |
| C4 | UPDATE delegates to UPDATE workflow | auth topic handed to UPDATE workflow — loads existing doc, shows section summary |
| C5 | CREATE delegates to CREATE workflow | proxy-resilience topic handed to CREATE workflow — resolves path, runs doc-writing CREATE |
| C6 | New rules allowed in CREATE | CREATE for a new pattern may add rules not previously documented — skill does not restrict to existing wording only |
| C7 | Correct git command used | Skill runs `git diff --name-only HEAD~{n} HEAD` — never `HEAD~{n}` alone which would include the working tree |
| C8 | Empty diff stops immediately | If `git diff --name-only HEAD~{n} HEAD` returns nothing → skill outputs "nothing to sync" and stops — never hunts for changes manually |

## Fail signals

- auth/token.ts and auth/refresh.ts produce two separate topic entries
- New proxy files classified as UPDATE instead of CREATE
- package-lock.json handed to CREATE or UPDATE
- UPDATE workflow starts writing without loading the existing file first
- CREATE workflow skipped because "it's a sync, not a new doc request"
- Skill restricts new doc content to only wording from the diff
