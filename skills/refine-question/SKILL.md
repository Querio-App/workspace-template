---
name: refine-question
description: Help the user refine a prompt or analysis request by asking targeted follow-up questions. Use when the request is vague, broad, or could benefit from clarification before doing the work.
---

# Refine Question

Help the user turn a rough ask into a crisp, answerable prompt.

## Workflow

1. Identify what is ambiguous, missing, or underspecified.
2. Ask a short set of targeted questions that will materially improve the result.
3. Keep the questions concrete and easy to answer.
4. After the user answers, synthesize the refined request in plain language.

## Questioning Rules

- Ask only the highest-leverage questions
- Prefer 3 to 5 questions, not a long questionnaire
- Focus on goal, metric, timeframe, audience, constraints, and desired output
- If options would help, offer a few concrete choices

## Output

When helpful, provide:

- a tightened version of the user's prompt
- one or two alternative phrasings
- a note about what assumptions would be made if they want to proceed immediately

## Guardrails

- Do not start analysis before clarifying the core ambiguity unless the user explicitly wants a best-effort draft
- Avoid generic questions that do not change the work
