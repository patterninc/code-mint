---
name: autonomy--env-creator
description: Creates the capacity for an agent to load all environment variables from scratch, based on the output of autonomy--env-auditor. Use when an env audit report exists and environment loading needs to be remediated. Do not use when no audit report exists (run autonomy--env-auditor first) or when debugging a single missing variable.
---

# Environment Creator

Build out a repository's environment variable configuration so that an agent (or new developer) can go from zero to a fully configured `.env` in a single command. Based on findings from `autonomy--env-auditor`.

Prefer one documented bootstrap path that writes local env config from the repo's real source of truth. In many repos that is `scripts/env.sh`. A `.env.example` file can still exist, but it should be secondary.

## Prerequisites

Read the audit report at `.agents/reports/autonomy--env-auditor-audit.md`. If no report exists, instruct the user to run `autonomy--env-auditor` first.

Before changing anything:

1. Assume the current local `.env` may contain the only surviving copy of a secret.
2. Never delete or overwrite the active `.env` as the first step.
3. Prefer improving an existing bootstrap flow over replacing it.
4. Keep `.env` and backup files ignored by Git.

## Step 1: Review and Classify Findings

1. Read the audit report's variable inventory and findings.
2. Order by severity: Critical (hard blockers) first, then High, Medium, Low.
3. Classify each variable into one of two categories:

| Category | Description | Example |
|---|---|---|
| **Provisioned Secret** | Secrets, credentials, and shared config that should come from the project's secret source | `DATABASE_URL`, `JWT_SECRET`, `STRIPE_API_KEY` |
| **Local Override** | Values that are always the same for local development and can be hardcoded in the script | `NODE_ENV=development`, `LOG_FORMAT=text`, `AWS_REGION=us-west-2` |

## Step 2: Confirm the Secret Source

Collaborate with the user to confirm where provisioned secrets come from and how the repo should fetch them.

The example below uses **AWS SSM Parameter Store** as a worked reference. If your project uses HashiCorp Vault, Doppler, 1Password CLI, GCP Secret Manager, Azure Key Vault, or another secret backend, adapt the fetch commands while following the same pattern: agree on a prefix/path, verify parameters exist, identify gaps, and confirm naming conventions.

If the project uses AWS SSM, use the steps below. Otherwise adapt the same pattern to the project's secret system.

1. **Agree on the SSM prefix** with the user. Convention: `/<project-name>/` (e.g., `/my-project/`, `/api-service/`). The prefix should match what Terraform or IaC uses for container secrets.
2. **Verify parameters exist** by running:
   ```bash
   aws ssm get-parameters-by-path \
     --path "/<project-name>/" \
     --recursive \
     --with-decryption \
     --output json \
     --region <region> | jq -r '.Parameters[].Name'
   ```
3. **Identify gaps:** Compare the list of SSM parameters against the audit report's variable inventory. Flag any missing parameters.
4. **SSM naming convention:** Parameter names must use the env var name as the last path segment, in uppercase (e.g., `/<project-name>/DATABASE_URL`, not `/<project-name>/database_url`). This is what the script uses to derive the variable name.
5. For any missing parameters, ask the user to create them (or create them together if the user has the values). Do NOT create SSM parameters containing production secrets without explicit user approval.

## Step 3: Create the Primary Bootstrap Command

Create or update the project's primary env bootstrap command. In many repos this is `scripts/env.sh`. The generated temp file must always end by being promoted into `.env` — either directly, or after validation.

```bash
#!/bin/bash

# Usage:  bash scripts/env.sh [--profile <aws_profile>]
#
# Populates .env by fetching secrets from AWS SSM Parameter Store.
# AWS SSO authentication is required.
#
# Prerequisites: aws cli, jq
# Login first:   aws sso login --profile <profile>

set -eo pipefail

PROFILE=""
ENV_FILE=".env"
TMP_ENV_FILE=".env.tmp.$$"
SSM_PREFIX="/<project-name>"  # [CUSTOMIZE] set to the project's SSM prefix
AWS_REGION="us-west-2"        # [CUSTOMIZE] set to the project's AWS region

# ─── Parse Arguments ────────────────────────────────────────────

while [[ $# -gt 0 ]]; do
  case "$1" in
    --profile)
      PROFILE="$2"
      shift 2
      ;;
    *)
      echo "Unknown option: $1"
      echo "Usage: bash scripts/env.sh [--profile <aws_profile>]"
      exit 1
      ;;
  esac
done

PROFILE_FLAG=""
if [[ -n "$PROFILE" ]]; then
  PROFILE_FLAG="--profile $PROFILE"
fi

# ─── Check AWS Credentials ──────────────────────────────────────

if ! aws sts get-caller-identity $PROFILE_FLAG &>/dev/null; then
  echo "AWS credentials not valid. Logging in..."
  if [[ -n "$PROFILE" ]]; then
    aws sso login --profile "$PROFILE"
  else
    echo "Error: No active AWS session. Run: aws sso login --profile <profile>"
    exit 1
  fi
fi

# ─── Back Up Existing .env ──────────────────────────────────────

if [[ -f "$ENV_FILE" ]]; then
  BACKUP="$ENV_FILE.bak.$(date +%Y%m%d%H%M%S)"
  cp "$ENV_FILE" "$BACKUP"
  echo "Backed up existing .env to $BACKUP"
fi

# ─── Start Fresh Temp File ──────────────────────────────────────

rm -f "$TMP_ENV_FILE"
touch "$TMP_ENV_FILE"

# ─── Fetch SSM Parameters ───────────────────────────────────────

echo "Fetching SSM parameters from $SSM_PREFIX ..."

aws ssm get-parameters-by-path \
  --path "$SSM_PREFIX" \
  --recursive \
  --with-decryption \
  --output json \
  --region "$AWS_REGION" \
  $PROFILE_FLAG | \
  jq -r '.Parameters[] | (.Name | split("/") | .[-1]) as $key | select($key | test("^[A-Z]")) | "\($key)=\(.Value | @sh)"' \
  >> "$TMP_ENV_FILE"

if [[ ! -s "$TMP_ENV_FILE" ]]; then
  echo "Warning: No SSM parameters found under $SSM_PREFIX"
  echo "Verify the prefix and that parameters exist in the $AWS_REGION region."
fi

# ─── Local Development Overrides ────────────────────────────────
# Values below are appended after SSM parameters.
# They either override SSM values for local dev or add variables
# that don't belong in Parameter Store.

cat <<'EOF' >> "$TMP_ENV_FILE"

# ─── Local Development Overrides ─────────────────────────────
# These values are specific to local development and are not
# stored in SSM. Edit as needed for your environment.

# NODE_ENV=development
# DATABASE_URL=postgresql://localhost:5432/<project>_dev
# REDIS_URL=redis://localhost:6379
# LOG_FORMAT=text
EOF
```

