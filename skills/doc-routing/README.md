# doc-routing — Configuration

> Tested on `openai/gpt-oss-120b`. Works across models — no Claude-specific features used.

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
