---
description: Per-section interview prompts and XML examples for AGENTS.md — load during the interview to get the example for each section.
---

# Per-Section Examples — agents-md

For each section:
1. Ask the user the plain-text question under "Ask:"
2. Use the XML block as the schema — what fields this section needs
3. Apply `<constraints>` from `references/techniques.md` to the content (imperative language, section length, etc.)
4. Write the output

## Table of Contents
- [Section 1 — Project Scope](#section-1)
- [Section 2 — Permissions & Boundaries](#section-2)
- [Section 3 — Hard Constraints](#section-3)
- [Section 4 — Coding Standards](#section-4)
- [Section 5 — Tooling & Commands](#section-5)
- [Section 6 — Default Workflow](#section-6)
- [Section 7 — Verification Checklist](#section-7)
- [Section 8 — Output Expectations](#section-8)
- [Section 9 — Safety Rules](#section-9)
- [Section 10 — Agent Roles](#section-10)
- [Techniques](#techniques)

---

## Section 1 — Project Scope

Ask: "What does this project do, what's the stack, and what areas should the agent focus on?"

```xml
<scope>
  <purpose>E-commerce platform — product catalog, cart, and checkout</purpose>
  <stack>
    <language>TypeScript (strict)</language>
    <framework>Next.js App Router</framework>
    <styling>Tailwind CSS</styling>
    <database>PostgreSQL via Prisma</database>
    <package-manager>pnpm</package-manager>
  </stack>
  <focus>Frontend components and API routes — not infrastructure or DB migrations</focus>
</scope>
```

---

## Section 2 — Permissions & Boundaries

Ask: "What can the agent freely change? What needs your approval? What is completely off-limits?"

```xml
<permissions>
  <allowed>
    <item>Read any file in the repository</item>
    <item>Edit files under src/components/, src/app/, src/lib/</item>
    <item>Run linter and formatter</item>
    <item>Run tests</item>
  </allowed>
  <ask-first>
    <item>Install or remove any package</item>
    <item>Create new files outside src/</item>
    <item>Modify any configuration file (next.config.ts, tsconfig.json, etc.)</item>
    <item>git commit or git push</item>
  </ask-first>
  <never>
    <item>Modify prisma/schema.prisma</item>
    <item>Edit .env or any secrets file</item>
    <item>Push to main or production branch</item>
    <item>Delete files without explicit instruction</item>
  </never>
</permissions>
```

---

## Section 3 — Hard Constraints

Ask: "What are the absolute non-negotiable rules — things that must never happen regardless of context?"

```xml
<constraints>
  <never>Add new npm dependencies without explicit user approval</never>
  <never>Change the database schema</never>
  <never>Perform destructive actions (drop, delete, truncate) without written confirmation</never>
  <never>Modify shared infrastructure or CI/CD configuration</never>
  <never>Introduce breaking changes to public API contracts</never>
</constraints>
```

---

## Section 4 — Coding Standards

Ask: "What are the code style rules, naming conventions, and framework-specific requirements?"
If insufficient: "For example — what naming convention? Any banned patterns? Reuse vs create preference?"

```xml
<standards>
  <naming>
    <components>PascalCase</components>
    <functions>camelCase</functions>
    <files>kebab-case</files>
    <constants>SCREAMING_SNAKE_CASE</constants>
  </naming>
  <rules>
    <!-- hard constraints first -->
    <rule><requirement>No default exports — named exports only.</requirement><example>Write `export function MyComponent()`, never `export default function MyComponent()`.</example></rule>
    <rule><requirement>No `any` type — use `unknown` and narrow.</requirement><example>Replace `param: any` with `param: unknown` and add a type guard before use.</example></rule>
    <rule><requirement>Use absolute imports via @/ alias — no ../.. chains.</requirement><example>Import as `import { Button } from '@/components/button'`, not `'../../components/button'`.</example></rule>
    <!-- soft preferences after -->
    <rule><requirement>Reuse existing utilities before creating new ones.</requirement><example>Search for `formatDate` in @/lib before writing a new date formatter.</example></rule>
    <rule><requirement>Use Tailwind classes only — no inline styles.</requirement><example>Replace `style={{ marginTop: 8 }}` with `className="mt-2"`.</example></rule>
  </rules>
</standards>
```

---

## Section 5 — Tooling & Commands

Ask: "What are the exact commands the agent must use? Include all flags."
If insufficient: "For example — how do you run tests? What's the lint command with its flags?"

```xml
<commands>
  <test>pnpm test --run</test>
  <lint>pnpm eslint src/ --max-warnings 0</lint>
  <typecheck>pnpm tsc --noEmit</typecheck>
  <format>pnpm prettier --write src/</format>
  <build>pnpm build</build>
  <dev>pnpm dev</dev>
</commands>
```

---

## Section 6 — Default Workflow

Ask: "What steps must the agent always follow before making any change?"
Note: propose the example below as a strong default — most projects benefit from this exact sequence.

```xml
<workflow>
  <step order="1">Analyze — understand the full request before touching any file</step>
  <step order="2">Locate — find all files relevant to the change using search</step>
  <step order="3">Plan — write a one-paragraph plan; stop if scope exceeds 5 files</step>
  <step order="4">Edit — make the smallest change that achieves the goal</step>
  <step order="5">Verify — run lint, typecheck, and tests before responding</step>
  <rule><requirement>Stop and report before proceeding when the plan reveals unexpected scope.</requirement><example>Changing an API endpoint turns into a DB migration — stop and ask before touching the schema.</example></rule>
  <rule><requirement>Report a failing command after two identical errors — never retry blindly.</requirement><example>If `pnpm build` fails twice with the same TS error, report it instead of running a third time.</example></rule>
  <rule><requirement>Ask one clarifying question then stop when intent is uncertain.</requirement><example>Ask "Should I update all usages or only this file?" before making a widespread change.</example></rule>
</workflow>
```

---

## Section 7 — Verification Checklist

Ask: "What must be true before you consider a task complete?"

```xml
<verification>
  <check>pnpm test --run passes with no failures</check>
  <check>pnpm eslint reports zero warnings</check>
  <check>pnpm tsc --noEmit reports no errors</check>
  <check>No files outside the allowed scope were modified</check>
  <check>No new dependencies were added without approval</check>
  <check>Output matches the original requirement — re-read the request before confirming done</check>
</verification>
```

---

## Section 8 — Output Expectations

Ask: "Should the agent implement directly or propose first? What format should responses take?"

```xml
<output>
  <mode>implement-directly</mode>
  <!-- alternatives: propose-first, explain-only -->
  <format>show changed files as diffs — do not reprint unchanged files</format>
  <explanation>brief — one sentence per non-obvious decision</explanation>
  <never>summarize what you just did at the end — the diff speaks for itself</never>
</output>
```

---

## Section 9 — Safety Rules

Ask: "What are the data safety rules? What actions require explicit user confirmation?"

```xml
<safety>
  <never>Output API keys, tokens, passwords, or secrets in any form</never>
  <never>Hardcode sensitive values — reference environment variables only</never>
  <never>Log or print user PII</never>
  <confirm-before>
    <action>Any action that cannot be undone</action>
    <action>Calls to external APIs with side effects</action>
    <action>Sending emails, messages, or notifications</action>
  </confirm-before>
</safety>
```

---

## Section 10 — Agent Roles

Ask: "Do different agents have different responsibilities in this project?"
Only include if multiple specialized agents share the codebase.

```xml
<roles>
  <role name="frontend">
    <scope>src/components/, src/app/ — UI and routing only</scope>
    <never>Touch API routes or server actions</never>
  </role>
  <role name="backend">
    <scope>src/app/api/, src/lib/db/ — data and business logic</scope>
    <never>Touch UI components or styling</never>
  </role>
</roles>
```

