---
skill: "doc-routing"
behavior: "Path resolution — topic + subdir derive the correct path from hardcoded config"
when: "Any CREATE or UPDATE workflow where a file path must be resolved"
---

# Test dr-02 — Path resolution

## Behavior

> The skill reads `root`, `subdirs`, and `pattern` from the hardcoded `<config>` block and derives the full file path before writing anything.
> **When:** Any CREATE or UPDATE that requires resolving a target path.

## Why this test exists

Wrong paths silently place files in the wrong location. The skill must derive the path mechanically from config — never guess, never use a flat root when a subdir applies.

## Input

Run the doc-routing skill three times, once per prompt:

**Prompt A — known subdir:**
```
create a doc about proxy error handling in the back domain
```

**Prompt B — front subdir:**
```
add a doc about form validation patterns for the front domain
```

**Prompt C — no subdir mentioned:**
```
create a doc about deployment checklist
```

## What to observe

Config in `SKILL.md`:
```
root: .pi/notes/
subdirs: [front, back]
pattern: {subdir}.{topic}.md
```

| Prompt | Topic | Subdir | Expected path |
|---|---|---|---|
| A | `proxy-error-handling` | `back` | `.pi/notes/back/back.proxy-error-handling.md` |
| B | `form-validation-patterns` | `front` | `.pi/notes/front/front.form-validation-patterns.md` |
| C | `deployment-checklist` | derived: `deployment` | `.pi/notes/deployment/deployment.deployment-checklist.md` |

## Checks

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Prompt A resolves correct path | Path is `.pi/notes/back/back.proxy-error-handling.md` (or close slug) — not a flat root path |
| C2 | Prompt B resolves correct path | Path is `.pi/notes/front/front.form-validation-patterns.md` |
| C3 | Pattern applied correctly | Filename follows `{subdir}.{topic}.md` — subdir prefix present in filename |
| C4 | Root from config used | Path starts with `.pi/notes/` — not `.`, `notes/`, or any other root |
| C5 | No match derives new subdir | Prompt C derives `deployment` from topic and places at `.pi/notes/deployment/deployment.deployment-checklist.md` |

## Fail signals

- File placed at `.pi/notes/proxy-error-handling.md` (missing subdir folder and prefix)
- File placed at `back/back.proxy-error-handling.md` (missing root)
- Filename uses spaces or uppercase (`Proxy Error Handling.md`)
- Skill invents a subdir not in config (e.g. `db/`, `infra/`)
- Prompt C writes without asking which subdir to use
