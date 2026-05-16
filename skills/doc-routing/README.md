# doc-routing — Configuration

> Tested on `openai-codex/gpt-5.4-mini`. Works across models.

The placement rules are hardcoded directly in `SKILL.md` inside the `<config>` block.
No external config file is needed — agents read the skill and use the values as-is.

## Current values

| Key | Value | Meaning |
|---|---|---|
| `root` | `.pi/notes/` | Root directory where all agent docs live |
| `subdirs` | `[]` | Predefined subdirectory list. Empty = flat layout — `{subdir}` is a filename prefix only, no directory created |
| `pattern` | `{subdir}.{topic}.md` | Naming convention for every doc file |
| `skip_patterns` | `["**/*.lock", "**/dist/**", "**/*.generated.*", "**/build/**", ".pi/skills/**"]` | Glob patterns always classified as SKIP during SYNC |
| `never_skip` | `[]` | Glob patterns that override `skip_patterns` — always UPDATE or CREATE |
| `.last-sync` | `{root}/.last-sync` | Runtime file — stores last synced commit hash |
| `_index.md` | `{root}/_index.md` | Runtime file — plain TOC regenerated after every write |

## How to change the config

Edit the `<config>` block in `SKILL.md`:

```xml
<config>
  root: .pi/notes/
  subdirs: []
  pattern: "{subdir}.{topic}.md"
  skip_patterns: ["**/*.lock", "**/dist/**", "**/*.generated.*", "**/build/**", ".pi/skills/**"]
  never_skip: []
</config>
```

Update this README table to match after any change.

## .last-sync

Written automatically by the SYNC workflow after each run using `git rev-parse {end}` — always records the end of the last synced range, not necessarily HEAD.

Seed it manually to set a starting point:
```
echo "dfbad62c2d472689add05e380f20cfa75c1ea8fb" > .pi/notes/.last-sync
```

## _index.md

Generated automatically after every CREATE, UPDATE, or SYNC that writes at least one file. Plain markdown list of links — no metadata, no dates:

```markdown
- [back.auth.md](back.auth.md)
- [front.ui.md](front.ui.md)
```

Never edit `_index.md` manually — it is always overwritten by the skill.

## Cold start

On a blank project (no `.last-sync`, no existing docs), any sync intent triggers cold-start mode automatically — no special phrase required. The skill shows a confirmation block before scanning:

```
Cold start detected — no previous sync found.
Root: <repo root>
Skipping: <skip_patterns>
Files in scope: ~N files
Proceed? [yes / cancel]
```

On `yes` → scans all tracked files with `git ls-files`, respects `skip_patterns`, groups into topics, derives domain-layer prefixes (`back`, `front`, `ext`, `test`, etc.), reads source files, applies `references/extraction.md` to extract behavioral facts, executes doc-writing CREATE inline for each topic, writes `.last-sync` with HEAD.

On `cancel` → terminates cleanly, nothing written.

If docs exist but `.last-sync` is missing → hard stop: "No last-sync found. Provide a starting commit or range."

## Extraction guide

`references/extraction.md` is loaded once before processing any topic in COLD-START and SYNC. It defines what to extract from source files:

- **Code** — public function behavioral facts (preconditions, error paths, side effects), not import lists or implementation details
- **Config** — env vars and their runtime effect, not default values that never change
- **TODO/FIXME** — known behavioral limitations only, not task reminders

The quality rule: every extracted fact must describe **behavior**, not structure. "Returns 401 when token is missing" is kept; "Has a token validation function" is skipped.

## skip_patterns vs never_skip

`never_skip` takes precedence over `skip_patterns`. Use it to whitelist specific files inside a broad skip glob:

| Goal | Config |
|---|---|
| Skip all JSON files | add `**/*.json` to `skip_patterns` |
| But always process `package.json` | add `package.json` to `never_skip` |

## Path examples

**`subdirs: []` — flat layout (prefix only, no directory):**

| Topic | Derived prefix | Resolved path |
|---|---|---|
| `auth` | `back` | `.pi/notes/back.auth.md` |
| `form-flow` | `front` | `.pi/notes/front.form-flow.md` |
| `workflow-pi-review` | `github` | `.pi/notes/github.workflow-pi-review.md` |

**`subdirs: ["back", "front"]` — directory layout:**

| Topic | Matched subdir | Resolved path |
|---|---|---|
| `auth` | `back` | `.pi/notes/back/back.auth.md` |
| `form-flow` | `front` | `.pi/notes/front/front.form-flow.md` |
