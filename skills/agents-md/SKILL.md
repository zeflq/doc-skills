---
name: agents-md
description: "Guided creation and update of AGENTS.md — the unified project instruction file recognized by Pi (AGENTS.md) and Claude Code (CLAUDE.md or AGENTS.md). Use when a user asks to create, generate, write, or update an AGENTS.md or CLAUDE.md file for their project. Covers: project scope, permissions, hard constraints, coding standards, tooling, workflow, verification, output expectations, safety, and agent roles."
---

# agents-md Skill

This skill produces a **fixed 10-section AGENTS.md** — an opinionated best-practice structure. Sections 1–7 are required, 8–10 are recommended. It does not discover sections from the project; use `agent-doc` for freeform agent documents.

Use the `agent-doc` skill to write every section — it owns the techniques, writing guide, and verification rules.
Read `references/example-doc.md` when writing any section — use it as the AGENTS.md-specific format reference.

## Mode Detection

| Trigger | Action |
|---|---|
| File path or pasted content provided | → **REWRITE workflow** |
| "update section X" / "change only X" / targeted edit | → **UPDATE workflow** |
| No content provided | → **CREATE workflow** |

### REWRITE workflow
Use when the user provides an existing AGENTS.md and wants it fully improved — no interview, no section selection.

1. Load content: file path → read it; pasted → use directly.
2. File not found → ask: *"Can you confirm the path or paste the content?"* — stop here.
3. Run Deduplication step before writing.
4. Map content into the 10-section structure using the `agent-doc` skill to apply techniques to every section.
5. Write to: same path if provided · ask *"Write to [path] or a new file?"* if unclear.
6. Run Verification checklist.

### UPDATE workflow
Use when the user targets a specific section — not a full rewrite.

1. Load the file: path provided → read it; pasted → use directly; neither → read `AGENTS.md` or `CLAUDE.md` from project root.
2. File not found → ask: *"Confirm the path, or create from scratch?"* — stop here.
3. Show a one-line summary per section. Ask which to update.
4. Run Deduplication step on the target section.
5. Never reproduce a section without at least one concrete `<example>`.
6. Run Verification checklist.

### CREATE workflow
Use when no content is provided — start from scratch via interview.

1. Read `references/sections.md`. Walk sections 1–7 (required), then 8–10 (recommended), interviewing per that file.
2. Run Deduplication step before writing.
3. Write each section using the `agent-doc` skill to apply techniques.
4. Run Verification checklist.

## Interview Rules

- Never bundle multiple sections in one message.
- Never leave soft language in any rule — rewrite before writing.
- If the user says **"skip"**: include the section as an XML comment block.

## Deduplication

1. Scan all input content for rules with the same intent, even if worded differently.
2. Keep the most specific version. Remove the abstract restatement.
3. List every duplicate found and confirm removal with the user before writing.

<example>
  DUPLICATE — different wording, same intent:
    section A: "Always validate the token before processing."
    section B: "Return 401 when the token is missing or invalid."
  Keep section B (specific enforcement). Remove section A (abstract restatement).
</example>

## Writing Rules

Use the `agent-doc` skill for all format, language, and token budget decisions.

Before writing each section, answer:

| Question | Answer → Format |
|---|---|
| Must items execute in a specific order? | Yes → numbered `<step>` · No → `<rule>` |
| Does section route on *keyword triggers* from user input (synonyms map to one action)? | Yes → 2-column trigger word table |
| Does section route on *conditions* the agent evaluates (task type, file exists, scope)? | Yes → IF / ELSE IF branches with numbered steps |
| Do items mix hard and soft? | Yes → hard first, then soft · No → any order |

## Section Order

| # | Section | Status |
|---|---|---|
| 1 | Project Scope | Required |
| 2 | Permissions & Boundaries | Required |
| 3 | Hard Constraints | Required |
| 4 | Coding Standards | Required |
| 5 | Tooling & Commands | Required |
| 6 | Default Workflow | Required |
| 7 | Verification Checklist | Required |
| 8 | Output Expectations | Recommended |
| 9 | Safety Rules | Recommended |
| 10 | Agent Roles | Optional |

## Output

1. Write the completed file to `./AGENTS.md`.
2. Include `description:` frontmatter — start with "Load when" or "Use when".
3. Confirm: *"AGENTS.md written — Sections filled: [list]. Skipped: [list]."*

## Verification

Rewrite any failure before confirming output.

<self-verification>
  <check>description: frontmatter is present and starts with "Load when" or "Use when".</check>
  <check>Sections 1–7 are all present or have an XML comment explaining why each was skipped.</check>
  <check>Hard rules appear before soft in every mixed section.</check>
  <check>No rule or content block is duplicated by intent anywhere in the document.</check>
  <check>Every section that routes on user input uses a trigger word table.</check>
  <check>Every workflow document ends with a self-verification checklist.</check>
  <check>Every rule and step has a concrete example.</check>
  <check>No section exceeds 10–15 lines — enforced by agent-doc writing guide.</check>
</self-verification>