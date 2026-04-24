---
description: Load when discovering sections for a new agent-readable document. Ordered interview workflow for scope, sections, and technique assignment.
---

# Discovery Interview — agent-doc

<workflow>
  <step number="1" name="Ask discovery questions">
    <action>Ask all four questions in one message.</action>
    <action>Capture answers as draft inputs for scope, audience, decisions, and load trigger.</action>
    <example>
      1. What is this document for?
      2. Who reads it, and in what context?
      3. What decisions should the reader make differently after reading it?
      4. In one sentence, when should an agent load this file?
    </example>
  </step>

  <step number="2" name="Decide if scope is sufficient">
    <if>IF scope, audience, and decision intent are clear, continue to Step 3.</if>
    <else>ELSE ask only the minimum follow-up questions required to remove ambiguity, then continue to Step 3.</else>
    <rule>Never ask for information already provided by the user.</rule>
  </step>

  <step number="3" name="Propose section structure">
    <action>Propose sections and assign one writing technique per section from references/techniques.md.</action>
    <action>To assign a technique: match the section's content type against the `<when>` condition of each technique. The first technique whose `<when>` matches is the one to use.</action>
    <action>Ask the user to confirm, add, or remove sections before drafting content.</action>
    <example>"Proposed sections: steps (strict sequential workflow — ordered actions), rules (XML tags — 3+ fields per item), verification (self-verification loop — output the agent could get wrong). Keep or edit?"</example>
  </step>

  <step number="4" name="Interview section by section">
    <action>Handle one confirmed section at a time.</action>
    <action>Show one filled example in the selected technique format, then ask the user to adapt or replace it.</action>
    <rule>Never draft a later section before the current section is confirmed.</rule>
  </step>
</workflow>

<section-template-map>

| Document type | Example sections (XML tag names) |
|---|---|
| Architecture / tech guide | `<overview>`, `<stack>`, `<patterns>`, `<anti-patterns>`, `<verification>` |
| Runbook / workflow | `<prerequisites>`, `<steps>`, `<verification>`, `<rollback>` |
| API contract | `<authentication>`, `<endpoints>`, `<error-handling>`, `<rate-limits>` |
| Style / coding guide | `<naming>`, `<file-structure>`, `<patterns>`, `<forbidden>` |
| Schema / data guide | `<entities>`, `<allowed-operations>`, `<query-patterns>`, `<off-limits>` |
| Deployment guide | `<environments>`, `<steps>`, `<verification>`, `<rollback>` |
| Generic rules doc | `<allowed-scope>`, `<rules>`, `<exceptions>`, `<verification>` |

</section-template-map>

<self-verification>
  <check>All four discovery questions were asked before section drafting.</check>
  <check>Each proposed section has exactly one selected technique.</check>
  <check>Every drafted section includes a concrete example.</check>
  <check>Hard constraints appear before soft guidelines in mixed sections.</check>
</self-verification>
