# doc-routing — Configuration

The placement rules are hardcoded directly in `SKILL.md` inside the `<config>` block.
No external config file is needed — agents read the skill and use the values as-is.

## Current values

| Key | Value | Meaning |
|---|---|---|
| `root` | `.pi/notes/` | Root directory where all agent docs live |
| `subdirs` | `[]` | Valid subdirectories under root |
| `pattern` | `{subdir}.{topic}.md` | Naming convention for every doc file |
| `.last-sync` | `{root}/.last-sync` | Runtime file — stores last synced commit hash |

`.last-sync` is written automatically by the SYNC workflow after each run. Seed it manually with a commit hash to set the starting point:
```
echo "dfbad62c2d472689add05e380f20cfa75c1ea8fb" > .pi/notes/.last-sync
```

## How to change the config

Edit the `<config>` block in `SKILL.md`:

```xml
<config>
  root: .pi/notes/
  subdirs: []
  pattern: "{subdir}.{topic}.md"
</config>
```

Update this README table to match after any change.

## Examples

| Topic | Subdir | Resolved path |
|---|---|---|
| `auth` | `back` | `.pi/notes/back/back.auth.md` |
| `form-flow` | `front` | `.pi/notes/front/front.form-flow.md` |
| `proxy-errors` | `back` | `.pi/notes/back/back.proxy-errors.md` |
