---
description: Canonical format and language rules for agent-readable documents. Read before writing any section.
---

# Agent Document Writing Techniques

## Format Selection

<techniques>
  <technique>
    <name>XML tags</name>
    <when>3+ fields per item, OR agent acts on items one at a time (reads, calls, fetches individually).</when>
    <example>
      CORRECT — 3 fields, agent acts on each tool separately:
        &lt;tool&gt;
          &lt;name&gt;search&lt;/name&gt;
          &lt;trigger&gt;user asks for current info&lt;/trigger&gt;
          &lt;action&gt;call web_search with extracted keywords&lt;/action&gt;
        &lt;/tool&gt;
    </example>
  </technique>
  <technique>
    <name>Tables</name>
    <when>Exactly 2 flat columns. All rows consumed together. No future fields expected. Agent reads whole table — does not act on rows individually.</when>
    <example>
      CORRECT — 2 columns, all rows read together for routing:
        | User said | Action |
        | "create" | → CREATE workflow |
        | "update" | → UPDATE workflow |
    </example>
  </technique>
  <technique>
    <name>Numbered list</name>
    <when>Ordered steps where sequence matters.</when>
  </technique>
  <technique>
    <name>Trigger word table</name>
    <when>Section routes agent behavior based on user input. Synonyms share one row.</when>
    <example>
      | "delete", "remove", "drop" | Run the deletion confirmation step |
    </example>
  </technique>
  <technique>
    <name>If-then logic</name>
    <when>Any decision the agent must make consistently. Write as explicit IF / ELSE IF branches with numbered steps. Never use prose for decisions. Each branch ends with a concrete action — never "use judgment".</when>
    <example>
      IF file exists at path → read it and run UPDATE workflow.
      ELSE IF user pasted content → use directly and run REWRITE workflow.
      ELSE → ask: "What file should I update?" — stop here.
    </example>
  </technique>
  <technique>
    <name>Self-verification loop</name>
    <when>Any workflow that produces output the agent could get wrong silently. Place at the end of the workflow as a checklist of &lt;check&gt; elements.</when>
  </technique>
</techniques>

## Language Rules

Rewrite every soft expression before writing any section. No exceptions.
This applies inside all XML child elements — `<requirement>`, `<action>`, `<rule>`, `<condition>`. No field is exempt.

| Soft (forbidden) | Required replacement |
|---|---|
| prefer not to | never |
| prefer X | always use X |
| try to | always |
| should | must |
| avoid | never |
| ideally | always |

## Constraint Ordering

Within every section: hard rules first, soft preferences after. Never interleave.

Hard = contains "never" / "always", OR breaking it causes security, correctness, or data-integrity failure.
Soft = style, performance, convention.

<example>
  CORRECT order — hard first, soft after:
    &lt;rule&gt;Never return HTTP 200 with an error body.&lt;/rule&gt;
    &lt;rule&gt;Never omit request_id in any response.&lt;/rule&gt;
    &lt;rule&gt;Use camelCase for JSON keys.&lt;/rule&gt;
    &lt;rule&gt;Keep response payloads under 1MB.&lt;/rule&gt;
</example>

## Always Required

- Every agent-readable file must have `description:` frontmatter.
- Every rule or step that could be misapplied must have a concrete `<example>`.
- Every rule lives in exactly one section — never duplicated by intent across files.