# Agent Rules -  MonoRepo

This file defines **mandatory behavior and constraints** for my AI agent working in this repository.
It is the primary source of truth for repo-specific rules.

---

## MANDATORY PRE-FLIGHT CHECK 🚨

**BEFORE reading ANY code files, ALWAYS follow these steps:**

### Step 1: Identify the Domain
- Frontend task? (`doitnow-front`)
- Backend task? (`doitnow-back`)
- Migration/database change?
- Testing?

### Step 2: Load the Index File FIRST

**Frontend** = `pl/instructions/front/front.index.md`
**Backend** = `pl/instructions/back/back.index.md`

The index explains the project structure, patterns, and load documentation.

> *The pi-context extension has made all documentation available in the "Available Content Files" section above.*

**BEFORE proposing solutions or reading code, you MUST:**
1. **Review "Available Content Files"** — scan file descriptions shown in stub
2. **"Match semantically"** — find files whose descriptions relate to user's question

---

## How Matching Works

**pi-context shows file stubs with descriptions only:**

```
pl/instructions/front/front-auth.md        - Authentication and authorization - OAuth flow
pl/instructions/front/front-form-data-flow.md - Form schema loading, data fetching, and submission flow
```

**You must match user's question against these descriptions semantically.**

**Keywords are for validation AFTER loading, not for initial matching.**

### Decision Rule

- **If** user's question semantically matches a file's description
- **THEN** proceed to load that file
- **ELSE** confirm relevance

### Example: Correct Workflow

User asks: *"How do I handle form submission in XaaS actions?"*

**Your process:**
1. Scan "Available Content Files" stubs
2. Locate: `front.xaas-data-flow.md` — "Form schema loading, data fetching, and submission flow"
3. Load: `read pl/instructions/front/front-form-data-flow.md`
4. Verify keywords: `[Form, schema, Swagger, submission, XaaS, external API]`
5. Matched!
6. Answer using loaded content + also load `front.xaas-form-pattern.md`

**Wrong approach:**
- ❌ Scan "Available Content Files" stubs
- ❌ Grep codebase for "XaaS"
- ❌ Miss domain component files

### Performance Tip

Load files in parallel when multiple matches found:

```bash
read pl/instructions/front/front-form-data-flow.md
read pl/instructions/front/front.xaas-form-pattern.md
```

Not sequentially (slower).

---

**ONLY AFTER completing the pre-flight check may you proceed to read code files.**

---

## Code Quality Principles (Required Defaults)

All code changes must follow these principles unless explicitly overridden:

- **Prefer simple**: Clear naming, small functions, single responsibility.
- **"Uncle Bob"**: Clear naming, small functions, single responsibility.
- **"Clean Code"**: prefer simple, readable solutions.
- **"Minimal interface errors"**: avoid unneeded indirection.
- **"Consistency over Personal preference"**: follow existing patterns in the codebase.
- **"Readability over Cleverness"**.

These are "guiding principles", not an excuse for large refactors.

---

## Architecture Rules

Respect existing architectural boundaries:
- Frontend: component and data-flow patterns as documented.
- Business logic must not depend directly on infrastructure or framework details.
- Dependencies must follow the project's layering rules.
- Do not introduce new architectural patterns unless explicitly requested.

---

## Database, Schema & Migrations (Strict)

**All database schema or service-catalog changes must only be implemented via migrations.**
- The agent must never directly modify production data outside a migration.

Required references before any work:
- `pl/skills/din-migrations/SKILL.md`
- `pl/instructions/back/back.migrations.md`

---

## Development Environment Constraints

This MonoRepo runs on a **remote Linux server via VS Code remote SSH**.

### Filesystem Rules

