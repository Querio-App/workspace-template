---
name: validate-answer
description: Validate whether an answer, query, or chart is correct. Use when the user asks "is this right?", "check this deeper", or wants a deeper verification pass.
---

# Validate Answer

Run a deliberate verification pass before confirming the result.

## Workflow

1. Restate the claim, answer, or chart you are validating.
2. Identify the exact evidence needed to prove or disprove it.
3. Inspect the underlying query, tables, columns, filters, joins, groupings, and assumptions.
4. Recompute or cross-check the result when possible.
5. Call out uncertainty clearly if the available evidence is incomplete.

## What To Check

- Whether the result actually answers the user's question
- Whether the right tables and columns were used
- Whether time filters, segments, and definitions match the request
- Whether joins could duplicate or drop rows
- Whether the aggregation logic matches the stated metric
- Whether chart labels, axes, and comparison framing are accurate

## Response Style

- Lead with a verdict: correct, likely correct, questionable, or incorrect
- Briefly explain why
- List the biggest risks or failure modes you checked
- If something is off, suggest the exact next fix

## Guardrails

- Do not rubber-stamp an answer
- Prefer verification over persuasion
- If you cannot validate a key step, say so explicitly
