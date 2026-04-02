---
name: autonomy--sre-agent
description: Diagnose production issues by querying logs, metrics, and traces from observability, cloud, and CI/CD CLIs. Use when investigating errors, debugging production issues, checking service health, reviewing CI/CD status, checking recent deployments, reproducing a reported bug with evidence, or when the user mentions logs, metrics, monitoring dashboards, container services, CI pipelines, or production incidents. Do not use when the issue is a known feature request or cosmetic change (use clarity--ticket-writer) or when auditing whether tools are accessible (use autonomy--sre-auditor).
---

# SRE Agent

Investigate production issues using observability, cloud, and CI/CD CLIs. Follow the investigation workflow in order. Do not skip to a fix before confirming the root cause.

This skill ships with **Datadog Pup, AWS CLI, and GitHub CLI** as a worked example. These are the tools the skill was validated against, and they serve as a complete reference implementation. If your project uses different tooling (e.g., Grafana/Loki, GCP Cloud Logging, Azure Monitor, GitLab CI, Jenkins), replace the tool-specific command sections while preserving the investigation workflow structure, hypothesis discipline, and `[CUSTOMIZE]` pattern.

This skill also serves as an onboarding proof tool. If the team wants to prove `Bug Reproduction` or `SRE Investigation`, use this workflow to gather evidence and record the outcome in `docs/onboarding-checklist.md`.

## Prerequisites

Ensure these are available before running queries. If any fail, stop and tell the user what's missing. If multiple tools are unavailable, advise running `autonomy--sre-auditor` to diagnose access issues.

Also check whether the project-local `[CUSTOMIZE]` sections in this skill and `references/recipes.md` have been filled in for the current repository. If key identifiers such as service names, log groups, AWS profiles, repo owner/name, or deploy workflow are still placeholders, stop and ask the user to complete the onboarding customization step first rather than guessing.

### Datadog Pup CLI

Pup is a Datadog CLI with 200+ commands across 33+ products. For installation, updates, and full API coverage, fetch the latest README: https://github.com/datadog-labs/pup

```bash
brew tap datadog-labs/pack && brew install datadog-labs/pack/pup
# Update: brew update && brew upgrade datadog-labs/pack/pup
```

