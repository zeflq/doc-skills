---
description: Load before writing any section. Defines token budget, section length limits, split rules, and multi-file system guidance.
---

# Agent Document Writing Guide

## Token Budget

| Level | Max |
|---|---|
| Root agent doc | 80–120 lines |
| Per section | 10–15 lines |
| Total across all linked files | 300 lines |

| Cut | Keep |
|---|---|
| Explanatory prose | Hard constraints |
| Redundant examples | Workflow steps |
| Speculative rules | Verification checklist |
| Empty XML blocks | Permissions model |

## When to Split Into Linked Files

Every linked file must have a `description:` frontmatter field.

Split a section into a linked file when any condition is true:

<conditions>
  <condition>Section exceeds 20 lines.</condition>
  <condition>Section applies to fewer than 30% of tasks.</condition>
  <condition>Section changes at a different rate than the rest of the file.</condition>
</conditions>

| Always in root (loaded always) | Move to linked file (loaded on demand) |
|---|---|
| Hard constraints | Detailed reference tables |
| Permissions and boundaries | Examples library |
| Default workflow | Rollback procedures |
| | Edge-case rules |

## Multi-File Document Systems

<rules>
  <rule>Each doc owns exactly one domain — never let two docs govern the same topic.</rule>
  <rule>Never copy rules across docs — link to the doc that owns them instead.</rule>
  <rule>Every doc must have a `description:` frontmatter field scoped to its own domain.</rule>
  <rule>Create one root index doc when the system has 3+ docs — it describes the system and links to each doc. It owns no domain rules itself.</rule>
</rules>

<example>
  CORRECT cross-reference: "See [auth.md](auth.md) for token rules."
  INCORRECT: copy-pasting the token rules inline from auth.md.
</example>

## Violation-Driven Iteration

Never add rules speculatively — only after a real failure.

<steps>
  <step number="1">Record exactly what the agent did wrong.</step>
  <step number="2">Identify the exact decision point where it failed.</step>
  <step number="3">Add one specific rule targeting that decision point.</step>
  <step number="4">Include the real failure as an `<example>` inside the rule.</step>
</steps>

```xml
<learned-rules>
  <rule date="YYYY-MM-DD" title="[What went wrong]">
    <symptom>[What the agent did and what broke]</symptom>
    <rule>[The specific constraint that prevents recurrence]</rule>
  </rule>
</learned-rules>
```

## Staleness Check

Run on every major update to the system the document describes.

| Question | Action if yes |
|---|---|
| Are any specific file paths hardcoded? | Replace with capability descriptions |
| Are any rules longer than 3 lines? | Must cut to the essential constraint |
| Has any section gone untouched for 3+ months? | Must delete or archive it |
| Is the file longer than 120 lines? | Apply token budget and trim |
| Do any rules contradict each other? | Resolve immediately |
