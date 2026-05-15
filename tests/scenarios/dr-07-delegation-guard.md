---
skill: "doc-routing"
behavior: "Delegation guard — stops immediately if doc-writing skill cannot be loaded"
when: "Any CREATE or UPDATE workflow that requires delegating to doc-writing"
---

# Test dr-07 — Delegation guard

## Behavior

> Before any write step, the skill reads `skills/doc-writing/SKILL.md`. If the file is not found, it stops immediately with a clear error — it never proceeds to write without doc-writing loaded.
> **When:** Any CREATE or UPDATE invocation.

## Why this test exists

Without this guard, a missing doc-writing skill causes the agent to improvise its own writing rules — producing files that violate placement, frontmatter, or structure requirements silently.

## Input

Run doc-routing with doc-writing skill absent (remove or rename `skills/doc-writing/SKILL.md`):

**Prompt A — CREATE without doc-writing:**
```
create a doc about deployment checklist
```

**Prompt B — UPDATE without doc-writing:**
```
update the auth doc, fix the token section
```

## What to observe

Both prompts must stop before writing anything and surface a clear error.

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | CREATE stops before any file is written | no `.md` file created under `{root}` |
| C2 | UPDATE stops before any file is modified | existing doc untouched |
| C3 | Error message names the missing skill | output contains "doc-writing skill not found" or equivalent |
| C4 | Error message tells user what to do | output contains "install it before running doc-routing" or equivalent |
| C5 | No fallback writing attempted | skill does not attempt to write using its own rules or training knowledge |

## Fail signals

- File created under `.pi/notes/` despite doc-writing being absent
- Skill outputs a doc without frontmatter or with wrong path
- Error message says "file not found" without naming the skill or recovery action
- Skill silently continues after a failed read
- Skill offers a menu of options ("1. Create skill 2. Proceed differently") instead of stopping cleanly
