# SRE Investigation Recipes

Pre-built investigation patterns for common production scenarios. [CUSTOMIZE] the service names, tags, and infrastructure identifiers for your project.

These recipes use Datadog Pup, AWS ECS, and GitHub CLI as a worked example. Adapt the tool commands for your project's observability, cloud, and CI/CD infrastructure.

---

## HTTP Error Spike

**Symptom:** Users reporting 500s or elevated error rates.

```bash
# Check error trend
pup metrics query --query "sum:trace.[FRAMEWORK].request.errors{service:[SERVICE]}.as_count()" --from "4h"

# Find error logs
pup logs search --query "service:[SERVICE] @http.status_code:>=500" --from "1h" --limit 20

# Get stack trace
pup logs search --query "service:[SERVICE] @http.status_code:500 @error.stack:*" --from "1h" --limit 1 --json
```

---

## Streaming / Connection Errors

**Symptom:** Incomplete responses, connection resets, transfer encoding errors.

```bash
pup logs search --query "service:[SERVICE] (*TransferEncodingError* OR *RemoteProtocolError* OR *ConnectionError*)" --from "4h"
pup logs search --query "service:[SERVICE] status:error (*timeout* OR *timed out* OR *deadline*)" --from "4h"
```

---

## Slow Responses / High Latency

**Symptom:** Slow API responses, user complaints about performance.

```bash
# P95 latency trend
pup metrics query --query "avg:trace.[FRAMEWORK].request.duration.by.resource_service.95p{service:[SERVICE]}" --from "4h"

# Find slow traces (>30s)
pup traces search --query "service:[SERVICE] @duration:>30000000000" --from "1h"

# Check CPU (is the service overloaded?)
pup metrics query --query "avg:ecs.fargate.cpu.percent{service:[SERVICE],env:production}" --from "4h"
```

---

## Rate Limiting

**Symptom:** 429 errors from upstream providers.

```bash
pup logs search --query "service:[SERVICE] (*429* OR *rate limit* OR *RateLimitError* OR *ThrottlingException*)" --from "4h"
```

---

## ECS Task Health

**Symptom:** Service instability, restarts, OOM kills, or deployment issues.

```bash
# Current service state
aws ecs describe-services --cluster [CLUSTER] --services [SERVICE] \
  --profile [PROFILE] --region [REGION] \
  --query 'services[0].{status:status,running:runningCount,desired:desiredCount}'

# Recently stopped tasks
TASKS=$(aws ecs list-tasks --cluster [CLUSTER] --desired-status STOPPED \
  --profile [PROFILE] --region [REGION] --query 'taskArns[:5]' --output text)

for TASK in $TASKS; do
  aws ecs describe-tasks --cluster [CLUSTER] --tasks $TASK \
    --profile [PROFILE] --region [REGION] \
    --query 'tasks[0].{stoppedAt:stoppedAt,stoppedReason:stoppedReason,stopCode:stopCode}'
done

# Check for OOM kills
pup logs search --query "service:[SERVICE] *OOMKilled*" --from "1d"
```

---

## Database Issues

**Symptom:** Slow queries, connection pool exhaustion, timeouts.

```bash
pup logs search --query "service:[SERVICE] (*connection pool* OR *statement_timeout* OR *OperationalError* OR *InterfaceError*)" --from "1h"
pup metrics query --query "avg:postgresql.connections{service:[SERVICE]}" --from "4h"
```

---

## Bad Deployment / Regression

**Symptom:** Errors started around a specific time, suspected deployment caused it.

```bash
# Narrow the error window
pup logs search --query "service:[SERVICE] status:error" --from "4h" --limit 5

# Find recent deployments
gh run list --workflow [DEPLOY_WORKFLOW] --limit 10

# Compare commits between deploys
gh api repos/[OWNER]/[REPO]/compare/<PREV_SHA>...<DEPLOY_SHA> \
  --jq '.commits[] | {sha: .sha[:8], message: .commit.message}'

# See changed files
gh api repos/[OWNER]/[REPO]/compare/<PREV_SHA>...<DEPLOY_SHA> \
  --jq '.files[] | {filename, status, changes}'
```

---

## CI Failure

**Symptom:** CI failing on a branch or after merge.

```bash
gh run list --status failure --limit 5
gh run view <RUN_ID> --log-failed
gh run view --job <JOB_ID> --log-failed
```

---

## Memory / Resource Exhaustion

**Symptom:** Services degrading over time, eventual crashes.

```bash
# Memory trend
pup metrics query --query "avg:ecs.fargate.mem.usage{service:[SERVICE],env:production}" --from "24h"

# CPU trend
pup metrics query --query "avg:ecs.fargate.cpu.percent{service:[SERVICE],env:production}" --from "24h"

# Check for container restarts
aws ecs describe-services --cluster [CLUSTER] --services [SERVICE] \
  --profile [PROFILE] --region [REGION] \
  --query 'services[0].deployments[*].{status:status,running:runningCount,desired:desiredCount,rollout:rolloutState}'
```
