---
technique: "No duplication — pre-write step"
when: "Always — scan for duplicate intent before writing any section. List duplicates and confirm removal before producing output."
---

# Test 18 — Pre-write deduplication

## Technique

> Scan all input content for rules with the same intent, even if worded differently. Keep the most specific version. Remove the abstract restatement. List every duplicate found and confirm removal with the user before writing.
> **When:** Always — applies to CREATE and UPDATE workflows before any section is written.

## Why this test exists

Test 11 verifies the output contains no duplicates. This test verifies the **process** — the skill must catch duplicates before writing and surface them to the user, not silently consolidate or discover them after. The input below contains duplicates with different wording (same intent, not verbatim repeats) to test intent-level detection.

## Input

Paste this as the user prompt when running the agent-doc skill:

```
Write a code review standards doc for my team.

Before reviewing:
- Always check that the test suite passes before approving
- Confirm CI is green before giving approval

During review:
- Never approve code without reading every changed file
- Leave inline comments on non-obvious logic
- Reject any PR where tests are skipped or disabled

After review:
- Add a summary comment with your overall verdict
- Never give approval if any test is missing or bypassed
```

## What to observe

The input contains two duplicate pairs — same intent, different wording:

| Pair | Rule A | Rule B | Intent |
|---|---|---|---|
| 1 | "check that the test suite passes before approving" | "tests are skipped or disabled" + "any test is missing or bypassed" | Tests must pass — appears 3× across 3 sections |
| 2 | "CI is green before giving approval" | Implied by test suite passing | Overlaps with pair 1 |

The skill must:
1. Detect that "test suite passes", "tests are skipped", and "test is missing or bypassed" are the same rule
2. List the duplicates before writing
3. State which version it will keep (the most specific: "Never approve a PR where any test is skipped, disabled, or missing")
4. Confirm before proceeding

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Pre-write message produced | Skill produces a message listing duplicates BEFORE any document output |
| C2 | Intent-level detection | Skill identifies the test-suite rule as duplicated across 3 locations — not just verbatim matches |
| C3 | Kept version stated | Skill identifies which formulation it will keep and why (most specific) |
| C4 | Single rule in output | The test-suite constraint appears exactly once in the written document |
| C5 | No silent consolidation | Skill does not simply write the output without mentioning the duplicates first |

## Fail signals

- Skill writes the document without any pre-write duplication message
- Skill only catches verbatim duplicates — misses "test suite passes" vs "tests are skipped" as the same intent
- Three versions of the test rule appear in the output
- Skill removes two of the three occurrences silently without listing them
