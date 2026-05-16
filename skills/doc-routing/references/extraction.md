---
description: Load when reading source files before writing a doc. Defines what to extract from code, config, and TODO/FIXME comments, and the quality test for separating behavioral rules from structural noise.
---

# Source Extraction Guide

<extraction-sources>
  <source>
    <type>Code</type>
    <extract>Public function signatures + preconditions/postconditions; error handling paths and their status codes; side effects (writes, emits, external calls); branching on env vars or feature flags.</extract>
    <skip>Import lists, helper internals, implementation details that do not affect callers.</skip>
    <example>EXTRACT: "validateToken throws 401 when token is missing or expired." SKIP: "auth.ts exports validateToken, refreshToken, and revokeToken."</example>
  </source>
  <source>
    <type>Config</type>
    <extract>Environment variables and their effect on runtime behavior; limits, thresholds, and timeouts; service dependencies that activate or gate features.</extract>
    <skip>Default values that never change in practice; comments that restate the variable name.</skip>
    <example>EXTRACT: "MAX_RETRIES=3 — requests fail hard after 3 attempts, no further fallback." SKIP: "PORT=3000 — default HTTP port."</example>
  </source>
  <source>
    <type>TODO/FIXME only</type>
    <extract>Known behavioral limitations or intentional deferrals — only when they describe a constraint the agent must know to avoid broken behavior.</extract>
    <skip>All other comment types: task reminders, scaffolding headers, auto-generated blocks, prose explanations of adjacent code.</skip>
    <example>EXTRACT: "// FIXME: rate limiter not enforced for admin tokens — callers must handle throttle themselves." SKIP: "// TODO: add pagination."</example>
  </source>
</extraction-sources>

<quality-rule>
  <rule><requirement>Every extracted fact must describe behavior, not structure.</requirement><example>BEHAVIORAL (keep): "Returns 401 when token is missing." STRUCTURAL (skip): "Has a token validation function."</example></rule>
</quality-rule>

<self-verification>
  <check>Every extracted fact passes the behavioral test — describes what the code does, not what it contains.</check>
  <check>No extracted fact comes from a non-TODO/FIXME comment.</check>
  <check>Config entries include their behavioral effect — not just the variable name and value.</check>
</self-verification>
