---
description: Load when you need a concrete example of a correctly formatted agent-readable document.
---

# Example — Agent-Readable Document

This is a complete correctly formatted output. Use it as a reference when writing any section.

```xml
---
description: Load when deploying the backend service to production.
---

<context-routing>
| Request type | Load first |
|---|---|
| Billing / invoices / payments | `docs/billing-rules.md` |
| User accounts / authentication | `docs/auth-rules.md` |
| Reporting / analytics | `docs/reporting-rules.md` |
</context-routing>

<pre-deployment>
  <rule><requirement>Never deploy on Fridays.</requirement><example>Block release when `date +%u` returns `5`.</example></rule>
  <rule><requirement>Never deploy without a passing test suite.</requirement><example>Run `pnpm test` and require exit code `0` before continuing.</example></rule>
  <rule><requirement>Confirm staging validation is complete for the release tag.</requirement><example>Require sign-off on `staging-checklist.md` before promoting.</example></rule>
</pre-deployment>

<deployment-steps>
  <step number="1"><action>Pull the latest image from the registry.</action><example>`docker pull registry.example.com/backend:latest`</example></step>
  <step number="2"><action>Run database migrations.</action><example>`pnpm db:migrate --env=production`</example></step>
  <step number="3"><action>Swap the load balancer to the new instance.</action><example>Set target group weight to `100%` for `backend-v2`.</example></step>
</deployment-steps>

<post-deployment>
  <rule><requirement>Roll back immediately when error rate exceeds 1%.</requirement><example>Trigger rollback when `5xx_rate > 1%` for 1 minute.</example></rule>
  <rule><requirement>Monitor error rate for 10 minutes after cutover.</requirement><example>Watch `5xx_rate` dashboard from T+0 to T+10.</example></rule>
</post-deployment>

<post-deployment-check>
  IF error rate exceeds 1% during the monitoring window:
    <step number="1"><action>Immediately route traffic back to the previous stable instance.</action><example>Set target group weight to `100%` for `backend-v1`.</example></step>
    <step number="2"><action>Redeploy the previous stable image tag.</action><example>`docker pull registry.example.com/backend:previous-stable`</example></step>
  ELSE IF error rate stays below 1% for 10 minutes:
    <step number="1"><action>Mark the deployment healthy and close the monitoring window.</action><example>Update the release ticket status to "production-verified".</example></step>
  END IF
</post-deployment-check>

<self-verification>
  <check>Friday gate enforced before any deployment action.</check>
  <check>Test suite passed before deployment started.</check>
  <check>Migrations completed before load balancer swap.</check>
  <check>Error rate monitored for 10 minutes after cutover.</check>
</self-verification>
```

Key patterns to follow in every document:
- Every `<rule>` has `<requirement>` and `<example>` as child elements
- Hard rules (containing "never" or requiring immediate action) appear before soft rules
- Sequential sections use numbered `<step>` elements — not `<rule>` elements
- Sections that route behavior based on user input use a 2-column trigger word table — not prose or `<rule>` elements
- Conditional logic uses explicit IF / ELSE IF / END IF blocks with numbered `<step>` elements inside each branch — never prose conditions
- The document always ends with `<self-verification>` containing `<check>` elements — never a bullet list
