---
name: doc-routing
description: Use when placing, updating, or syncing agent-readable .md files in a project; loads placement rules, plans doc syncs, then delegates writing to doc-writing.
---

# doc-routing Skill

<context-routing>
| User intent | Workflow |
|---|---|
| "add", "create", "new doc", "new note", "new file" | CREATE workflow |
| "update", "edit", "fix", "improve", "clarify" | UPDATE workflow |
| "sync", "docs after code", "git changes", "commit range" | SYNC workflow |
</context-routing>

<config>
  root: .pi/notes/
  subdirs: [front, back]
  pattern: "{subdir}.{topic}.md"
  <!-- To change these values, edit this block and update README.md -->
</config>

<core-rules>
  <rule><requirement>Never proceed until placement rules are confirmed.</requirement><example>Stop before writing if the root path or naming pattern is still unknown.</example></rule>
  <rule><requirement>Every file must have YAML frontmatter with `description:` starting with "Load when" or "Use when".</requirement><example>Use `description: Load when working in the payments docs.`</example></rule>
  <rule><requirement>Keep one topic per file.</requirement><example>Split auth and billing into separate files instead of merging them.</example></rule>
  <rule><requirement>Use concise inline examples.</requirement><example>Link to a longer example file instead of pasting a full 300-line sample.</example></rule>
</core-rules>

<delegation>
  <rule><requirement>Read `skills/doc-writing/SKILL.md` with `read`, then execute its workflow inline.</requirement><example>Load doc-writing, follow its pre-flight, and run CREATE or UPDATE yourself.</example></rule>
  <rule><requirement>Apply project placement rules to every linked file doc-writing creates.</requirement><example>A split-off linked file must still follow `{subdir}.{topic}.md`.</example></rule>
</delegation>

<workflows>
  <create>
    <step number="1">
      Derive subdir from user intent by matching topic against config subdirs.
      IF topic matches a config subdir → apply pattern `{subdir}.{topic}.md` under `{root}/{subdir}/`.
      ELSE → derive subdir from the topic's domain, create it, and apply pattern.
      <example>Topic `proxy errors` + "back" matched → `.pi/notes/back/back.proxy-errors.md` · Topic `deployment checklist` + no match → derive `deployment` → `.pi/notes/deployment/deployment.deployment-checklist.md`</example>
    </step>
    <step number="2"><action>Run doc-writing CREATE with the resolved path and project constraints.</action><example>Create the new file, then apply the same placement rules to any linked file.</example></step>
    <step number="3"><action>Run the validation checklist before committing.</action><example>Check frontmatter, topic scope, and path naming before finish.</example></step>
  </create>

  <update>
    <step number="1"><action>Load the existing file using config path rules.</action><example>Read `.pi/notes/back/back.auth.md` before changing the auth section.</example></step>
    <step number="2"><action>Show a one-line summary per section.</action><example>Summaries: error-mapping, handling-rules, logging, tests — user picks handling-rules.</example></step>
    <step number="3">
      IF requested section exists → proceed with doc-writing UPDATE on that section only.
      ELSE IF human context → ask: "No [section] found. Add it as a new section, or update an existing one? ([closest existing section] covers related content.)"
      ELSE → add it as a new section automatically and note it in the output.
      <example>Agent runs UPDATE, retry section missing → adds a new retry section and outputs: "No retry section found — added as new section."</example>
    </step>
    <step number="4"><action>Run doc-writing UPDATE on the confirmed section only.</action><example>Edit handling-rules without touching error-mapping or logging.</example></step>
    <step number="5"><action>Run the validation checklist before committing.</action><example>Verify only the targeted topic changed.</example></step>
  </update>

  <sync>
    <step number="1">
      Pick git range from argument or default to `HEAD~1`.
      Run: `git diff --name-only HEAD~{n} HEAD` to list files changed in those commits only.
      IF output is empty → output "No changed files in last {n} commits — nothing to sync." and stop.
      <example>No args → `git diff --name-only HEAD~1 HEAD` · "3" → `git diff --name-only HEAD~3 HEAD` · empty result → stop.</example>
    </step>
    <step number="2">
      Group changed files by topic (directory prefix or shared module name).
      IF user explicitly asks to include uncommitted changes → also run `git diff --name-only` (unstaged) and `git diff --name-only --cached` (staged).
      <example>`auth/token.ts` + `auth/refresh.ts` → one `auth` topic · do not mix in staged/unstaged files unless requested.</example>
    </step>
    <step number="3"><action>Classify each topic as UPDATE, CREATE, or SKIP.</action><example>`package-lock.json` → SKIP · changed auth doc → UPDATE · new undocumented pattern → CREATE.</example></step>
    <step number="4">
      Hand each entry to the CREATE or UPDATE workflow.
      For UPDATE: use the diff to identify which existing sections are affected, then update those sections in place.
      <example>Workflow key renamed → read the diff, locate the setup section, update it in place with the new key name.</example>
    </step>
  </sync>
</workflows>

<validation-checklist>
  <check>Paths resolved from hardcoded config before any file was written.<example>Topic and subdir mapped to a full path before calling doc-writing.</example></check>
  <check>Every file has frontmatter with a `description:` that starts with "Load when" or "Use when".</check>
  <check>Each file covers exactly one topic.<example>Auth and billing never share one doc.</example></check>
  <check>Sync plans are shown before any write starts.<example>UPDATES / CREATES / SKIPPED appear before execution.</example></check>
  <check>doc-writing was followed inline and not delegated away.<example>Read the skill, then run its workflow directly.</example></check>
</validation-checklist>

<self-verification>
  <check>The skill now covers placement, updates, and git-based sync planning.</check>
  <check>No workflow writes before placement is confirmed.</check>
  <check>Every workflow step has a concrete example.</check>
  <check>The sync workflow stops for confirmation before execution.</check>
</self-verification>
