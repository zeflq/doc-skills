---
description: Load when running the COLD-START workflow — first-ever sync on a blank project with no .last-sync and no existing docs.
---

# COLD-START Workflow

<resume-rule>IF the conversation shows a "Proceed? [yes / cancel]" cold-start block AND the user's current message is "yes" → run `git ls-files` immediately as your first tool call. Do not narrate a plan before executing. Proceed with step 2.<example>User said "yes" after cold-start block → run `git ls-files` now, not step 1 confirmation.</example></resume-rule>

## Step 1 — Confirm scope

Run `git ls-files | wc -l` to estimate file count.
Output exactly:
```
Cold start detected — no previous sync found.
Root: {repo root}
Skipping: {skip_patterns}
Files in scope: ~N files (will be grouped into topics — typical repos produce 5–20 docs)
Proceed? [yes / cancel]
```
IF user confirms → proceed to step 2. Do not ask about limiting scope — grouping handles that.
ELSE → output "Cold start cancelled." and terminate.
<example>137 tracked files → after grouping → ~6 topics (root, src, ui, docs, tests, extensions).</example>

## Step 2 — Scan and group

Run `git ls-files`. Filter out files matching `skip_patterns`. Group remaining files by topic (directory prefix or shared module name). Eliminate any topic whose files all match skip_patterns — do not create a doc for it.
IF topic has more than 3 files → split by subdirectory prefix into sub-topics. Re-evaluate each sub-topic against this rule.
<example>`src` has 9 files → split into `src/auth` (3 files), `src/billing` (4 files), `src/utils` (2 files) → `src/billing` still has 4 → split into `src/billing/invoices` (2 files) and `src/billing/payments` (2 files).</example>
<example>40 tracked files → filter `**/*.lock` → `dist/` matches `**/dist/**` → dist topic eliminated → group remaining into `auth`, `ui`, `proxy` topics.</example>

## Step 3 — Execute CREATE for every topic

<rule>Read `references/extraction.md` once before processing any topic.<example>Load extraction guide before the first topic — not once per topic.</example></rule>
<rule>Run tool calls directly — do not output a plan or summary before or between topics.<example>Agent starts writing `src/auth` doc immediately — does not first output "I will now process auth, billing, proxy..."</example></rule>
<rule>Execute doc-writing CREATE for every topic before advancing to step 4. You must have a physically written file for every topic before proceeding.</rule>
Before executing CREATE for any topic: derive the file path using SKILL.md `<create>` step 1 rules — prefix must reflect the domain layer (e.g. `back`, `front`, `ext`, `test`), never a generic project name like `repo` or `app`.
For each topic:
  1. Read each source file in the topic. Apply the extraction guide to identify behavioral facts — do not use git log messages as content.
  2. Execute the doc-writing full workflow inline (including pre-flight) with the extracted facts. The doc must describe what the code currently does.
<rule>Before advancing to step 4: list `{root}` and count files. Count must equal topic count — if any topic file is missing, write it now.</rule>
<example>`src` topic → read `src/auth.ts`, `src/proxy.ts` → extract behavioral facts → execute doc-writing CREATE inline → write `.pi/notes/back.src.md`.</example>

## Step 4 — Write .last-sync

<rule>Execute the write — do not narrate it. Run `git rev-parse HEAD` and write the result to `{root}/.last-sync`. Not complete until the file is written.</rule>
<example>run `git rev-parse HEAD` → write full hash to `.pi/notes/.last-sync`.</example>

## Step 5 — Write _index.md

Execute `<write-index>`.
