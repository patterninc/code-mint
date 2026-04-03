# Outcome Checklist

Use this file as the canonical progress tracker during onboarding. The goal is not to mark steps complete; it is to prove that the repository can support a small set of agent capabilities with real evidence.

This file is intended to be copied into the target repository and updated there throughout onboarding.

## How To Use

1. Start with `Validate Current State` so the repository has a baseline.
2. Work toward one proof at a time. Do not try to unlock every outcome at once.
3. Record evidence each time an outcome changes state.
4. Keep `.agents/code-mint-status.json` in sync with this checklist. It is the machine-readable index; this file is the detailed evidence record.
5. Revisit the file when the user comes back later. It should be obvious what is done, blocked, or still unproven.

## Status Key

| Status | Meaning |
|---|---|
| `Not Started` | No meaningful work has been done on this outcome yet. |
| `Blocked` | The next proof is known, but something external or risky is preventing it. |
| `In Progress` | Work is underway, but the outcome is not yet proven with evidence. |
| `Proven` | The outcome has been demonstrated and the proof is captured below. |
| `N/A` | The outcome does not apply to this repository. |

## Outcome Tracker

| Outcome | Status | Last Validated |
|---|---|---|
| `Validate Current State` | Not Started | — |
| `Navigate` | Not Started | — |
| `Self-Test` | Not Started | — |
| `Smoke Path` | Not Started | — |
| `Bug Reproduction` | Not Started | — |
| `SRE Investigation` | Not Started | — |

## Outcome Details

Copy one section per outcome as it moves forward. Each section is the detailed evidence record for that outcome.

### Validate Current State

The agent can assess the repo before making changes.

- **Status:** Not Started
- **Date:** —
- **Evidence:** applicable audit reports plus `.agents/reports/onboarding-summary.md`
- **Exact Check:** Run all applicable Phase 1 auditors and summarize what is working, blocked, risky, and next to prove.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** Establish this first.

### Navigate

The agent can explain where to work and why modules exist.

- **Status:** Not Started
- **Date:** —
- **Evidence:** `AGENTS.md`, subdirectory `AGENTS.md`, sample repo explanation
- **Exact Check:** Ask the agent to explain the repo map and target module for a sample task, then record the grounded answer.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** Favor durable repo-local docs over chat-only explanations.

### Self-Test

The agent can run the smallest relevant automated checks and trust the result.

- **Status:** Not Started
- **Date:** —
- **Evidence:** exact test command, test file, CI reference
- **Exact Check:** Run one targeted test path for a real module or behavior change and capture the pass/fail signal.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** Record what is still manual or flaky.

### Smoke Path

The agent can prove the runtime is meaningfully usable.

- **Status:** Not Started
- **Date:** —
- **Evidence:** runtime guide, screenshot, log excerpt, curl output
- **Exact Check:** Execute one trusted, non-destructive smoke path with documented prerequisites, steps, stop conditions, and success signal.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** Smallest safe check wins.

### Bug Reproduction

The agent can reproduce a reported issue before proposing a fix.

- **Status:** Not Started
- **Date:** —
- **Evidence:** failing test, script, or repro recipe linked to the issue
- **Exact Check:** Turn one real bug report into a deterministic failing case another person or agent can rerun.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** The repro is the proof.

### SRE Investigation

The agent can inspect logs, metrics, traces, CI, or infra evidence.

- **Status:** Not Started
- **Date:** —
- **Evidence:** investigation note, query output, linked dashboard/log command
- **Exact Check:** Gather operational evidence, produce a ranked hypothesis, and record the next actions or `N/A` reason.
- **What Passed:** —
- **What Is Still Missing:** —
- **Next Action:** Mark `N/A` only when no operational tooling exists.

## Suggested Evidence Patterns

| Outcome | Good Evidence |
|---|---|
| `Validate Current State` | Audit reports plus a short plain-language summary in `.agents/reports/onboarding-summary.md` covering working, blocked, risky, and next proof |
| `Navigate` | Root and module `AGENTS.md` files plus a sample repo explanation that answers where a representative task should happen |
| `Self-Test` | Exact targeted test command, the module or behavior it verifies, and the pass/fail signal it produced |
| `Smoke Path` | Prerequisites, startup order, smoke command, concrete success signal, and "stop here" safety notes |
| `Bug Reproduction` | Failing test or deterministic repro script tied to a reported issue and rerunnable by another person or agent |
| `SRE Investigation` | Query commands, time range, observed symptoms, ranked hypotheses, and next actions or an `N/A` reason |

## Skill Mapping

| Outcome | Skills To Reach It |
|---|---|
| `Validate Current State` | `meta--onboarding` plus all applicable auditor skills |
| `Navigate` | `legibility--auditor`, `legibility--enhancer` |
| `Self-Test` | `autonomy--test-readiness-auditor`, `autonomy--test-readiness-creator` |
| `Smoke Path` | `autonomy--env-auditor`, `autonomy--env-creator`, `autonomy--runtime-auditor`, `autonomy--runtime-creator` |
| `Bug Reproduction` | `autonomy--sre-agent`, `autonomy--test-readiness-*` |
| `SRE Investigation` | `autonomy--sre-auditor`, `autonomy--sre-agent` |

## Return-Visit Questions

When someone comes back to onboarding later, this file should answer:

- What has been proven already?
- What is the next missing proof?
- Which command, prompt, or scenario should be run next?
- What is blocked on human approval or missing tooling?