**Auth** — Preferred: `pup auth login` (OAuth2, browser-based). Fallback: `DD_API_KEY` + `DD_APP_KEY` env vars (some commands like `logs search` require API key auth and don't support OAuth2). Check status: `pup auth status`.

**App Key Setup** — Some commands require an Application Key (`DD_APP_KEY`). Create one at **Organization Settings -> Application Keys** in Datadog with these recommended read-only scopes:

| Category | Scopes |
|---|---|
| Logs | `logs_read_data`, `logs_read_config`, `logs_live_tail` |
| Metrics | `metrics_read`, `timeseries_query` |
| APM | `apm_read`, `apm_api_catalog_read`, `apm_service_catalog_read` |
| Monitors | `monitors_read` |
| Dashboards | `dashboards_read`, `notebooks_read` |
| Events | `events_read` |
| SLOs | `slos_read` |
| Databases | `dbm_read` |
| Debugging | `debugger_read`, `continuous_profiler_read`, `error_tracking_read` |

### AWS CLI

```bash
aws sts get-caller-identity --profile [CUSTOMIZE: profile name]
```

Should succeed and return account ID, user ARN, and user ID. [CUSTOMIZE: Document your AWS profile names and which environments they map to.]

### GitHub CLI

```bash
gh auth status
```

Should succeed. [CUSTOMIZE: Document your repository and whether to use `--repo owner/repo` or run from the repo directory.]

## Onboarding Proof Mode

When this skill is used during onboarding rather than a live incident, the goal is to produce one of these proofs:

1. **Bug Reproduction** - turn a reported issue into a deterministic failing test, script, or repro recipe
2. **SRE Investigation** - gather operational evidence and produce a ranked hypothesis with clear next actions, or record why the outcome is `N/A`

Record the evidence in `docs/onboarding-checklist.md` with:

- the exact bug report, symptom, or scenario investigated
- the commands or queries used
- the failing test or repro recipe, when one exists
- the observed signals, ranked hypotheses, and next safe actions

Do not mark either outcome as complete until that evidence exists in-repo.

## Discovering Commands

Use the CLIs themselves — don't memorize commands. Drill down progressively:

```bash
pup --help              # Top-level command groups
pup logs --help         # Subcommands for a group
pup logs search --help  # Flags for a specific command
aws ecs help            # AWS uses `help` without --
gh run --help           # Same pattern as pup
```

## Infrastructure Reference

[CUSTOMIZE: Replace this section with your project's actual infrastructure. These values are intended to be filled collaboratively during onboarding and preserved as project-local configuration.]

### Infrastructure-as-Code Repository

Check the root AGENTS.md "Related Repositories" section for the IaC companion repo (often `{project}-tf`). If you need to understand resource configuration, networking, IAM policies, or debug infrastructure-level issues, clone it:

```bash
git clone [CLONE_URL from AGENTS.md Related Repositories] /tmp/{repo-name}
```

[CUSTOMIZE: e.g., IaC repo: `myapp-tf` — contains Terraform modules for ECS, RDS, ElastiCache, ALB, IAM, and VPC configuration.]

### Services

| Component | Identifier | Scaling |
|---|---|---|
| [CUSTOMIZE: e.g., API (prod)] | [CUSTOMIZE: e.g., prod-myapp-api] | [CUSTOMIZE: e.g., 2-5 tasks] |
| [CUSTOMIZE: e.g., Workers (prod)] | [CUSTOMIZE: e.g., prod-myapp-workers] | [CUSTOMIZE: e.g., 1-16 tasks] |

### Database

[CUSTOMIZE: e.g., Aurora PostgreSQL 16 (Serverless v2). Cluster: `myapp-prod`.]

### Caching / Queues

[CUSTOMIZE: e.g., ElastiCache Redis cluster: `myapp-redis-cluster`.]

### Datadog Tags

| Tag | Value |
|---|---|
| `service` | [CUSTOMIZE: e.g., `myapp`] |
| `env` | [CUSTOMIZE: e.g., `dev` / `prod`] |

### CloudWatch Log Groups

[CUSTOMIZE: e.g., `/ecs/prod-myapp-api`, `/ecs/prod-myapp-workers`.]

### Domain Names

[CUSTOMIZE: e.g., Dev: `dev-myapp.example.com`. Prod: `myapp.example.com`.]

## Datadog Pup CLI

### Log Queries

```bash
pup logs search --query "service:[SERVICE] status:error" --from "1h"
pup logs search --query "service:[SERVICE] @http.status_code:500" --from "30m"
pup logs search --query "service:[SERVICE] @dd.trace_id:<TRACE_ID>"
```

Time ranges: `15m`, `1h`, `4h`, `1d`, `1w`, or ISO timestamps. Use `--limit N` and `--json` as needed.

### Metrics

```bash
pup metrics query --query "avg:ecs.fargate.cpu.percent{service:[SERVICE],env:production}" --from "1h"
pup metrics query --query "sum:trace.[FRAMEWORK].request.errors{service:[SERVICE]}.as_count()" --from "1h"
pup metrics query --query "avg:trace.[FRAMEWORK].request.duration.by.resource_service.95p{service:[SERVICE]}" --from "1h"
```

### Monitors, Dashboards, SLOs

Dashboards and SLOs don't support server-side filtering — pipe through `jq`:

```bash
pup monitors list --tags="service:[SERVICE]"
pup dashboards list | jq '[.data.dashboards[] | select(.title | test("[SERVICE]";"i")) | {id, title}]'
pup slos list | jq '[.data.data[] | select(.name | test("[SERVICE]";"i")) | {id, name}]'
```

Use `pup dashboards get <id>` to extract metric queries powering a dashboard, then run them with `pup metrics query`.

### Traces (APM)

```bash
pup traces search --query "service:[SERVICE] status:error" --from "1h"
pup traces search --query "service:[SERVICE] resource_name:POST_/api/v1/endpoint" --from "1h"
```

## AWS CLI

Primarily used for ECS task health and CloudWatch log fallback.

```bash
# Service status
aws ecs describe-services --cluster [CLUSTER] --services [SERVICE] \
  --profile [PROFILE] --region [REGION] \
  --query 'services[0].{status:status,running:runningCount,desired:desiredCount}'

# Recently stopped tasks
aws ecs list-tasks --cluster [CLUSTER] --desired-status STOPPED \
  --profile [PROFILE] --region [REGION]

# Why a task stopped
aws ecs describe-tasks --cluster [CLUSTER] --tasks <TASK_ARN> \
  --profile [PROFILE] --region [REGION] \
  --query 'tasks[0].{stoppedReason:stoppedReason,stopCode:stopCode,containers:containers[*].{name:name,exitCode:exitCode,reason:reason}}'

# CloudWatch log search (fallback when Datadog is unavailable)
aws logs filter-log-events --log-group-name [LOG_GROUP] \
  --filter-pattern "ERROR" --start-time $(date -v-1H +%s000) \ # macOS; on GNU/Linux use: $(date -d '1 hour ago' +%s000)
  --profile [PROFILE] --region [REGION]
```

## GitHub CLI

Primarily used for CI status, deployment correlation, and commit history.

```bash
gh run list --limit 10                                    # Recent CI runs
gh run list --workflow [DEPLOY_WORKFLOW] --limit 5        # Recent deployments
gh run view <RUN_ID> --log-failed                         # Failed run logs
gh run view --job <JOB_ID> --log-failed                   # Failed job logs

# Recent commits (correlate with errors)
gh api repos/[OWNER]/[REPO]/commits \
  --jq '.[:10] | .[] | {sha: .sha[:8], message: .commit.message, date: .commit.author.date}'

# Compare two deployments
gh api repos/[OWNER]/[REPO]/compare/COMMIT1...COMMIT2 \
  --jq '.files[] | {filename, status, changes}'
```

## Investigation Workflow

Follow these steps in order:

1. **Check error rate** — `pup metrics query` for error counts and trends. Determine if intermittent or persistent before diving into logs.
2. **Search logs** — `pup logs search` for the error message, status code, or endpoint. Get stack traces.
3. **Correlate with traces** — Use `dd.trace_id` from logs to find full APM traces.
4. **Check service health** — `aws ecs describe-services` for task counts and deployment state.
5. **Correlate with deployments** — `gh run list --workflow [DEPLOY_WORKFLOW]` to find recent deploys around the time errors started.
6. **Find the code** — Use the stack trace and `gh api` commit comparison to identify the culprit change.

### Hypothesis-Driven Analysis

If the workflow above doesn't immediately surface the cause, form ranked hypotheses:

1. List 2-3 candidate root causes, ordered by likelihood.
2. For each hypothesis, identify what evidence would confirm or refute it.
3. Prioritize hypotheses that explain ALL symptoms, not just some.

**Common root cause categories:**
- Deployment regression (new code introduced the bug)
- Infrastructure (resource limits, OOM kills, network partitions)
- External dependency (upstream API errors, rate limiting, timeouts)
- Data (unexpected input, state mutation, migration side effects)
- Configuration (environment variable change, feature flag, scaling policy)

### Test Before Fix

Write a failing test case that reproduces the bug BEFORE proposing any fix. If you cannot reproduce the bug in a test, you do not understand it well enough to fix it.

For onboarding proof, the failing test or deterministic repro recipe is itself the evidence artifact. Capture it in `docs/onboarding-checklist.md` and update `.agents/code-mint-status.json` with the current `bug_reproduction` and `sre_investigation` outcome statuses and dates before moving on.

## Fix and Confirm

1. Search for existing utilities or patterns in the codebase that already solve the problem. Extend existing code rather than duplicating.
2. Implement the minimal fix that addresses the confirmed root cause.
3. Run the failing test — it must now pass.
4. Run the full test suite for the affected module — no regressions.
5. Review your own changes: Does this fix introduce new edge cases? Does it respect the project's dependency direction?
6. If the fix reveals an undocumented assumption, add it to the relevant AGENTS.md under "Gotchas."

## Tips

- Start with your observability platform's logs — they typically have the richest context (trace IDs, tags, structured fields). In the reference implementation, that is Datadog.
- Use your cloud provider's native log service as a fallback if the primary observability tool is unavailable. In the reference implementation, that is CloudWatch.
- Always check if an error is intermittent vs. persistent using metrics before diving into logs.
- Use your CI CLI after pushing to verify the pipeline passes. In the reference implementation, that is `gh run list`.
- See [references/recipes.md](references/recipes.md) for pre-built investigation patterns for common scenarios.
- Teams using a different observability, cloud, or CI/CD stack should replace the tool-specific sections above while keeping the investigation workflow, hypothesis model, and `[CUSTOMIZE]` pattern intact.
