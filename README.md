# doc-skills

A collection of skills for writing agent-readable documents — structured files that AI agents load as context to govern their behavior.

## Skills

### doc-writing

Write or improve any agent-readable `.md` document — architecture guides, runbooks, API contracts, style guides, database schema guides, or any file an agent will load as context.

**Use when:** you need a structured document that an AI agent will read, with sections tailored to the document's purpose.

**Three modes — no interview needed if you already have content:**

| You provide | Mode | What happens |
|---|---|---|
| A file path or pasted content | **REWRITE** | Rewrites the full document applying agent-optimized techniques — no questions asked |
| "update section X" | **UPDATE** | Shows a section summary, rewrites only the target section |
| Nothing | **CREATE** | Interviews you to discover sections, then writes |

Applies structured writing techniques to every section: XML tags for structured items, trigger word tables for routing, numbered steps for sequences, hard constraints before soft guidelines.

---

### agentsmd

Create or improve `AGENTS.md` / `CLAUDE.md` — the unified project instruction file recognized by Claude Code, Pi, and GitHub Copilot.

**Use when:** you want to set up or improve an agent's project-level instructions.

Same three modes as doc-writing. Produces a fixed opinionated 10-section structure:

| # | Section | Required? |
|---|---|---|
| 1–3 | Project Scope, Permissions, Hard Constraints | Required |
| 4–7 | Coding Standards, Tooling, Workflow, Verification | Required |
| 8–10 | Output Expectations, Safety Rules, Agent Roles | Recommended |

Delegates all writing technique decisions to the `doc-writing` skill — install both.

**Use doc-writing instead** when you need a freeform document with project-specific sections.

---

### doc-routing

The canonical doc workflow for agent-readable `.md` files in a project — with pi-context discovery rules baked in.

**Use when:** you need to decide where a file goes, when to split an oversized doc, how to organize context files across a project, or how to map git changes to doc updates.

Four workflows:

| Trigger | Workflow | What happens |
|---|---|---|
| "add", "create", "new doc" | **CREATE** | Resolves path per pi-context rules → delegates to doc-writing CREATE |
| "update", "edit", "fix" | **UPDATE** | Confirms single-responsibility → delegates to doc-writing UPDATE |
| "sync", "docs after code", "git changes" | **SYNC** | Groups changed files by topic → plans UPDATE / CREATE / SKIP → waits for confirmation |
| "split", "extract", "too large" | **SPLIT** | Breaks into subtopic files → original becomes index → updates cross-links |

Enforces: `description:` frontmatter, config dir placement (`.pi/` / `.claude/` / `.agents/`), kebab-case naming, 5 KB size limit, no ephemeral change-log blocks.

Delegates all writing to `doc-writing` — install both.

---

## Architecture

```
doc-writing          ← core: writing techniques, format rules, verification
   ├── agentsmd     ← delegates to doc-writing for all writing decisions
   └── doc-routing← delegates to doc-writing for CREATE / UPDATE / SYNC / SPLIT workflows
```

`doc-writing` is the single source of truth for technique and writing guide rules. `agentsmd` and `doc-routing` call it rather than duplicating its references. `doc-routing` is the canonical workflow for doc placement, doc updates, and git-based doc sync.

## Using with skill-creator

### Write the SKILL.md

If the `doc-writing` skill is available, use it to write all reference files and any agent-readable `.md` files the skill will load as context. Write `SKILL.md` itself directly — it is a process document for Claude, not an agent-consumed context file.

---

## Prompts

No standalone `/doc-sync` prompt ships with this package anymore.

Use the `doc-routing` skill's SYNC workflow when you need to map git changes to doc updates. It scans the selected commit range, groups files by topic, shows an UPDATE / CREATE / SKIP plan, and waits for confirmation before any write happens.

---

## What makes these docs agent-optimized

All skills apply the same writing techniques owned by `doc-writing`:

- **XML structure** — named blocks with clear boundaries, reliably parsed by LLMs
- **Constraint ordering** — hard rules ("never", immediate actions) always before soft guidelines
- **Examples on every rule** — concrete examples prevent misapplication
- **Trigger word tables** — keyword → action mappings for consistent routing
- **Self-verification loops** — `<check>` checklists the agent runs before confirming completion
- **Deduplication** — same-intent rules consolidated before writing, even when worded differently

