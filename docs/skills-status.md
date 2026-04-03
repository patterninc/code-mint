# Skills Status Compatibility Tracker

**When to use this file:** Open it when you need a **skill-by-skill** view—for example to see which outcome a skill supports, to record when an auditor last ran, or to align operational work with the harness. **Do not** use this file as the primary onboarding progress tracker; it does not replace the six outcome rows in `docs/onboarding-checklist.md`. Track capability proofs (what is proven with evidence) only in the checklist. Use this matrix as a secondary map, not a duplicate checklist.

`docs/onboarding-checklist.md` remains the canonical system of record for onboarding progress.

## Status Key

| Status | Meaning |
|---|---|
| `Not Started` | The skill has not been used yet. |
| `Audited` | The auditor has run and produced findings. |
| `In Progress` | Remediation or collaborative work is underway. |
| `Complete` | The skill's current work is complete for now. |
| `Needs Refresh` | The repo changed enough that the prior result may be stale. |
| `N/A` | The skill does not apply to this repository. |

This status vocabulary is intentionally different from `docs/onboarding-checklist.md`. The checklist tracks **outcome-level** progress (`Proven`, `Blocked`), while this file tracks **skill-level** activity (`Audited`, `Complete`, `Needs Refresh`). The two files serve different purposes and should not be forced into a single vocabulary.

## Skill-To-Outcome Map

| Skill | Category | Supports Outcome | Status | Last Run | Notes |
|---|---|---|---|---|---|
| `meta--onboarding` | Meta | `Validate Current State` | Not Started | — | Records the baseline audits and the summary of what is working, blocked, risky, and next to prove. |
| `legibility--auditor` | Legibility | `Navigate` | Not Started | — | Assesses repo navigability and identifies where durable guidance is missing. |
| `legibility--enhancer` | Legibility | `Navigate` | Not Started | — | Creates durable `AGENTS.md` guidance that can answer sample task-routing questions. |
| `autonomy--env-auditor` | Autonomy | `Smoke Path` | Not Started | — | Confirms env loading is safe and understandable before runtime proof. |
| `autonomy--env-creator` | Autonomy | `Smoke Path` | Not Started | — | Remediates env gaps that block a documented smoke path. |
| `autonomy--runtime-auditor` | Autonomy | `Smoke Path` | Not Started | — | Assesses install, startup, and smoke-path readiness, including success signals. |
| `autonomy--runtime-creator` | Autonomy | `Smoke Path` | Not Started | — | Builds the runtime path and smoke-check support with clear prerequisites and steps. |
| `autonomy--test-readiness-auditor` | Autonomy | `Self-Test` | Not Started | — | Baselines whether the repo has a smallest relevant automated check the agent can trust. |
| `autonomy--test-readiness-creator` | Autonomy | `Self-Test` and `Bug Reproduction` | Not Started | — | Improves targeted tests and regression coverage so exact checks and repros are runnable. |
| `autonomy--sre-auditor` | Autonomy | `SRE Investigation` | Not Started | — | Checks whether operational tooling is reachable before investigation proof begins. |
| `autonomy--sre-agent` | Autonomy | `Bug Reproduction` and `SRE Investigation` | Not Started | — | Produces deterministic repros and evidence-backed investigation notes with ranked hypotheses and next actions. |
| `clarity--ticket-writer` | Clarity | Post-onboarding activation | Not Started | — | Refines vague requests once the repo is legible and verifiable. |
