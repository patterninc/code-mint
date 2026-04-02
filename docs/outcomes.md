# North-Star Outcomes

Code-mint is most useful when the user can see a small number of concrete capabilities getting unlocked over time. These outcomes are the public promise of the onboarding flow.

## The Outcomes

| Outcome | What It Means | Proof Of Completion | Primary Skills |
|---|---|---|---|
| `Validate Current State` | The agent can assess the repo as it exists today instead of guessing. | All applicable baseline audit outputs are recorded, and a concise summary captures what is working, blocked, risky, and next to prove. | `meta--onboarding`, applicable auditor skills |
| `Navigate` | The agent can explain the repo structure, identify the right module for a sample task, and justify where work should happen. | Durable in-repo guidance plus a grounded repo explanation or sample task walkthrough that shows the map is actually usable. | `legibility--auditor`, `legibility--enhancer` |
| `Self-Test` | The agent can run the smallest relevant automated check for a real module or behavior and trust the result. | An exact targeted test command or test target, the scope it covers, and a trustworthy pass/fail signal are captured in the checklist. | `autonomy--test-readiness-auditor`, `autonomy--test-readiness-creator` |
| `Smoke Path` | The agent can execute one safe, non-destructive runtime confidence check. | A documented smoke path with prerequisites, exact steps, stop conditions, and a concrete success signal such as a response, log line, or health check. | `autonomy--env-auditor`, `autonomy--env-creator`, `autonomy--runtime-auditor`, `autonomy--runtime-creator` |
| `Bug Reproduction` | The agent can turn a real reported issue into a reproducible failing case before proposing a fix. | A failing test, script, or deterministic reproduction recipe tied to the reported issue and runnable by another person or agent. | `autonomy--sre-agent`, `autonomy--test-readiness-*` |
| `SRE Investigation` | The agent can inspect logs, metrics, traces, CI, or infra context and produce a ranked hypothesis. | A short investigation log captures the evidence sources used, observed signals, ranked hypothesis, and next actions, or records why the outcome is `N/A`. | `autonomy--sre-auditor`, `autonomy--sre-agent` |

## Why These Outcomes

These outcomes map to three pillars:

1. **Legibility** — durable guidance so an agent can navigate intentionally.
2. **Autonomy** — trustworthy tests, safe runtime checks, and where applicable operational tooling (see `docs/framework.md` for what “autonomy” does and does not promise).
3. **Clarity** — after the core outcomes are in place, collaborative plans and tickets precise enough to execute.

`clarity--ticket-writer` should be activated after the core onboarding outcomes are in place. Once the repo is legible and verifiable, better ticket writing compounds the rest of the system.

## Evidence Model

Every outcome should be tracked with the same evidence fields:

- `Status`: `Not Started`, `Blocked`, `In Progress`, `Proven`, or `N/A`
- `Evidence`: the artifact that proves the outcome, such as a report path, `AGENTS.md`, test name, screenshot, prompt transcript, or investigation note
- `Exact Check`: the command, prompt, or scenario that produced the evidence
- `Last Validated`: when the evidence was last refreshed
- `Notes`: what is still manual, flaky, blocked, approval-gated, or intentionally marked `N/A`

Use `docs/onboarding-checklist.md` as the canonical template for that evidence. `.agents/code-mint-status.json` mirrors outcome statuses as a machine-readable index for cross-repo scanning. For an optional skill-by-skill matrix aligned to outcomes, see [`docs/skills-status.md`](skills-status.md).

## What "Done" Looks Like

An onboarding effort is working when the user can point to a short checklist and say:

- the current state has been baselined with recorded audit evidence
- the repo map can answer where work should happen for a sample task
- the agent can run the smallest relevant automated check and trust the result
- the agent can perform one trusted smoke check with a concrete success signal
- a reported issue can be turned into a deterministic failing repro
- operational evidence can be gathered into a ranked investigation hypothesis

That is the story the rest of the repository should reinforce.
