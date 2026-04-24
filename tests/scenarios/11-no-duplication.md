---
technique: "No duplication"
when: "Always — user-provided content may contain duplicate rules or repeated blocks. Never preserve duplication from the input."
---

# Test 11 — No duplication

## Technique

> Every rule or content block must live in exactly one place. If user-provided content contains duplicate rules or repeated blocks, consolidate before writing — never preserve duplication from the input.
> **When:** Always — apply when writing or updating any section, regardless of how the content was provided.

## Why this test exists

Users often write the same rule in multiple sections without realising it — once as a hard constraint, once buried in a workflow step. The skill must detect duplicates **before writing** — listing them and confirming removal with the user — then produce output where each rule appears exactly once.

The pre-write step is the primary behavior being tested. The output is the secondary check.

## Input

Paste this as the user prompt when running the agent-doc skill:

```
Write a deployment runbook for my backend service.

Pre-deployment checks:
- Run the full test suite before deploying
- Never deploy on Fridays
- Confirm staging has been validated

Deployment steps:
1. Pull the latest image from the registry
2. Never deploy on Fridays
3. Run database migrations
4. Swap the load balancer to the new instance

Post-deployment checks:
- Monitor error rate for 10 minutes
- Run the full test suite to confirm no regressions
- Roll back immediately if error rate exceeds 1%
```

## What to observe

The input contains two duplicated rules:
- "Never deploy on Fridays" — appears in Pre-deployment checks AND as step 2 of deployment steps
- "Run the full test suite" — appears in Pre-deployment checks AND Post-deployment checks

Expected behavior (in order):
1. Skill lists both duplicates before writing anything
2. Skill states which location it will keep each rule in
3. Skill confirms removal with the user (or proceeds if auto-confirm is acceptable)
4. Written output contains each rule exactly once

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Duplicates listed before writing | Skill identifies both duplicate rules in a pre-write message — not discovered after output is produced |
| C2 | Kept location stated | Skill states which section each rule will be kept in before writing |
| C3 | "Never deploy on Fridays" appears once in output | Rule present in exactly one section — not repeated across pre-deployment and steps |
| C4 | "Run the full test suite" appears once in output | Rule present in exactly one section — not repeated across pre-deployment and post-deployment |
| C5 | No content lost | All distinct rules are present — consolidation removes duplicates only, not unique content |

## Fail signals

- Skill writes the full doc first, then notices duplicates (wrong order — dedup must be pre-write)
- Skill silently removes one occurrence without listing it to the user
- "Never deploy on Fridays" appears in both pre-deployment checks and deployment steps
- "Run the full test suite" appears in both pre-deployment and post-deployment sections
- Any rule appears verbatim in two or more sections
