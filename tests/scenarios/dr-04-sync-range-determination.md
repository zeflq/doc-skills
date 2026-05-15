---
skill: "doc-routing"
behavior: "SYNC step 1 — all range input forms resolve to correct start and end"
when: "Any SYNC workflow where a git range must be resolved"
---

# Test dr-04 — SYNC range determination

## Behavior

> Step 1 resolves `start` and `end` from the user's input or `.last-sync`. Every case must produce a concrete pair — never guess or default to HEAD~1.
> **When:** Any SYNC invocation, regardless of how the range is expressed.

## Why this test exists

Five distinct range inputs map to five different resolution paths. Conflating them causes syncs that silently miss commits or re-sync already-covered work.

## Input

Each prompt is independent. Run once per prompt.

**Prompt A — version tags, both bounds:**
```
sync from v1.2 to v1.3
```

**Prompt B — commit hash, from-only:**
```
sync since abc1234
```

**Prompt C — last N (alternate phrasing):**
```
sync from last 4 commits
```

**Prompt D — no .last-sync (blank project):**
```
sync notes
```
*No `.pi/notes/.last-sync` exists.*

**Prompt E — branch name (invalid):**
```
sync notes from main to feature/x
```

## What to observe

| Prompt | Expected start | Expected end | On invalid |
|---|---|---|---|
| A | `v1.2` | `v1.3` | — |
| B | `abc1234` | `HEAD` | — |
| C | `HEAD~4` | `HEAD` | — |
| D | — | — | asks: "No last-sync found. Provide a starting commit or range." |
| E | — | — | asks user to provide a commit hash or tag instead |

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Prompt A uses both bounds | `end = v1.3` — never defaults to HEAD |
| C2 | Prompt B defaults end to HEAD | `start = abc1234`, `end = HEAD` |
| C3 | Prompt C resolves HEAD~N | `start = HEAD~4` — "from last N" and "last N commits" both match |
| C4 | Prompt D stops and asks | output contains "No last-sync found. Provide a starting commit or range." and does not proceed to step 2 — extra context after the message is acceptable |
| C5 | Prompt E rejects branch names | asks for a hash or tag — never passes `main` to a git command |
| C6 | No silent fallback to HEAD~1 | ambiguous input never defaults to HEAD~1 |

## Fail signals

- Prompt A ignores `to v1.3` and syncs to HEAD
- Prompt C runs `git diff --name-only HEAD~4` (one ref) instead of `HEAD~4 HEAD`
- Prompt D shows a commit list without asking for a range
- Prompt D proceeds to step 2 (shows a commit list) without the stop message
- Prompt D never outputs the stop message at all
- Prompt E resolves `main` via `git rev-parse` and proceeds silently
