# Rules

This directory is intentionally empty. Rules are project-specific persistent instructions that apply automatically based on file patterns — unlike skills, which activate on demand.

Add your own rules here to enforce coding standards, naming conventions, architectural constraints, and other guardrails that should be injected into every relevant agent prompt.

## What Belongs In Rules

Good candidates for rules:

- naming conventions that should not drift
- dependency direction constraints
- file placement rules
- error handling requirements
- test naming or co-location requirements
- prohibited commands or approval-gated actions

What should stay out of rules:

- long tutorials
- one-time onboarding instructions
- project history
- optional best practices with many exceptions
- large checklists that belong in a skill or doc

## Rule Selection Test

Create a rule only when all of the following are true:

1. The guidance should apply automatically, not only when requested.
2. The rule is stable enough to survive many tasks.
3. The rule can be stated clearly in a small amount of text.
4. Violating the rule would create repeated review churn or architectural drift.

## Example Rule Topics

- `frontend/**`: component naming, state management boundaries, test expectations
- `backend/**`: error envelopes, service boundaries, structured logging
- `infra/**`: approval-gated actions, change review expectations, environment targeting

The core product story for code-mint lives in `README.md`, `docs/outcomes.md`, and `docs/onboarding-checklist.md`. Project-specific rules belong here only after onboarding reveals which constraints are worth injecting automatically.