The script is not complete until one of these endings exists:

1. **Preferred:** Validate `"$TMP_ENV_FILE"` and then run `mv "$TMP_ENV_FILE" "$ENV_FILE"`.
2. **Minimal fallback:** If validation is skipped, still end with `mv "$TMP_ENV_FILE" "$ENV_FILE"` so the temp output becomes the active `.env`.

### Adapting the Template

When creating the script for a specific project:

1. **Set `SSM_PREFIX`** to the project's actual prefix (from Step 2).
2. **Set `AWS_REGION`** to the project's region.
3. **Replace the local overrides heredoc** with the actual local override variables identified in Step 1. Uncomment lines that apply and remove placeholder examples.
4. **Add any exclude filters** if certain SSM parameters should not appear in the local `.env` (e.g., production-only values). Add a `grep -vE` filter after the `jq` pipeline:
   ```bash
   grep -vE '^(PRODUCTION_ONLY_VAR|ANOTHER_VAR)=' \
   ```
5. **Make the script executable:** `chmod +x scripts/env.sh`
6. **Add `.env`, `.env.tmp.*`, and `.env.bak.*` to `.gitignore`** if not already present.

## Step 4: Add Validation (Optional)

If the project doesn't already validate environment variables at startup, add a validation step — either inside `env.sh` or as application startup logic. Promotion from the temp file to `.env` should happen only after validation.

### In-script validation (recommended)

Add a validation block at the end of `env.sh` that checks required variables are present in the generated temp file before promotion:

```bash
# ─── Validate Required Variables ─────────────────────────────

echo "Validating required environment variables..."
MISSING=0

for VAR in DATABASE_URL JWT_SECRET; do  # ← list required vars
  if ! grep -q "^${VAR}=" "$TMP_ENV_FILE"; then
    echo "  MISSING: $VAR"
    MISSING=$((MISSING + 1))
  fi
done

if [[ $MISSING -gt 0 ]]; then
  echo "Error: $MISSING required variable(s) missing from .env"
  echo "Check that the SSM parameters exist under $SSM_PREFIX"
  rm -f "$TMP_ENV_FILE"
  exit 1
fi

mv "$TMP_ENV_FILE" "$ENV_FILE"
echo "All required variables present."
```

If you skip the validation block, add a final promotion step at the end of the script:

```bash
mv "$TMP_ENV_FILE" "$ENV_FILE"
echo "Done — wrote $ENV_FILE"
```

### Application startup validation

If the project has a typed config layer (e.g., Go struct, Zod schema, Pydantic model), add validation there that fails fast with a clear error naming the missing variable.

## Step 5: Document in AGENTS.md and README

Update the project's AGENTS.md and/or README to include:

1. **How to populate `.env`:**
   ```
   bash scripts/env.sh --profile <your-aws-profile>
   ```
2. **Prerequisites:** The CLI tools required by your secret backend (e.g., AWS CLI v2 and `jq` for SSM; `vault` for HashiCorp Vault; `doppler` for Doppler).
3. **How to authenticate:** The login command for your secret backend (e.g., `aws sso login --profile <profile>` for AWS SSM).
4. **What it does:** Fetches or assembles the local env config from the project's source of truth and writes `.env`.
5. **When to re-run:** After SSM parameters are added/changed, or when setting up a new machine.
6. **Recovery behavior:** The script creates a timestamped backup before replacing `.env`. If generation or validation fails, keep the prior `.env` and restore from the backup if needed.

## Step 6: Verify and Archive

1. If a `.env` file already exists, create a timestamped backup before verification.
2. Run `bash scripts/env.sh --profile <profile>` and confirm it writes a complete `.env`.
3. Start the application and verify it boots successfully.
4. If generation or startup verification fails, keep the old `.env` in place or restore it from the backup before trying again.
5. If a `.env.example` file exists, keep it as a reference or compatibility file only. Point it to the primary bootstrap path if helpful.
6. Archive the audit report to `.agents/reports/completed/autonomy--env-auditor-audit-{YYYY-MM-DD}.md`.
7. Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `smoke_path` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
