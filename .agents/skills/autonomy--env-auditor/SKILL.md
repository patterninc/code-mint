---
name: autonomy--env-auditor
description: Audit whether a repository's environment variable configuration allows an agent to load all required .env variables from scratch. Use when evaluating a new repository's agent-readiness, when environment loading fails during agent setup, or as part of an initial harness assessment. Do not use when .env files already load correctly and the goal is to add a single new variable, or when creating env loading capability (use autonomy--env-creator).
---

# Environment Auditor

Evaluate whether an agent can configure and load all required environment variables from a cold start — with zero pre-existing `.env` files or manual setup.

## Step 1: Discover Environment Requirements

1. **Find all environment variable references** in the codebase:
   - Search for `process.env.`, `os.environ`, `os.Getenv`, `ENV[`, or equivalent patterns.
   - Check configuration files: `scripts/env.sh`, `.env.example`, `.env.sample`, `docker-compose.yml`, CI config.
   - Check infrastructure files: Terraform, CloudFormation, Kubernetes manifests.

2. **Catalog each variable:**
   - Variable name
   - Where it's referenced (file and line)
   - Whether it has a default/fallback value
   - Whether it's required or optional
   - What it connects to (database, API key, feature flag, etc.)
   - Whether it contains a secret (credentials, tokens, API keys)

## Step 2: Evaluate Environment Loading Mechanism

Check how the project loads environment variables:

- [ ] **Primary setup exists:** Is there a documented command or script (`scripts/env.sh`, `make env`, etc.) that produces local env config from the project's source of truth?
- [ ] **Secret source is documented:** Is the secret system or provisioning flow clear (SSM, Vault, Doppler, 1Password, etc.)?
- [ ] **Setup covers all variables:** Does the primary setup plus local overrides cover ALL required variables from Step 1?
- [ ] **Setup is documented:** Does the README or AGENTS.md explain how to run it?
- [ ] **Prerequisites are documented:** Required CLIs, auth steps, and local tools.
- [ ] **Loading mechanism documented:** Is the tool/library used to load `.env` documented (dotenv, direnv, mise, etc.)?
- [ ] **Template fallback:** If no primary setup exists, is there at least a `.env.example` or `.env.sample` template file?

## Step 3: Test Cold Start Loading

**Caution:** Starting the application may have side effects (database creation, external API calls, email sends). Only attempt startup if you are confident it is safe in the current environment (e.g., local development with no production credentials). If unsure, evaluate readiness without starting the application and note that a live test was not performed.

Attempt to load the environment from scratch:

1. **Simulate a cold start:** If a documented bootstrap command exists, run it. If only a `.env.example` exists, note that the repo relies on a template-based flow and evaluate whether it is sufficient for local setup. Only copy the template to `.env` if that is clearly safe and expected for the project. If safe, attempt to start the application. If not safe, check whether the configuration files parse correctly without starting.
2. **Identify failures:** Which variables cause failures when missing or set to placeholder values?
3. **Categorize blockers:**
   - **Hard blockers:** Application crashes or refuses to start (database connection, required API keys)
   - **Soft blockers:** Application starts but features are degraded (analytics, email, optional integrations)
   - **Non-blockers:** Variables with sensible defaults that work in development

## Step 4: Evaluate Secret Accessibility

- [ ] **Secrets are not hardcoded** in source files, committed `.env` files, or documentation
- [ ] **Secret retrieval is documented:** The process for obtaining each secret is described somewhere
- [ ] **Secret rotation is possible:** Secrets can be rotated without code changes
- [ ] **Development secrets exist:** There are development/test values for secrets that allow local development without production credentials

## Output

Ensure the report directory exists: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Write the report to `.agents/reports/autonomy--env-auditor-audit.md`:

```
# Environment Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Status:** [Pass / Partial / Fail]

## Summary
- Total Variables Found: [N]
- Primary Setup: [Script / Command / Template Only / None]
- Secrets Identified: [N]
- Cold Start Result: [Success / Partial / Failure]

## Variable Inventory
| Variable | Required | Has Default | Secret | Documented | Status |
|---|---|---|---|---|---|
| DATABASE_URL | Yes | No | Yes | No | Missing from primary setup |
| ... | ... | ... | ... | ... | ... |

## Findings

### [Finding Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [what exists]
- **Required State:** [what should exist]
- **Recommended Action:** [specific step]
- **Next Skill / Step:** [e.g., Run `autonomy--env-creator`]

## Cold Start Blockers
[List of variables that prevent application startup]

## Next Steps
Run `autonomy--env-creator` to remediate findings.
```

After writing the report, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `smoke_path` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
