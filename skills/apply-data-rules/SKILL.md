---
name: apply-data-rules
description: Audit an analysis against workspace data rules, vetted queries, and trusted patterns. Use when checking whether the chosen tables, columns, joins, or logic follow the approved rules.
---

# Apply Data Rules

Perform a strict governance and correctness pass before finalising the answer.

## Workflow

1. Identify the exact question being answered.
2. List the tables, columns, joins, filters, and calculations used or proposed.
3. Inspect vetted queries, golden queries, and workspace knowledge for approved logic.
4. Compare the proposed analysis against those rules and patterns.
5. Flag deviations, risks, and anything unsupported by the trusted sources.

## What To Verify

- Whether the selected tables are the approved source of truth
- Whether the selected columns match the intended business definitions
- Whether joins follow known relationships and safe join keys
- Whether filters and date logic match known rules
- Whether there is an existing vetted query that should be preferred
- Whether metric logic is consistent with prior trusted usage

## Response Style

- State whether the analysis is compliant, partially compliant, non-compliant or simply lacks workspace rules
- Cite the relevant vetted query or rule when available
- Explain any mismatch plainly
- Recommend the corrected query path if needed

## Guardrails

- Prefer trusted workspace rules over improvisation
- If the knowledge base is incomplete, say exactly what could not be verified
