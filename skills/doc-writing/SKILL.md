---
name: doc-writing
description: "Write or update any agent-readable document — instructions, architecture guides, runbooks, API contracts, style guides, or any .md file an agent will read as context. Use when a user needs to create, update, edit, or improve a structured document that an AI agent will consume: frontend architecture doc, deployment runbook, database schema guide, API usage rules, etc."
---

# doc-writing Skill

## Pre-flight

<pre-flight>
  <step number="1">
    <action>Read `references/techniques.md`.</action>
    IF loaded → continue to step 2.
    ELSE → stop and ask: *"references/techniques.md not found — please provide it."*
    <example>File loads and contains format selection rules, language rules, and constraint ordering → continue.</example>
  </step>
  <step number="2">
    <action>Read `references/writing-guide.md`.</action>
    IF loaded → continue to step 3.
    ELSE → stop and ask: *"references/writing-guide.md not found — please provide it."*
    <example>File loads and contains token budget and split rules → continue.</example>
  </step>
  <step number="3">
    <action>Read `references/example-doc.md`.</action>
    IF loaded → proceed to Mode Detection.
    ELSE → stop and ask: *"references/example-doc.md not found — please provide it."*
    <example>File loads and contains a complete example document → proceed to Mode Detection.</example>
  </step>
  <rule>Never proceed to Mode Detection until all three files are confirmed loaded.</rule>
</pre-flight>

## Mode Detection

| Trigger | Action |
|---|---|
| File path or pasted content provided | → **REWRITE workflow** |
| "update section X" / "change only X" / targeted edit | → **UPDATE workflow** |
| No content provided | → **CREATE workflow** |

### REWRITE workflow
Use when the user provides existing content and wants it improved — no interview, no section selection.

1. Load content:
   IF file path provided → read the file.
   ELSE IF content pasted → use directly.
   <example>User pastes 200 lines of AGENTS.md → use directly, skip file read.</example>
2. IF file not found → ask: *"Can you confirm the path or paste the content?"* — stop here.
   <example>Path `pl/AGENTS.md` returns not found → ask before proceeding.</example>
3. Run Deduplication step before writing.
4. Rewrite the full document — apply Writing Rules to every section.
5. Write output:
   IF path was provided → write to same path.
   ELSE → ask: *"Write to [path] or a new file?"* — stop here.
   <example>User gave `docs/runbook.md` → overwrite that file without asking.</example>
6. Run Verification checklist.

### UPDATE workflow
Use when the user targets a specific section — not a full rewrite.

<rule>Never reproduce a section without at least one concrete `<example>`.</rule>

1. Load the file:
   IF file path provided → read it.
   ELSE IF content pasted → use directly.
   ELSE → ask: *"What file should I update?"* — stop here.
   <example>User says "update section 3 of auth.md" → read `auth.md` before showing section summary.</example>
2. IF file not found → ask: *"Confirm the path, or create from scratch?"* — stop here.
   <example>Path `docs/auth.md` returns not found → ask before proceeding.</example>
3. Show a one-line summary per section. Ask which to update.
4. Run Deduplication step on the target section.
5. Run Verification checklist.

### CREATE workflow
Use when no content is provided — start from scratch via interview.

1. Read `references/interview.md` — IF missing: stop and ask *"references/interview.md not found. Please provide it."* Run the discovery interview.
2. Run Deduplication step before writing.
3. Write each section — apply Writing Rules to every section.
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

<rules>
  <rule>Apply `references/techniques.md` for all format and language decisions.</rule>
  <rule>Apply `references/writing-guide.md` for token budget and split decisions.</rule>
</rules>

Before writing each section:

IF items must execute in a specific order → use numbered `<step>` elements.
ELSE IF section routes on keyword triggers from user input → use a 2-column trigger word table.
ELSE IF section routes on conditions the agent evaluates → use IF / ELSE IF branches with numbered steps.
ELSE IF items mix hard and soft rules → hard first, then soft.
ELSE → use `<rule>` elements.
<example>Section lists "never" constraints and style preferences → output hard `<rule>` blocks first, soft `<rule>` blocks after.</example>

## Output

1. Write to the path the user specifies.
2. Include `description:` frontmatter — start with "Load when" or "Use when".
   <example>`description: Load when working in the payments service.` — not `description: This file covers payments.`</example>
3. For any skipped section: add `<!-- section-name: reason -->`.
   <example>`<!-- rollback: not applicable — read-only service -->` — not silently omitting the section.</example>
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