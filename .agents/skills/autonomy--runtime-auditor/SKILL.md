---
name: autonomy--runtime-auditor
description: Audit whether an AI agent can go from a clean checkout to a runnable application or local runtime equivalent, without unsafe provisioning or destructive setup. Use when evaluating development or staging-like runtime readiness, onboarding a new project, or checking whether the agent can install dependencies, start services, and perform smoke checks. Do not use when evaluating test strategy and coverage depth (use autonomy--test-readiness-auditor) or when debugging a live production issue (use autonomy--sre-agent).
---

# Runtime Auditor

Evaluate whether an agent can go from a clean checkout to a safe local runtime or staging-like equivalent, or to the closest safe local simulation of that environment. This skill focuses on install, setup, local services, startup, smoke checks, and infrastructure inspection. It does not require cloud provisioning.

## Collaboration Rules

Do not treat runtime readiness as a pure checklist exercise. Inspect the repository first, then ask only for the operational context the codebase cannot answer confidently.

When runtime facts are unclear, ask the developer for freeform explanation, examples, or a short walkthrough of how they actually get the system running today. Prefer prompts like:

- "Walk me through the safest real path a new developer uses to get this repo running."
- "Which step usually breaks first on a clean machine?"
- "What is the smallest meaningful smoke check your team trusts?"

Capture those answers in the report as evidence, not just as assumptions.

## Step 0: Establish the Runtime Target

Before evaluating details, determine which target is realistic:

1. Safe local runtime
2. Staging-like local simulation
3. Partial local runtime plus read-only infrastructure inspection

If the repository does not make the target obvious, ask the developer which target is intended and why. Use the safest meaningful target available; do not assume full cloud-backed staging is required.

## Runtime Discovery Questions

If the repository cannot answer these confidently, ask the developer and record the answers:

1. Which services must run for a meaningful smoke test?
2. In what order should dependencies and the app start?
3. What readiness signals show that each dependency is actually usable?
4. Which environment values are required, and which can use safe placeholders locally?
5. Are migrations or seed data required before the app can be exercised meaningfully?
6. Does the smoke path require authentication, fixtures, or a pre-existing account?
7. What is the safest smoke path the team trusts today?
8. Which steps are unsafe, shared-environment dependent, or require human approval?

For concrete smoke-test patterns and safety boundaries, see [references/smoke-test-guide.md](references/smoke-test-guide.md).

## Step 1: Evaluate Dependency Management

Check whether the application dependencies can be installed autonomously:

- [ ] Package manager is documented
- [ ] Install command is documented and works from a clean checkout
- [ ] Dependencies are pinned with a committed lock file when applicable
- [ ] System-level dependencies are documented
- [ ] Language or runtime version is specified
- [ ] No undocumented manual install steps remain

## Step 2: Evaluate Environment and Configuration Handoff

Cross-reference `autonomy--env-auditor` findings if that report exists.

- [ ] The runtime depends on a documented env loading path
- [ ] Required local or development-safe values are available
- [ ] The startup flow clearly describes when placeholder values are acceptable
- [ ] Secrets are not assumed to exist only in a cloud secret store when local development uses a different path

## Step 3: Evaluate Infrastructure Visibility

The goal is inspection-readiness, not cloud provisioning.

- [ ] Local runtime dependencies are documented, such as databases, caches, queues, search, or browsers
- [ ] If IaC exists in this repo, its location is documented
- [ ] If IaC lives in a companion repo, that repo is documented in `AGENTS.md` or README
- [ ] Companion repos are inspected before marking infrastructure as undocumented
- [ ] Read-only infrastructure inspection can happen through clone access or VCS host CLI if available (e.g., `gh` for GitHub, `glab` for GitLab)
- [ ] Manual infrastructure-only steps are clearly called out instead of being treated as automation failures

## Step 4: Evaluate Local Service Startup

