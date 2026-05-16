---
skill: "doc-routing"
behavior: "SYNC cold start — sync-entry routes to COLD-START workflow when no .last-sync and no docs; confirmation block shown; cancel terminates; docs exist + no .last-sync → hard stop in sync-entry"
when: "SYNC intent with no .last-sync and no existing docs under {root}"
---

# Test dr-06 — Cold start

## Behavior

> sync-entry (SKILL.md) lists `{root}`, finds no docs and no `.last-sync` → loads `references/cold-start.md` and runs COLD-START workflow. Confirmation block shown before any write. Hard stop fires in sync-entry when docs exist but `.last-sync` is missing.
> **When:** First-ever sync on a blank project.

## Why this test exists

Cold-start is detected from project state (no notes, no `.last-sync`), not from trigger phrases. The confirmation block is the safety gate. Hard stop is reserved for the case where notes exist but `.last-sync` was lost.

## Setup A/B — blank project (no docs, no .last-sync)

No `.pi/notes/`, no `.last-sync`. Git repo with tracked files:

```
src/auth/token.ts
src/proxy/retry.ts
package-lock.json
```

Config:
```xml
<config>
  root: .pi/notes/
  skip_patterns: ["**/*.lock"]
</config>
```

---

## Prompt A — blank project, user confirms

```
sync notes
```
User replies: `yes`

### What to observe

**Step 2 — confirmation block:**
```
Cold start detected — no previous sync found.
Root: <repo root>
Skipping: ["**/*.lock"]
Files in scope: ~3 files
Proceed? [yes / cancel]
```
After `yes` → `git ls-files` scanned, `package-lock.json` filtered, `auth` and `proxy` topics created, `.last-sync` = HEAD, `_index.md` written.

### Checks — Prompt A

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | Cold-start block shown | output contains "Cold start detected — no previous sync found." |
| C2 | Block includes repo root, skip_patterns, file count | all three fields present |
| C3 | No files written before `yes` | no `.md` file created until user confirms |
| C4 | `git ls-files` used for file scan | not `git diff` — all tracked files scanned |
| C5 | skip_patterns applied | `package-lock.json` excluded from topics |
| C6 | `.last-sync` written with HEAD hash | `{root}/.last-sync` contains resolved HEAD hash |
| C7 | `_index.md` generated | `{root}/_index.md` created as plain TOC after writes |

---

## Prompt B — blank project, user cancels

```
sync notes
```
User replies: `cancel`

### Checks — Prompt B

| # | Check | Pass condition |
|---|-------|----------------|
| C8 | Confirmation block shown | same block as Prompt A |
| C9 | Terminate on cancel | output contains "Cold start cancelled." — no files written |
| C10 | `.last-sync` not created | `{root}/.last-sync` does not exist after cancel |

---

## Setup C — docs exist, no .last-sync

`.pi/notes/back.auth.md` exists. No `.last-sync`.

## Prompt C — docs exist, no .last-sync

```
sync notes
```

### Checks — Prompt C

| # | Check | Pass condition |
|---|-------|----------------|
| C11 | Hard stop fires | output is exactly "No last-sync found. Provide a starting commit or range." |
| C12 | No cold-start block shown | "Cold start detected" not in output |
| C13 | No git commands run | no git output — skill terminates before any command |

---

## Fail signals

- Prompt A hard-stops instead of showing confirmation block
- Prompt A writes files before `yes`
- Prompt B writes files or `.last-sync` after `cancel`
- Prompt C shows cold-start block instead of hard-stopping
- Prompt C proceeds to list commits without hard-stopping
