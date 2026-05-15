---
name: doc-routing
description: "Use when placing, updating, or syncing agent-readable .md files in a project. Handles CREATE (new doc from topic), UPDATE (edit a section), and SYNC (update notes from a git range, after a sprint, after a PR, or after X commits). Triggers on phrases like: sync notes, update the notes, sync my docs, create a note, add a doc, update the doc, new note. Loads placement rules, plans doc syncs, then delegates writing to doc-writing."
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
  subdirs: []
  pattern: "{subdir}.{topic}.md"
  skip_patterns: ["**/*.lock", "**/dist/**", "**/*.generated.*", "**/build/**", ".pi/skills/**"]
  never_skip: []
  <!-- To change these values, edit this block and update README.md -->
</config>

<core-rules>
  <rule><requirement>Never proceed until placement rules are confirmed.</requirement><example>Stop before writing if the root path or naming pattern is still unknown.</example></rule>
  <rule><requirement>Every file must have YAML frontmatter with `description:` starting with "Load when" or "Use when".</requirement><example>Use `description: Load when working in the payments docs.`</example></rule>
  <rule><requirement>Keep one topic per file.</requirement><example>Split auth and billing into separate files instead of merging them.</example></rule>
  <rule><requirement>Always use concise inline examples.</requirement><example>Link to a longer example file instead of pasting a full 300-line sample.</example></rule>
</core-rules>

<delegation>
  <rule>
    <requirement>Read `skills/doc-writing/SKILL.md` with `read` before any write step.</requirement>
    IF loaded → execute its workflow inline.
    ELSE → stop and output: "doc-writing skill not found — install it before running doc-routing." Do not offer alternatives or options.
    <example>File not found at `skills/doc-writing/SKILL.md` → output the message and stop. Never offer to scaffold, create, or work around the missing skill.</example>
  </rule>
  <rule><requirement>Apply project placement rules to every linked file doc-writing creates.</requirement><example>A split-off linked file must still follow `{subdir}.{topic}.md`.</example></rule>
</delegation>

