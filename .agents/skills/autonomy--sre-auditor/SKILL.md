---
name: autonomy--sre-auditor
description: Audit whether an agent has working access to the SRE CLIs the project uses — version control host CLI, cloud provider CLI, and observability CLI. Tests each connection and reports access gaps. The steps below use GitHub CLI, AWS CLI, and Datadog Pup as a worked example; adapt for your project's actual tooling. Use when setting up a new agent environment, troubleshooting tool access, or as part of an initial harness assessment. Do not use when debugging a specific production issue (use autonomy--sre-agent) or when auditing general runtime readiness (use autonomy--runtime-auditor).
---

# SRE Auditor

Verify that an agent has authenticated, working access to the CLI tools required for SRE operations. Each tool is tested with a non-destructive read-only command.

The steps below use **GitHub CLI, AWS CLI, and Datadog Pup** as a reference profile. If your project uses a different VCS host (e.g., GitLab, Azure DevOps), cloud provider (e.g., GCP, Azure), or observability platform (e.g., Grafana, New Relic, Prometheus), substitute the equivalent CLI and read-only verification commands while following the same audit structure.

## Step 1: Audit GitHub CLI Access

### Check Installation
```bash
gh --version
```
If not installed, record as a Critical finding.

### Check Authentication
```bash
gh auth status
```
Expected: Shows authenticated user and token scopes.

### Check Repository Access
```bash
gh repo view --json name,owner
```
Expected: Returns the current repository's metadata.

### Check Required Capabilities
- [ ] Can list PRs: `gh pr list --limit 1`
- [ ] Can list issues: `gh issue list --limit 1`
- [ ] Can view CI status: `gh run list --limit 1`
- [ ] Can create branches: `gh api repos/{owner}/{repo}/branches` (read access confirms API works)

Record the authentication method (token, OAuth, SSH) and token scopes.

## Step 2: Audit AWS CLI Access

### Check Installation
```bash
aws --version
```
If not installed, record as a Critical finding.

### Check Authentication
```bash
aws sts get-caller-identity
```
Expected: Returns account ID, user ARN, and user ID.

### Check Required Service Access

Test non-destructive read commands for each service the project uses:

- [ ] **ECS (if applicable):** `aws ecs list-clusters`
- [ ] **CloudWatch Logs:** `aws logs describe-log-groups --limit 1`
- [ ] **S3 (if applicable):** `aws s3 ls` (list buckets)
- [ ] **RDS (if applicable):** `aws rds describe-db-instances`
- [ ] **SSM Parameters (if applicable):** `aws ssm describe-parameters --max-results 1`
- [ ] **Lambda (if applicable):** `aws lambda list-functions --max-items 1`

Record the IAM identity, account, region, and which services are accessible.

If your project uses a different cloud provider (e.g., GCP, Azure), replace the commands above with equivalent read-only checks such as `gcloud auth list` / `gcloud projects describe` or `az account show` / `az group list`.

### Check Least Privilege

Note the permission scope. An SRE agent should have:
- **Read access** to logs, metrics, service status, and infrastructure state
- **Limited write access** to restart services, scale tasks, and update configurations
- **No access** to delete resources, modify IAM, or change billing

## Step 3: Audit Monitoring Tool Access

### Datadog Pup (or Equivalent)

Check installation:
```bash
pup --version
```

If Datadog Pup is the monitoring tool:
- [ ] `pup` CLI is installed
- [ ] API key is configured (`DD_API_KEY` environment variable or config file)
- [ ] App key is configured (`DD_APP_KEY` environment variable or config file)
- [ ] Can query metrics: `pup metric query --query "avg:system.cpu.user{*}" --from 1h`
- [ ] Can query logs: `pup log list --query "status:error" --from 1h --limit 1`

If another monitoring tool is used (Grafana, New Relic, Prometheus, etc.):
- [ ] CLI or API client is installed
- [ ] Authentication is configured
- [ ] Can query metrics programmatically
- [ ] Can query logs programmatically
- [ ] Output is machine-readable (JSON, structured text)

Record the tool name, version, authentication method, and accessible data sources.

## Step 4: Audit Telemetry Output Format

For each monitoring tool, verify that output is machine-readable:

- [ ] Log output includes structured fields (timestamp, level, service, message, trace ID)
- [ ] Metrics are queryable by service, time range, and tag
- [ ] Traces connect requests across services
- [ ] Error context includes stack traces, request IDs, and relevant input data

Machine-readable telemetry is required for `autonomy--sre-agent` to operate effectively.

## Output

Ensure the report directory exists: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Write the report to `.agents/reports/autonomy--sre-auditor-audit.md`:

```
# SRE Tooling Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Status:** [Pass / Partial / Fail]

## Summary
| Tool | Installed | Authenticated | Functional | Notes |
|---|---|---|---|---|
| GitHub CLI (`gh`) | [Yes/No] | [Yes/No] | [Yes/No] | [details] |
| AWS CLI (`aws`) | [Yes/No] | [Yes/No] | [Yes/No] | [details] |
| Monitoring (`pup`/other) | [Yes/No] | [Yes/No] | [Yes/No] | [details] |

## Top Blockers
[Highest-severity access gaps preventing SRE investigation]

## Human Decisions Needed
[Authentication ownership, missing API keys, required scopes, or approval for broader access]

## Safe To Automate
[Install checks, read-only verification commands, or re-run steps that are safe without additional approval]

## GitHub CLI Details
- Version: [X.Y.Z]
- Auth Method: [token/OAuth/SSH]
- Token Scopes: [list]
- Repository Access: [Yes/No]

## AWS CLI Details
- Version: [X.Y.Z]
- Identity: [ARN]
- Account: [ID]
- Region: [region]
- Accessible Services: [list]

## Monitoring Details
- Tool: [name]
- Version: [X.Y.Z]
- Auth Method: [API key/token/etc.]
- Queryable: [Yes/No]
- Machine-Readable Output: [Yes/No]

## Findings

### [Finding Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [what exists]
- **Required State:** [what should exist]
- **Recommended Action:** [specific step]
- **Next Skill / Step:** [e.g., Install/authenticate tooling manually, then re-run `autonomy--sre-auditor`; once ready, use `autonomy--sre-agent`]

## Next Steps
Address findings to enable `autonomy--sre-agent` to operate effectively. If tooling or auth is missing, complete the manual install/authentication work first, then re-run `autonomy--sre-auditor`.
```

After writing the report, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `sre_investigation` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
