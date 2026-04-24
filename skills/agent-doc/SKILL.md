---
name: agent-doc
description: "Write or update any agent-readable document — instructions, architecture guides, runbooks, API contracts, style guides, or any .md file an agent will read as context. Use when a user needs to create, update, edit, or improve a structured document that an AI agent will consume: frontend architecture doc, deployment runbook, database schema guide, API usage rules, etc."
---

# agent-doc Skill

Read `references/techniques.md` before writing any section.
Read `references/writing-guide.md` before writing any section.
Read `references/example-doc.md` when writing any section — use it as the format reference.

## Mode Detection

| Trigger | Action |
|---|---|
| File path or pasted content provided | → **REWRITE workflow** |
| "update section X" / "change only X" / targeted edit | → **UPDATE workflow** |
| No content provided | → **CREATE workflow** |

### REWRITE workflow
Use when the user provides existing content and wants it improved — no interview, no section selection.

1. Load content: file path → read it; pasted → use directly.
2. File not found → ask: *"Can you confirm the path or paste the content?"* — stop here.
3. Run Deduplication step before writing.
4. Rewrite the full document applying `references/techniques.md`.
5. Write to: same path if provided · ask *"Write to [path] or a new file?"* if unclear.
6. Run Verification checklist.

### UPDATE workflow
Use when the user targets a specific section — not a full rewrite.

1. Load the file: path provided → read it; pasted → use directly; neither → ask: *"What file should I update?"*
2. File not found → ask: *"Confirm the path, or create from scratch?"* — stop here.
3. Show a one-line summary per section. Ask which to update.
4. Run Deduplication step on the target section.
5. Never reproduce a section without at least one concrete `<example>`.
6. Run Verification checklist.

### CREATE workflow
Use when no content is provided — start from scratch via interview.

1. Read `references/interview.md`. Run the discovery interview.
2. Run Deduplication step before writing.
3. Write each section applying `references/techniques.md`.
4. Run Verification checklist.

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

Apply `references/techniques.md` for all format and language decisions.
Apply `references/writing-guide.md` for token budget and split decisions.

Before writing each section, answer:

| Question | Answer → Format |
|---|---|
| Must items execute in a specific order? | Yes → numbered `<step>` · No → `<rule>` |
| Does section route on *keyword triggers* from user input (synonyms map to one action)? | Yes → 2-column trigger word table |
| Does section route on *conditions* the agent evaluates (task type, file exists, scope)? | Yes → IF / ELSE IF branches with numbered steps |
| Do items mix hard and soft? | Yes → hard first, then soft · No → any order |

## Output

1. Write to the path the user specifies.
2. Include `description:` frontmatter — start with "Load when" or "Use when".
3. For any skipped section: add `<!-- section-name: reason -->`.
4. Confirm: *"[filename] written — Sections: [list]."*

## Verification

Rewrite any failure before confirming output.

<self-verification>
  <!-- These checks verify the SKILL output quality — not task execution. -->
  <!-- For the correct self-verification content to put IN the produced doc, see references/example-doc.md. -->
  <!-- Self-verification checks in the produced doc must cover task execution, not document structure. -->
  <check>description: frontmatter is present and starts with "Load when" or "Use when".</check>
  <check>Hard rules appear before soft in every mixed section.</check>
  <check>No rule or content block is duplicated by intent anywhere in the document.</check>
  <check>Every section that routes on keyword triggers uses a trigger word table.</check>
  <check>Every section that routes on evaluated conditions uses IF / ELSE IF branches.</check>
  <check>Every rule and step has a concrete example inside an XML element — not prose.</check>
  <check>No section exceeds 10–15 lines — see references/writing-guide.md.</check>
  <check>The produced doc's self-verification checks cover task execution — not document structure. See references/example-doc.md for correct examples.</check>
</self-verification>