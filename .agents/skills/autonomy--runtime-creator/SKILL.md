---
name: autonomy--runtime-creator
description: Builds the scripts, setup flow, and runtime documentation needed for an agent to install dependencies, start local services, boot the app, and run smoke checks from a clean checkout. Use when a runtime audit report exists and runtime readiness needs improvement. Do not use when no audit report exists (run autonomy--runtime-auditor first) or when the main gap is test coverage depth (use autonomy--test-readiness-creator).
---

# Runtime Creator

Build out a repository's runtime readiness so that an agent can go from a clean checkout to a runnable application or local runtime equivalent. Base all work on `autonomy--runtime-auditor`.

## Prerequisites

Read `.agents/reports/autonomy--runtime-auditor-audit.md`. If no report exists, instruct the user to run `autonomy--runtime-auditor` first.

## Collaboration Rules

Treat this as a guided runtime workshop, not just a scripting pass.

1. Reuse the audit findings before asking new questions.
2. Ask for freeform operational context when the runtime path is ambiguous.
3. Draft the intended setup and smoke path in plain language before changing files.
4. Get explicit approval before scripting anything that touches shared infrastructure, real cloud resources, destructive migrations, or workflows that are hard to reverse.

When needed, ask the developer:

- "What is the smallest smoke path that gives your team real confidence?"
- "Which local substitutions are acceptable compared with production?"
- "Which runtime steps are safe to automate, and which must stay manual?"

For smoke-test patterns and safety boundaries, see [../autonomy--runtime-auditor/references/smoke-test-guide.md](../autonomy--runtime-auditor/references/smoke-test-guide.md). The guide lives with the auditor skill intentionally so runtime audit and remediation share one canonical document (`docs/skill-development.md` documents this cross-skill link pattern).

## Step 1: Extract the Handoff Contract

Treat the audit report as the remediation queue:

1. Read `Top Blockers`, `Human Decisions Needed`, `Safe To Automate`, and `Cold Start Results`.
2. Fix upstream blockers first, such as dependency or env gaps before startup issues.
3. Separate low-risk setup tasks from approval-gated changes.
4. Stop and get explicit approval before changing CI, deployment behavior, shared infrastructure, or anything hard to reverse.

Before editing files, summarize the proposed runtime flow back to the developer:

1. Local dependencies to start
2. Expected startup order
3. Required env path
4. Migration and seed expectations
5. The proposed smoke check
6. Which boundaries remain manual or approval-gated

## Step 2: Improve Dependency and Setup Commands

Address in this order:

1. Specify runtime versions, such as `.node-version`, `.python-version`, `.tool-versions`, or equivalent
2. Ensure lock files exist where applicable
3. Document system dependencies in README or `AGENTS.md`
4. Create a single entry-point setup command if multiple steps are required

Target: a clear `make setup`, `scripts/setup.sh`, or equivalent entry point that handles the safe local setup path.

## Step 3: Improve Local Service Startup

If local dependencies are missing or unclear:

1. Add or improve Docker Compose, dev container, local scripts, or equivalent runtime helpers
2. Document each dependency's startup path
3. Add health checks or readiness commands where possible
4. Document migration and seed commands when they are required for meaningful runtime verification

If the repository supports multiple local runtime shapes, document the recommended default first and treat other paths as optional variants.

## Step 4: Improve Runtime Visibility

If the audit found infrastructure visibility gaps:

1. Document where IaC lives, whether in this repo or a companion repo
2. Add read-only inspection guidance for companion repos
3. Record when a VCS host CLI (e.g., `gh` for GitHub, `glab` for GitLab) is sufficient for inspection and when a clone is actually needed
4. Clearly separate "inspectable now" from "requires human cloud action"

Do not assume cloud provisioning must be automated in order for runtime readiness to be acceptable.

## Step 5: Improve Application Startup and Smoke Checks

Create or improve:

1. The documented app startup command
2. The documented local URL or access point
3. A health check or smoke verification path with explicit prerequisites
4. Browser-based or screenshot-based smoke steps if that is the normal way to validate the app
5. Expected success evidence, such as HTTP 200, a rendered page, a log line, or a screenshot artifact
6. "Do not run unless" notes for any step with side effects or shared-environment risk

Target: the agent can start the app and perform a basic confidence check without improvising.

Prefer the safest meaningful smoke path:

- API service: one health check plus one read-only request
- Web app: one page load or browser-based verification of a core screen
- Worker/service daemon: process startup plus queue, scheduler, or consumer readiness signal
- Multi-service application: dependency readiness followed by one app-level confidence check

If authentication is required, document the safest local path, such as a dev-only seed user or mock auth mode. If no safe path exists, document that boundary clearly instead of inventing one.

## Step 6: Verify End-to-End Runtime Flow

Run the safe local runtime sequence:

1. Install dependencies
2. Load or verify environment configuration
3. Start local dependencies
4. Run migrations if safe and required
5. Start the application
6. Perform the documented smoke check

Stop and ask for approval before continuing if a step would:

- modify shared data or infrastructure
- send external notifications or trigger third-party integrations
- require real cloud credentials not intended for agent use
- run migrations or seeds whose safety is unclear

If any step still depends on human approval or real cloud setup, document that boundary clearly instead of pretending the path is fully autonomous.

## Step 7: Archive

1. Archive the audit report to `.agents/reports/completed/autonomy--runtime-auditor-audit-{YYYY-MM-DD}.md`
2. Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `smoke_path` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
