---
skill: "doc-routing"
behavior: "SYNC full flow — confirmation gate, classification, delegation, and .last-sync tracking across two sequential runs from a blank project"
when: "Any SYNC workflow from project start through ongoing use"
---

# Test dr-05 — Full SYNC flow (blank project)

## Behavior

> Run 1 syncs a project with no existing docs and no `.last-sync`. Run 2 re-syncs after docs exist. Together they cover the confirmation gate, classification rules, delegation, and `.last-sync` tracking.
> **When:** First and second SYNC from a blank project.

## Why this test exists

From a blank project, the full SYNC path must work end-to-end without pre-seeded state. Run 1 creates docs and writes `.last-sync`. Run 2 uses that `.last-sync` to find new commits and produces UPDATEs — verifying that the written hash was correct.

## Setup

Blank project: no `.pi/notes/`, no `.last-sync`. Git repo with this history:

```
HEAD      abc1234  feat: add payment webhook handler
HEAD~1    def5678  feat: add proxy retry logic
HEAD~2    ghi9012  refactor: token.ts minor cleanup
HEAD~3    jkl3456  chore: update package-lock.json
```

Changed files per commit:
- `jkl3456`: `package-lock.json`
- `ghi9012`: `src/auth/token.ts` (minor rename, no behavioral change)
- `def5678`: `src/proxy/retry.ts` (new file)
- `abc1234`: `src/payments/webhook.ts` (new file)

Config:
```xml
<config>
  skip_patterns: ["**/*.lock"]
  never_skip: ["**/*.test.ts"]
</config>
```

---

## Run 1 — First sync, no docs, no .last-sync

**Prompt:**
```
sync from last 4 commits
```

### What to observe

**Step 2 — confirmation output:**
```
Last sync: Never synced
Proposed sync — 4 commits:
  · abc1234 feat: add payment webhook handler
  · def5678 feat: add proxy retry logic
  · ghi9012 refactor: token.ts minor cleanup
  · jkl3456 chore: update package-lock.json
Confirm? [yes / provide different range]
```
User replies: `yes`

**Step 4 — classification:**

| Files | Topic | Expected | Reason |
|---|---|---|---|
| `package-lock.json` | — | SKIP | matches `**/*.lock` |
| `src/auth/token.ts` | `auth` | CREATE | no doc exists |
| `src/proxy/retry.ts` | `proxy` | CREATE | no doc exists |
| `src/payments/webhook.ts` | `payments` | CREATE | no doc exists |

**Step 6 — .last-sync:**
`{root}/.last-sync` = hash of `HEAD` = `abc1234`

### Checks — Run 1

| # | Check | Pass condition |
|---|-------|----------------|
| C1 | "Never synced" shown | no `.last-sync` → label shows "Never synced" not a hash |
| C2 | 4 commits listed before confirmation | git log output shown — user must confirm before any write |
| C3 | No write before confirmation | no `.md` file created until user says yes |
| C4 | `package-lock.json` → SKIP | pattern match — never handed to CREATE or UPDATE |
| C5 | No existing doc → CREATE | `auth`, `proxy`, `payments` all routed to CREATE workflow |
| C6 | doc-writing loaded before any write | delegation guard fires — doc-writing SKILL.md read first |
| C7 | `.last-sync` written to `{root}/` | `{root}/.last-sync` = `abc1234` — not repo root |
| C8 | Never checks uncommitted changes | skill does not run `git diff --name-only` at any point |

---

## Run 2 — Second sync, docs exist, .last-sync = abc1234

After Run 1: `.pi/notes/` has `auth`, `proxy`, `payments` docs. New commit added:

```
HEAD      mno7890  fix: update auth token expiry logic
```
Changed file: `src/auth/token.ts` (behavioral change — expiry field renamed)

**Prompt:**
```
sync notes
```
*(no argument — uses `.last-sync`)*

### What to observe

**Step 1:** reads `.last-sync` → `start = abc1234`, `end = HEAD`

**Step 2 — confirmation output:**
```
Last sync: abc1234 — 2026-05-15 — feat: add payment webhook handler
Proposed sync — 1 commit:
  · mno7890 fix: update auth token expiry logic
Confirm? [yes / provide different range]
```
User replies: `yes`

**Step 4 — classification:**

| Files | Topic | Expected | Reason |
|---|---|---|---|
| `src/auth/token.ts` | `auth` | UPDATE | doc exists + diff impacts documented behavior |

**Step 6 — .last-sync:**
`{root}/.last-sync` = hash of new `HEAD` = `mno7890`

### Checks — Run 2

| # | Check | Pass condition |
|---|-------|----------------|
| C9 | `.last-sync` used as start | `start = abc1234` read from file — not re-derived from git log |
| C10 | Last sync message shows Run 1 commit | "Last sync: abc1234 — ... — feat: add payment webhook handler" |
| C11 | Only 1 new commit shown | commits already covered by Run 1 not re-listed |
| C12 | Behavioral change → UPDATE | `auth/token.ts` diff affects `back.auth.md` content → UPDATE, not SKIP |
| C13 | UPDATE targets affected section only | doc-writing UPDATE runs on the expiry section — other sections untouched |
| C14 | `.last-sync` updated to new HEAD | `{root}/.last-sync` = `mno7890` after Run 2 completes |

---

## Fail signals

- Run 1: writes files before user confirms
- Run 1: `package-lock.json` handed to CREATE
- Run 1: `.last-sync` not created, or created at repo root
- Run 2: ignores `.last-sync` and shows all 5 commits again
- Run 2: `auth/token.ts` classified SKIP despite behavioral change
- Run 2: writes HEAD hash (`mno7890`) before step 5 completes
