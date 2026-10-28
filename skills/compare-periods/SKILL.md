---
name: compare-periods
description: Compare a result, metric, or chart to a prior period. Use when the user wants period-over-period context such as versus last week, last month, last quarter, or the same period last year.
---

# Compare Periods

Extend the current analysis with period-over-period context.

## Workflow

1. Identify the current metric, grouping, and timeframe.
2. Determine the most appropriate comparison period.
3. Recreate the same logic for both periods.
4. Compare absolute change, percentage change, and notable pattern shifts.
5. Explain what changed in plain language.

## Comparison Rules

- Match the metric definition exactly across periods
- Keep filters, segments, and grain consistent
- Use an equivalent prior period unless the user specifies otherwise
- Call out seasonality or incomplete-period risks when relevant

## Output

Include:

- the current period
- the comparison period
- absolute delta
- percentage delta
- a brief interpretation of the change

## Guardrails

- Do not compare mismatched grains or partial windows without calling that out
- If the current answer lacks enough detail to reproduce the comparison, say what is missing
