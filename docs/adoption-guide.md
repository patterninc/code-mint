# Adoption Guide

This guide explains how to apply code-mint to a target repository in a way that stays understandable over a long, multi-session onboarding effort.

The key idea: track progress by proven outcomes, not by how many procedural steps have been completed.

## The Recommended Path

1. Copy code-mint into the target repository.
2. Run onboarding in assessment-first mode.
3. Record progress in `docs/onboarding-checklist.md`.
4. Prove one outcome at a time with explicit evidence.
5. Split the work into small PRs instead of one giant transformation.

## Setup

Copy the onboarding bundle into the target repo using the **Quick Start** in the root `README.md` (`TARGET_REPO` / `TARGET_SCOPE`, `git clone` of this library, `cp` of skills and core docs, `.agents/code-mint-status.json`, `.gitignore` for reports). If the target repository already has `.agents/`, `AGENTS.md`, rules, or customized skills, merge deliberately rather than overwriting them.

## What The User Should Understand Immediately

Before asking the user to do much setup work, the onboarding flow should make the [six north-star outcomes](outcomes.md) and their proof criteria obvious. Full definitions are in `docs/outcomes.md`; `docs/onboarding-checklist.md` is the system of record for status and evidence.

## Recommended Operator Prompt

Open the folder that matches your onboarding scope when you can. If you work from the git repository root instead, state the scope in the prompt so paths for `docs/`, `.agents/`, and `AGENTS.md` stay unambiguous. Example:

```text
Use the meta--onboarding skill to onboard this repository for AI-first development, keep docs/onboarding-checklist.md updated as the system of record, summarize the findings, and wait for approval before making changes.
```

The prompt says "wait for approval before making changes" to reinforce assessment-first mode. Phase 1 runs read-only auditors; remediation begins only after the user reviews the baseline.

When onboarding only a package or subdirectory, add a line such as: `Onboarding scope: [path relative to repo root]. Treat paths as relative to that directory. Also read the git repository root README.md and any README.md files along the path from the repo root to that scope for repo-wide setup, deploy, CI, or environment notes that may not exist under the scope.`

## Outcome-Driven Sequence

### Outcome 1: Validate Current State

Start with a read-only baseline:

1. Inspect the repository first.
2. Ask only the minimum scoping questions that the code cannot answer.
3. Run all applicable Phase 1 auditors in parallel.
4. Summarize what is working, blocked, risky, and next to prove.
5. Record the result in `docs/onboarding-checklist.md`.

Typical baseline artifacts:

- `.agents/reports/legibility--auditor-audit.md`
- `.agents/reports/autonomy--test-readiness-auditor-audit.md`
- `.agents/reports/autonomy--env-auditor-audit.md` when applicable
- `.agents/reports/autonomy--runtime-auditor-audit.md` when applicable
- `.agents/reports/autonomy--sre-auditor-audit.md` when applicable
- `.agents/reports/onboarding-summary.md`

### Outcome 2: Navigate

Once the baseline exists, make the codebase legible:

1. Create or improve the root `AGENTS.md`.
2. Add subdirectory `AGENTS.md` files for high-value modules.
3. Capture UX intent, module boundaries, and gotchas.
4. Prove the outcome by having the agent explain where work should happen for a sample task and recording that grounded answer.

### Outcome 3: Self-Test

Next, make verification practical:

1. Identify the smallest relevant test command for a real module or behavior.
2. Improve test targeting, isolation, or utilities as needed.
3. Record the exact command, covered scope, and pass/fail signal in the checklist.

### Outcome 4: Smoke Path

Then make the runtime meaningfully testable:

1. Choose the safest meaningful local or staging-like target.
2. Document dependencies, startup order, env assumptions, and stop conditions.
3. Define one trusted, non-destructive smoke path.
4. Mark the outcome `Proven` only when the evidence includes a concrete success signal.

### Outcome 5: Bug Reproduction

Add a real debugging proof:

1. Start from a real reported issue or representative failure.
2. Turn it into a deterministic failing test or repro recipe another person or agent can rerun.
3. Capture that failing case as the evidence artifact.

### Outcome 6: SRE Investigation

If operational tooling exists, prove the agent can inspect it:

1. confirm CLI and monitoring access
2. gather logs, metrics, traces, or CI evidence
3. produce a ranked hypothesis grounded in that evidence and record the next actions

If no operational tooling exists, mark the outcome `N/A` and record why.

## Interaction Style

During onboarding, prefer this loop:

1. Inspect first.
2. Draft what the codebase already reveals.
3. Ask the user only for missing intent, operational context, or hidden assumptions.
4. Confirm small drafts before writing durable docs.
5. Update the checklist after every meaningful proof.

This keeps the process collaborative without turning it into a long interview.

## PR Boundaries

Use small, reviewable PRs:

1. **Phase 1:** baseline reports and checklist initialization
2. **Phase 2:** `AGENTS.md` and repo-legibility work
3. **Phase 3:** self-test and smoke-path improvements
4. **Phase 4:** reproduction and SRE investigation workflows
5. **Phase 5:** verification refresh and optional activation of ongoing skills

These PR phases align with the playbook phases in the `meta--onboarding` skill (assessment → navigation → self-test/smoke → bug repro/SRE → verify/activate). Use the skill for step-by-step operator detail; use this section when splitting work across pull requests.

## What To Customize

These elements should be tailored to the target repository:

- root and subdirectory `AGENTS.md` files
- project-specific `[CUSTOMIZE]` blocks inside skills
- runtime and smoke-path details
- test commands and priority modules
- infrastructure identifiers, log groups, profiles, and workflow names

These elements are usually best kept stable:

- the outcome-first onboarding structure
- the auditor/creator pairing model
- the reproduce-before-fix discipline
- the progressive disclosure approach for documentation

## Updating From Upstream

To compare local customized skills against the latest code-mint version:

```bash
cd "$TARGET_REPO"
git clone https://github.com/patterninc/code-mint.git .code-mint-update
diff -ru "$TARGET_SCOPE/.agents/skills" .code-mint-update/.agents/skills
```

If you also customized copied onboarding docs under `docs/` (for example `framework.md` or `onboarding-checklist.md`), compare those paths the same way, for example `diff -ru "$TARGET_SCOPE/docs" .code-mint-update/docs`, or selectively re-copy files from `.code-mint-update/docs/` after reviewing the diff.

If you forked code-mint to your own organization, replace the clone URL above with your fork's URL.

Preserve local `[CUSTOMIZE]` values when adopting upstream changes.

## Governance

Designate a Rule Steward or equivalent owner to review changes to:

- `.agents/`
- `.agents/code-mint-status.json`
- `AGENTS.md`
- `SKILL.md`
- onboarding docs and checklist templates

Measure success by outcomes, not just activity:

- can it baseline the current state with explicit evidence?
- can it explain where work should happen for a representative task?
- can it run the smallest relevant automated check and trust the result?
- can it execute a smoke path with a concrete success signal?
- can it turn a reported issue into a deterministic repro?
- can it gather operational evidence into a ranked hypothesis?