<workflows>
  <create>
    <step number="1">
      Derive subdir from user intent by matching topic against config subdirs.
      IF subdirs is non-empty AND topic matches a config subdir → place under `{root}/{subdir}/{subdir}.{topic}.md`.
      ELSE → derive a domain prefix from the topic and place flat in `{root}/{prefix}.{topic}.md`. Note: `subdirs: []` means flat layout — `{subdir}` is a filename prefix only, no directory is created.
      <example>subdirs: ["back"] + topic `proxy errors` → `.pi/notes/back/back.proxy-errors.md` · subdirs: [] + topic `review` → derive prefix `back` → `.pi/notes/back.review.md` · subdirs: [] + topic `workflow-pi-review` → derive prefix `github` → `.pi/notes/github.workflow-pi-review.md`</example>
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
      <rule>Before running any git command: IF the user provided no explicit range AND `{root}/.last-sync` does not exist → output exactly: "No last-sync found. Provide a starting commit or range." and terminate. Do not run any git commands. Do not continue.</rule>
      <rule>Before running any git command: IF any bound provided by the user is a branch name (not a commit hash and not a version tag matching vX.Y format) → output exactly: "Ranges must use commit hashes or version tags (e.g. v1.2). Provide a hash or tag instead." and terminate. Do not run any git commands to check if the branch exists. Do not continue.</rule>
      Determine range start and end.
      IF user provides "from X to Y" → start = X, end = Y.
      ELSE IF user provides "from X" / "since X" → start = X, end = HEAD.
      ELSE IF user provides "last N commits" / "from last N" / "previous N" → start = HEAD~N, end = HEAD. Note: `git log HEAD~N..HEAD` returns exactly N commits — start is exclusive.
      ELSE IF user says "from the beginning" / "first commit" → start = `git rev-list --max-parents=0 HEAD`, end = HEAD.
      ELSE → start = content of `{root}/.last-sync`, end = HEAD.
      <example>"sync last 3 commits" → start = HEAD~3, end = HEAD · "from v1.2 to v1.3" → start = v1.2, end = v1.3 · no arg + `.last-sync` = dfbad62 → start = dfbad62, end = HEAD.</example>
    </step>
    <step number="2">
      <rule>Never check uncommitted changes unless the user explicitly asked for them in the original request.</rule>
      <step number="2a"><action>Run `git log -1 --format="%h — %cd — %s" --date=short {start}` → show as "Last sync: {result}" (or "Never synced" if no `.last-sync`).</action></step>
      <step number="2b"><action>Run `git log --oneline {start}..{end}` → list commits in scope. IF no commits → output "Notes are up to date." and stop.</action></step>
      <step number="2c">
        Output:
        ```
        Last sync: {hash} — {date} — {message}
        Proposed sync — {n} commits:
          · {hash} {message}
          · {hash} {message}
        Confirm? [yes / provide different range]
        ```
        IF user confirms → continue to step 3.
        IF user provides different range → restart from step 1 with new range.
      </step>
      <example>`.last-sync` = dfbad62 → "Last sync: dfbad62 — 2026-05-10 — fix(pi-api-key)" · 2 commits listed → user says yes → continue.</example>
    </step>
    <step number="3">
      Group changed files by topic (directory prefix or shared module name).
      <example>`auth/token.ts` + `auth/refresh.ts` → one `auth` topic · do not mix in staged/unstaged files unless requested.</example>
    </step>
    <step number="4">
      Classify each topic:
      IF any file in the topic matches a config `never_skip` glob → never SKIP, continue to UPDATE/CREATE check.
      ELSE IF any file in the topic matches a config `skip_patterns` glob → SKIP.
      ELSE IF a doc for this topic exists under `{root}` AND the diff has no impact on that doc's content → SKIP.
      ELSE IF a doc for this topic exists under `{root}` → UPDATE.
      ELSE → CREATE.
      <example>`package-lock.json` matches `**/*.lock` → SKIP · `auth/token.ts` changed + `.pi/notes/back/back.auth.md` exists + diff affects auth logic → UPDATE · `proxy/retry.ts` added + no doc under `.pi/notes/` → CREATE.</example>
    </step>
    <step number="5">
      Hand each entry to the CREATE or UPDATE workflow.
      For UPDATE: use the diff to identify which existing sections are affected, then update those sections in place.
      <example>Workflow key renamed → read the diff, locate the setup section, update it in place with the new key name.</example>
    </step>
    <step number="6">
      <rule>Execute the write — do not narrate it. Run `git rev-parse {end}` and write the output to `{root}/.last-sync`. This step is not complete until the file has been written. Always use the resolved `{end}` value — never substitute `HEAD` directly.</rule>
      <example>Range was HEAD~2..HEAD → run `git rev-parse HEAD` (end=HEAD here), write full hash to `.pi/notes/.last-sync` · Range was dfbad62..abc1234 → run `git rev-parse abc1234`, write that hash.</example>
    </step>
  </sync>
</workflows>

<validation-checklist>
  <check>Every file has frontmatter with a `description:` that starts with "Load when" or "Use when".</check>
  <check>Each file covers exactly one topic.<example>Auth and billing never share one doc.</example></check>
  <check>doc-writing was followed inline and not delegated away.<example>Read the skill, then run its workflow directly.</example></check>
</validation-checklist>

<self-verification>
  <check>doc-writing SKILL.md was read before any write step was executed.</check>
  <check>Full file path was resolved from config before calling doc-writing.</check>
  <check>SYNC showed the proposed commit list and received confirmation before any file was written.</check>
  <check>Every topic was classified (UPDATE / CREATE / SKIP) before any workflow was handed off.</check>
  <check>`{root}/.last-sync` was physically written (file write executed, not just narrated) with the resolved `{end}` hash after SYNC completed.</check>
</self-verification>
