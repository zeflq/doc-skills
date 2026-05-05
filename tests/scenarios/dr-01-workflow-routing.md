---
skill: "doc-routing"
behavior: "Workflow routing — trigger words map to the correct workflow"
when: "Any user request that contains a trigger word from the context-routing table"
---

# Test dr-01 — Workflow routing

## Behavior

> The skill reads the user's intent and routes to CREATE, UPDATE, or SYNC.
> **When:** User request contains a trigger word from the context-routing table. Synonyms on the same row all route to the same workflow.

## Why this test exists

Users phrase the same intent in many different ways ("add", "new", "create" all mean CREATE). The skill must route consistently — not guess from context, not ask for clarification when a trigger word is present.

## Input

Run the doc-routing skill three times, once per prompt:

**Prompt A — CREATE trigger:**
```
add a doc about proxy error handling in the back domain
```

**Prompt B — UPDATE trigger:**

> Prerequisite: run Prompt A first — `.pi/notes/back/back.proxy-error-handling.md` must exist.

```
fix the proxy error handling doc, the retry section is outdated
```

**Prompt C — SYNC trigger:**
```
sync docs after the last 2 commits
```

## What to observe

| Prompt | Trigger word | Expected workflow |
|---|---|---|
| A | "add" | CREATE — produces `.pi/notes/back/back.proxy-error-handling.md` |
| B | "fix" | UPDATE — loads the file Prompt A created |
| C | "sync" | SYNC |

The skill must not ask "which workflow do you want?" when a trigger word is present.

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Prompt A routes to CREATE | Skill starts the CREATE workflow — resolves a path, calls doc-writing CREATE |
| C2 | Prompt B routes to UPDATE | Skill starts the UPDATE workflow — loads the existing file, shows section summary |
| C3 | Prompt C routes to SYNC | Skill starts the SYNC workflow — picks git range, groups by topic |
| C4 | No clarification question when trigger word is present | Skill never asks "which workflow?" when a routing word is in the prompt |
| C5 | Wrong workflow never started | Prompt A never starts UPDATE or SYNC; Prompt B never starts CREATE or SYNC |
| C6 | Missing section handled without stalling | Human context: skill asks "add it or update closest?". Agent context: skill adds the section automatically and notes it in output — never silently fails or skips |

## Fail signals

- Skill asks "do you want to create or update?" when trigger word is unambiguous
- "new" or "note" fails to trigger CREATE
- "improve" or "clarify" fails to trigger UPDATE
- "git changes" or "commit range" fails to trigger SYNC
- Skill picks a workflow based on the topic noun instead of the trigger verb
