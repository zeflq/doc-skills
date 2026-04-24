# doc-skills

A collection of skills for writing agent-readable documents — structured files that AI agents load as context to govern their behavior.

## Skills

### agent-doc

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

### agents-md

Create or improve `AGENTS.md` / `CLAUDE.md` — the unified project instruction file recognized by Claude Code, Pi, and GitHub Copilot.

**Use when:** you want to set up or improve an agent's project-level instructions.

Same three modes as agent-doc. Produces a fixed opinionated 10-section structure:

| # | Section | Required? |
|---|---|---|
| 1–3 | Project Scope, Permissions, Hard Constraints | Required |
| 4–7 | Coding Standards, Tooling, Workflow, Verification | Required |
| 8–10 | Output Expectations, Safety Rules, Agent Roles | Recommended |

Delegates all writing technique decisions to the `agent-doc` skill — install both.

**Use agent-doc instead** when you need a freeform document with project-specific sections.

---

### skill-creator

> Copy of the [Claude Code built-in skill-creator](https://claude.ai/code) — included here so it is available in Pi and other runtimes.

Create new skills, improve existing ones, and measure skill performance with evals.

**Use when:** you want to build a new skill from scratch, iterate on an existing one, or benchmark skill quality across models.

Uses `agent-doc` to write all agent-readable reference files bundled with the skill.

---

## Architecture

```
agent-doc          ← core: writing techniques, format rules, verification
   ├── agents-md   ← delegates to agent-doc for all writing decisions
   └── skill-creator ← delegates to agent-doc for reference file authoring
```

`agent-doc` is the single source of truth for technique and writing guide rules. The other skills call it rather than duplicating its references.

---

## What makes these docs agent-optimized

All skills apply the same writing techniques owned by `agent-doc`:

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
| [Claude Code](https://claude.ai/code) | `~/.claude/skills/` or `.claude/skills/` | `/agent-doc` · `/agents-md` · `/skill-creator` |
| [Pi](https://github.com/badlogic/pi-mono) | `~/.agents/skills/` or `.agents/skills/` | `/skill:agent-doc` · `/skill:agents-md` |
| [GitHub Copilot](https://docs.github.com/en/copilot) | `~/.copilot/skills/` or `.github/skills/` | `/agent-doc` · `/agents-md` |

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

> Install `agent-doc` first — `agents-md` and `skill-creator` depend on it.

**Universal (Claude Code, Pi, Copilot):**
```bash
cp -r skills/agent-doc ~/.agents/skills/
cp -r skills/agents-md ~/.agents/skills/
```

**Pi** — install as a pi package via git ([pi package commands](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent#package-commands)):
```bash
pi install git:github.com/zeflq/doc-skills
```

Or project-local:
```bash
pi install -l git:github.com/zeflq/doc-skills
```

Skills are then available as `/skill:agent-doc` and `/skill:agents-md`.

**Claude Code:**
```bash
claude skill install agent-doc.skill
claude skill install agents-md.skill
```