---

## Compatibility

| Platform | Install path | Invoke |
|---|---|---|
| [Claude Code](https://claude.ai/code) | `~/.claude/skills/` or `.claude/skills/` | `/doc-writing` · `/agentsmd` |
| [Pi](https://github.com/badlogic/pi-mono) | `~/.agents/skills/` or `.agents/skills/` | `/skill:doc-writing` · `/skill:agentsmd` · `/skill:doc-routing` |
| [GitHub Copilot](https://docs.github.com/en/copilot) | `~/.copilot/skills/` or `.github/skills/` | `/doc-writing` · `/agentsmd` |

---

## Testing

### Evaluation checklist

Every generated document is evaluated against 7 checks:

| # | Check |
|---|---|
| 1 | Hard rules appear before soft in every mixed section |
| 2 | No rule or content block duplicated by intent anywhere in the document |
| 3 | Every section that routes on user input uses a trigger word table |
| 4 | Every workflow document ends with `<self-verification>` using `<check>` elements |
| 5 | Every rule and step has a concrete `<example>` |
| 6 | No section exceeds 10–15 lines |
| 7 | `description:` frontmatter present |

### Real-world results ( MonoRepo AGENTS.md)

Both skills were tested by rewriting a real 341-line human-written AGENTS.md across multiple runs and two models. Checks: ✓ pass · ✗ fail

| Check | Claude Sonnet 4.6 (best run) | GPT-5.4-mini (best run) |
|---|---|---|
| Hard before soft | ✓ | ✗ hard rule last in one section |
| No duplication | ✗ migration rule in 2 sections | ✓ |
| Routing uses trigger table | ✓ | ✓ |
| Self-verification last | ✓ | ✓ |
| Every rule has `<example>` | ✓ | ✓ |
| Section ≤15 lines | ✓ | ✓ |
| `description:` frontmatter | ✓ | ✓ |
| **Total failures (best run)** | **1** | **1** |

Both models reach 1 failure on their best run — on different checks.

Key observations:
- Claude is consistent on examples and section length when the skill is lean — regresses when the skill is over-specified
- GPT-5.4-mini struggles with constraints ordering but reliably adds per-rule examples
- The migration duplication failure is persistent across Claude runs — an intent-level detection ceiling

### Scenario test suite

18 targeted scenarios in `tests/scenarios/` — one per technique. Each scenario has a concrete input, what to observe, and pass/fail checks. Run via the evaluator in `tests/evaluate.md`.

| Scenarios | Coverage |
|---|---|
| 01–10 | Core techniques (XML, tables, imperative language, constraints, steps, verification, examples, frontmatter, trigger tables, if-then) |
| 11–15 | Deduplication, constraints and sequential workflow applied to any section name |
| 16–17 | Multi-file document systems, token budget |
| 18 | Pre-write intent-level deduplication |

---

## Install

> Install `doc-writing` first — `agentsmd` and `doc-routing` depend on it.

**Universal (Claude Code, Pi, Copilot):**
```bash
cp -r skills/doc-writing ~/.agents/skills/
cp -r skills/agentsmd ~/.agents/skills/
cp -r skills/doc-routing ~/.agents/skills/
```

**Pi** — install as a pi package via git ([pi package commands](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#package-commands)):
```bash
pi install git:github.com/zeflq/doc-skills
```

Or project-local:
```bash
pi install -l git:github.com/zeflq/doc-skills
```

Skills are then available as `/skill:doc-writing`, `/skill:agentsmd`, and `/skill:doc-routing`. Use `doc-routing` for all doc placement, updates, and sync planning.

**Claude Code — via plugin marketplace (recommended):**

Installs all three skills as one plugin and lets you pull updates with `/plugin`.

```
/plugin marketplace add zeflq/doc-skills
/plugin install doc-skills@doc-skills
```

Testing local changes before pushing? Point the marketplace at the repo path instead:

```
/plugin marketplace add /path/to/doc-skills
/plugin install doc-skills@doc-skills
```

Then invoke with `/doc-writing`, `/agentsmd`, `/doc-routing`.

**Claude Code — manual copy (no marketplace):**
```bash
cp -r skills/doc-writing skills/agentsmd skills/doc-routing ~/.claude/skills/
```