Root path: `/pl/...`
- **Never** use Windows-style backslashes (`\`).
- Always use forward slashes (`/`).

### File Creation Rules

**Never use the VS Code `create_file` tool** (causes false conflicts on remote).
- Use heredoc scripts with triple-quoted strings: `$(cat << 'EOF'...)`
- or `bash` heredocs (`cat << 'EOF'`) for plain content.

---

## Context Loading Rules (Mandatory)

**See "MANDATORY PRE-FLIGHT CHECK" at the top of this file for the complete workflow.**

### Quick Reference

**Frontend (`doitnow-front`):**
- Always load: `_pl/instructions/front/front.index.md`
- Load additional files based on trigger words (see pre-flight check table)

**Backend (`doitnow-back`):**
- Always load: `_pl/instructions/back/back.index.md`
- Load additional files based on trigger words (see pre-flight check table)

**Documentation:**
- Architecture and system behavior: `documentation/docs/doitnow-v4/`

**Skills:**
- Check skill descriptions for relevant workflows before implementing (ensures correct workflow followed)

---

## When to Update Documentation

### #1. Code Changes Affect Documented Patterns

If you modify architectures, workflows, or patterns that are already documented, update the corresponding:
- Frontend patterns = `_pl/instructions/front/front/`
- Backend patterns = `_pl/instructions/back/`
- System architecture = `documentation/docs/doitnow-v4/`

### #2. Discovery of New Patterns

If you discover undocumented patterns while working → See **MANDATORY POST-TASK CHECK** at end of this file.

---

## MANDATORY POST-TASK CHECK 🚨

**BEFORE marking any task as complete, ALWAYS run this self-assessment:**

### Step 1: Did You Learn Something?

**Ask yourself these questions:**
- ❓ Did I spend >10 minutes searching for information?
- ❓ Did I read code across multiple files to understand a pattern?
- ❓ Did I trace code to understand a non-obvious issue?
- ❓ Did I discover an architecture decision that wasn't documented?
- ❓ Did I learn a common mistake or anti-pattern?
- ❓ Did I find a best practice by reading implementation code?
- ❓ Did I debug an issue caused by missing documentation?
- ❓ Would the NEXT agent have to repeat my investigation?

---

### Step 2: Self-Assessment Workflow

**If YES to ANY question → Documentation is MANDATORY**

---

### A. No Learning Required (Quick Fix)

**If you:**
- ✅ Found answer immediately in existing docs
- ✅ Made trivial change (<5 min, no investigation)
- ✅ Fixed obvious typo or formatting

**Then:**
- Task complete — no documentation needed
- Proceed to final output

---

### B. Learning Occurred (Investigation Required)

**If you:**
- ❌ Spent >10 min searching
- ❌ Traced code to understand pattern
- ❌ Debugged non-obvious issue
- ❌ Discovered undocumented pattern

**Then you MUST:**

1. **Load the documentation skill:**
   ```bash
   read .pi/skills/din-manage-instructions/SKILL.md
   ```

2. **Determine what to document:**
   - New pattern? → Create new file
   - Clarify existing? → Update existing file
   - Common mistake? → Add to pitfalls file
   - Workflow? → Document step-by-step

3. **Create/update documentation following the skill.**

4. **Verify with mandatory checks** (see Step 3 below).

---

### Step 3: Follow the Skill and Verify Quality

**Load and follow the skill:**
```bash
read .pi/skills/din-manage-instructions/SKILL.md
```

**Use the appropriate workflow from the skill:**
- "Workflow: Create New Instruction File" → for new patterns
- "Workflow: Update Existing File" → for clarifications

**Critical Verification (MANDATORY):**
```bash
grep "your-file.md" pl/instructions/front/front.index.md
```
- Verify file is linked in index.
- "Workflow: Add frontmatter" → for fills missing metadata.

**If grep returns nothing — file is orphaned! Update index immediately.**

---

### Step 4: Output Task Completion with Documentation Status

**Only after completing all checks, output this template:**

```
**TASK COMPLETE**

Completed: [list]
Solved user's problem: [brief description]
Updated: `.pi/instructions/domains/<file>.md`
Added: [brief summary of what was documented]
Tests: [tests/updated/added]
Verified: File linked in index (grep confirmed)

Documentation:
[If learning occurred and was documented]
✅ Updated: `_pl/instructions/front/<file>.md`
✅ Added: [brief summary of what was documented]
✅ Tests: [tests/updated/added]
✅ Verified: File linked in index (grep confirmed)

[If no learning occurred:]
✅ No documentation needed (trivial change)
✅ [Any testing or verification steps]
```

**The documentation section proves you completed the POST-TASK CHECK.**

---

## Common Mistake: Skipping Documentation

**Wrong:**
- ❌ 1. Spend 30 min debugging
- ❌ 2. Fix the bug
- ❌ 3. "Task complete!" ← INCOMPLETE — no documentation

**Correct:**
- ✅ 1. Spend 30 min debugging
- ✅ 2. Fix the bug
- ✅ 3. Run POST-TASK CHECK → Document pattern → Verify with grep
- ✅ 4. Output completion template with documentation status
- ✅ 5. "Task complete!" ✓

**Impact:** First agent +10 min to document = Next 10 agents -28 min each = **Net savings: 270 min**

---

## Enforcement

**This is not a suggestion — it's a requirement.**

**If you complete a task without documenting significant learning:**
- ❌ Task is considered INCOMPLETE
- ❌ Future agents will waste time
- ❌ Documentation debt increases
- ❌ Patterns remain tribal knowledge

**When in doubt:**
Ask: *"Would I want this documented if I encountered this problem?"*
- If YES → Document it
- If MAYBE → Document it
- If NO → Only skip if truly trivial (<5 min task)

---

## Quick Reference: When to Document

| Time Spent               | Documentation Required? |
|--------------------------|-------------------------|
| <5 min (obvious fix)     | Probably no             |
| 5–10 min (minor investigation) | **YES — Mandatory** |
| 10–30 min (pattern discovery)  | **YES — Mandatory** |
| >30 min (deep investigation)   | **YES — Critical!!** |

**Rule of thumb:** If you learned it, document it.

For questions or ambiguity, defer to the maintainers and existing onboarding documentation.
