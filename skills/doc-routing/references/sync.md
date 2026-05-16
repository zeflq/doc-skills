---
description: Load when running the SYNC workflow — diff-based sync from a git range.
---

# SYNC Workflow

<resume-rule>IF the conversation shows a "Proposed sync — N commits:" block awaiting confirmation AND the user's current message is "yes" → skip to step 3 using the range shown. Do not re-run steps 1 or 2.</resume-rule>

## Step 1 — Resolve range

IF user provides "from X to Y" → start = X, end = Y.
ELSE IF user provides "from X" / "since X" → start = X, end = HEAD.
ELSE IF user provides "last N commits" / "from last N" / "previous N" → start = HEAD~N, end = HEAD. Note: `git log HEAD~N..HEAD` returns exactly N commits — start is exclusive.
ELSE IF user says "from the beginning" / "first commit" → start = `git rev-list --max-parents=0 HEAD`, end = HEAD.
ELSE → start = content of `{root}/.last-sync`, end = HEAD.
<example>"sync last 3 commits" → start = HEAD~3, end = HEAD · "from v1.2 to v1.3" → start = v1.2, end = v1.3 · no arg → start = .last-sync hash, end = HEAD.</example>

## Step 2 — Confirm

<rule>Never check uncommitted changes unless the user explicitly asked for them.</rule>
Run `git log -1 --format="%h — %cd — %s" --date=short {start}` → show as "Last sync: {result}".
Run `git log --oneline {start}..{end}` → list commits. IF no commits → output "Notes are up to date." and stop.
Output:
```
Last sync: {hash} — {date} — {message}
Range: {start}..{end}
Proposed sync — {n} commits:
  · {hash} {message}
  · {hash} {message}
Confirm? [yes / provide different range]
```
IF commit count > 20 → append: "⚠️ Large range ({n} commits) — batch into 10–20 commits per sync run."
IF user confirms → continue to step 3. IF user provides different range → restart from step 1.
<example>`.last-sync` = dfbad62 → 2 commits listed → user says yes → continue.</example>

## Step 3 — Group files

Run `git diff {start}..{end} --name-only`. Group changed files by topic (directory prefix or shared module name).
<example>`auth/token.ts` + `auth/refresh.ts` → one `auth` topic.</example>

## Step 4 — Classify topics

IF any file in the topic matches `never_skip` glob → never SKIP, continue to UPDATE/CREATE check.
ELSE IF any file matches `skip_patterns` glob → SKIP.
ELSE IF a doc for this topic exists under `{root}` AND the diff has no impact on its content → SKIP.
ELSE IF a doc for this topic exists under `{root}` → UPDATE.
ELSE → CREATE.
<example>`package-lock.json` matches `**/*.lock` → SKIP · `auth/token.ts` + `back.auth.md` exists + diff impacts it → UPDATE · `proxy/retry.ts` + no doc → CREATE.</example>

## Step 5 — Execute

<rule>Read `references/extraction.md` once before processing any topic.<example>Load extraction guide before the first topic — not once per topic.</example></rule>
<rule>Execute the workflow for every non-SKIP topic before advancing to step 6. Do not summarize, batch, or describe internally. You must have a physically written file for every non-SKIP topic before proceeding.</rule>
IF topic is CREATE:
  0. Derive the file path using SKILL.md `<create>` step 1 rules — prefix must reflect the domain layer (e.g. `back`, `front`, `ext`, `test`), never a generic project name like `repo` or `app`.
  1. Read each source file for this topic. Apply the extraction guide to identify behavioral facts — do not use the diff or git log messages as content.
  2. Execute the doc-writing full workflow inline (including pre-flight) with the extracted facts. The doc must describe what the code currently does — not what changed.
IF topic is UPDATE:
  1. Use the diff to identify which sections are affected. Read those source files and apply the extraction guide to the affected sections only.
  2. Execute the doc-writing full workflow inline (including pre-flight) with the extracted facts for the affected sections only. Skip doc-writing UPDATE step 3 — sections already resolved. The doc must describe current behavior — not what changed.
<example>CREATE `auth` → read `src/auth.ts` → extract behavioral facts → execute doc-writing CREATE inline · UPDATE `auth` → diff shows expiry logic changed → read `src/auth.ts` expiry section → extract behavioral facts → execute doc-writing UPDATE inline.</example>

## Step 6 — Write .last-sync

<rule>Execute the write — do not narrate it. Run `git rev-parse {end}` and write the result to `{root}/.last-sync`. Not complete until the file is written. Never substitute HEAD directly.</rule>
<example>end = HEAD → run `git rev-parse HEAD` → write full hash to `.pi/notes/.last-sync`.</example>

## Step 7 — Write _index.md

<rule>Execute only if at least one file was written in step 5.<example>All topics → SKIP → no files written → skip this step.</example></rule>
Execute `<write-index>`.
