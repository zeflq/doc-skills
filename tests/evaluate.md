---
description: Evaluator prompt for testing agent-doc technique application — load with a scenario file and generated document output to score.
---

# agent-doc Technique Evaluator

## How to use

1. Load the scenario file: `tests/scenarios/NN-technique-name.md`
2. Run the agent-doc skill with the scenario's **Input** section as the user prompt
3. Paste the generated document here as **Output**
4. Run this evaluator prompt to get a pass/fail score per technique check

---

## Evaluation prompt

```
You are a strict evaluator for the agent-doc skill.

You will be given:
- TECHNIQUE: the technique being tested (name + when condition)
- INPUT: what was given to the agent-doc skill
- OUTPUT: the document the skill produced
- CHECKS: the specific assertions to verify

For each CHECK, answer:
  PASS — the output satisfies the check
  FAIL — the output does not satisfy the check, explain why in one line
  N/A  — the check does not apply to this output

Then give an overall verdict: PASS (all checks pass) or FAIL (any check fails).

Be strict. Partial compliance counts as FAIL.
```

---

## Score table (fill in after each run)

| # | Technique | Status | Notes |
|---|-----------|--------|-------|
| 01 | XML tags for structure | — | |
| 02 | Tables for mappings | — | |
| 03 | Imperative language | — | |
| 04 | Constraints before guidelines | — | |
| 05 | Strict sequential workflow | — | |
| 06 | Self-verification loop | — | |
| 07 | Examples not explanations | — | |
| 08 | description frontmatter | — | |
| 09 | Trigger word tables | — | |
| 10 | If-then logic | — | |
| 11 | No duplication — output + pre-write step | — | C1-C2 test pre-write listing; C3-C5 test output |
| 12 | Constraints before guidelines — any section name | — | |
| 13 | Strict sequential workflow — any section name | — | |
| 14 | Constraints before guidelines — any section name v2 | — | |
| 15 | Strict sequential workflow — any section name v2 | — | |
| 16 | Multi-file document system | — | |
| 17 | Token budget | — | |
| 18 | Pre-write deduplication — intent-level | — | Tests same-intent detection before writing, not just verbatim |