- [ ] Each local dependency has a startup path, such as Docker Compose, dev container, local install, or hosted sandbox
- [ ] Health checks or simple readiness checks exist for dependencies
- [ ] Database migration commands are documented if startup requires them
- [ ] Seed data or fixture instructions exist when the app cannot be meaningfully exercised without data

## Step 5: Evaluate Application Startup

- [ ] A single documented path exists to start the application or relevant services
- [ ] The startup command is safe in the current environment
- [ ] The expected local URL, port, or access point is documented
- [ ] Smoke verification steps are documented with prerequisites and expected success evidence

## Step 6: Evaluate Runtime Smoke Checks

Only perform safe, local checks. Do not execute destructive operations.

Treat a smoke test as the smallest non-destructive check that gives meaningful confidence the runtime is usable. Prefer this order:

1. Dependency readiness check
2. App startup verification
3. One app-level confidence check

Examples:

- API service: health endpoint plus one read-only or obviously non-destructive request
- Web app: load the local URL and confirm one core screen or element renders
- Worker: verify process startup plus queue, scheduler, or job-consumer readiness
- Multi-service stack: dependency health first, then one end-to-end app check

Stop and mark the step as requiring human input instead of improvising if the only available smoke path would:

- mutate shared data
- trigger billing, email, webhooks, or third-party side effects
- require real production or shared staging credentials
- depend on a login flow or callback path that is undocumented or unsafe to fake
- require migrations or seeds whose safety is unclear

Assess whether the following would succeed:

| Step | Evaluate By |
|---|---|
| Install dependencies | Verify the command exists and appears runnable from a clean checkout |
| Load environment | Verify the documented env path exists or cross-reference the env audit |
| Start local dependencies | Check Docker Compose, setup scripts, or equivalent startup docs |
| Run migrations | Check whether the command exists and whether it is safe for a local or ephemeral database |
| Start app | Only attempt if the prior steps are safe and sufficient |
| Smoke test app | Check for documented smoke path, its prerequisites, its expected success signal, and whether it is safe to execute |
| Inspect infra context | Review IaC or companion repo context without provisioning |

Record which steps would succeed, which are blocked, and which require human decisions. When a smoke path is partial, say exactly what evidence is available today and what missing step prevents higher confidence.

## Output

Ensure the report directories exist: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Write the report to `.agents/reports/autonomy--runtime-auditor-audit.md`:

```markdown
# Runtime Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Status:** [Pass / Partial / Fail]

## Summary
- Agent Can Reach A Runnable Environment: [Yes / Partially / No]
- Local Simulation Available: [Yes / Partially / No]
- Runtime Smoke Path Available: [Yes / Partially / No]

## Top Blockers
[Highest-severity blockers preventing install, startup, or smoke verification]

## Human Decisions Needed
[Cloud topology choices, shared environment ownership, unsafe commands, or approval-gated setup]

## Safe To Automate
[Low-risk setup, startup, or documentation tasks that can proceed immediately]

## Dependency Management
[Assessment with specific gaps]

## Runtime Configuration
[Assessment with specific gaps]

## Infrastructure Visibility
[Assessment with specific gaps]

## Local Services
[Assessment with specific gaps]

## Startup and Smoke Checks
[Assessment with specific gaps]

## Cold Start Results
| Step | Status | Notes |
|---|---|---|
| Install | [Pass/Partial/Fail] | [details] |
| Env Setup | [Pass/Partial/Fail] | [details] |
| Local Dependencies | [Pass/Partial/Fail] | [details] |
| Migrations | [Pass/Partial/Fail] | [details] |
| Start App | [Pass/Partial/Fail] | [details] |
| Smoke Test | [Pass/Partial/Fail] | [details] |
| Infra Inspection | [Pass/Partial/Fail] | [details] |

## Findings

### [Finding Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [what exists]
- **Required State:** [what should exist]
- **Recommended Action:** [specific step]
- **Next Skill / Step:** [for example, run `autonomy--runtime-creator`]

## Next Steps
Run `autonomy--runtime-creator` to remediate findings.
```

After writing the report, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `smoke_path` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
