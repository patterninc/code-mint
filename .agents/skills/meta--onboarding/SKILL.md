---
name: meta--onboarding
description: Step-by-step playbook that transforms a repository for AI-first development. It scopes the repository, runs read-only audits, maintains an outcome-driven checklist, guides collaborative improvements, and verifies the results with evidence. Use when onboarding a new repository, when a user says "set up this repo for agents," or when starting a harness engineering transformation. Do not use when the repo is already onboarded and the goal is to run a single specific skill.
---

# Onboarding Playbook

Transform a repository for AI-first development in phases. This playbook is assessment-first, approval-gated, and outcome-driven. The user should always be able to tell what the next proof is and what evidence already exists.

## How It Works

- Start with discovery, not assumptions.
- Baseline the current state before making changes.
- Track progress in `docs/onboarding-checklist.md`, not only in chat.
- Ask only the minimum questions needed to unblock the next proof.
- Treat an outcome as complete only when evidence is recorded in-repo.

## North-Star Outcomes

Maintain these outcomes as the public promise of onboarding:

1. `Validate Current State`
2. `Navigate`
3. `Self-Test`
4. `Smoke Path`
5. `Bug Reproduction`
6. `SRE Investigation`

Use `docs/outcomes.md` for the glossary and `docs/onboarding-checklist.md` as the system of record.

**Phases vs outcomes:** This playbook runs in five phases; a single phase may touch more than one north-star outcome when the proofs are coupled (for example Self-Test before Smoke Path, or bug repro alongside operational investigation). That does not relax the rule: prove and record each outcome separately in the checklist—one row at a time in the recommended order—before treating the next outcome as done.

## Context Transfer

- Active audit reports live in `.agents/reports/*-audit.md`
- Archived reports live in `.agents/reports/completed/`
- Outcome tracking lives in `docs/onboarding-checklist.md`
- Status fingerprint lives in `.agents/code-mint-status.json` (committed to git)
- Optional skill-to-outcome compatibility lives in `docs/skills-status.md`
- Phase decisions and approvals live in `.agents/reports/onboarding-summary.md`

### Scoped scope and root-only artifacts

When the onboarding scope is **not** the git repository root, briefly check **as needed** (not an exhaustive audit): `.github/workflows/` at the repo root, root `package.json` and workspace manifests (pnpm, turbo, nx, and similar), root `docker-compose` / `Makefile` / `Justfile`, and root-level `docs/` or ADRs when they define release or CI behavior for the service. Summarize or link what matters in scoped `AGENTS.md`, `docs/`, or `.agents/reports/onboarding-summary.md` so work in the scope does not rely on unstated parent-only context. This extends the README discovery in Safety Preflight step 3 to other common root-only locations.

**Scoping rule:** The target may be an entire repository or a specific directory/project within a larger repository or monorepo. Infer the intended scope from the user's initial instructions, and treat paths, commands, and AGENTS.md placement as relative to that scope unless a step explicitly needs the containing repository root for git operations. Reading the git repository root `README.md` (and READMEs on the path to the scope) for discovery is separate from that rule: use them to find repo-wide workflows, then record or reference what matters inside the scope.

## Before You Begin

### Safety Preflight

Before running any auditors or creators:

1. Confirm the target scope from the user's instructions. It may be the repository root or a subdirectory/project inside a monorepo.
2. Identify the containing repository root separately for git operations such as branching, diffing, and committing.
3. If the onboarding scope is **not** the repository root: locate the repo root and any `README.md` files on the path from the repo root to the scope; skim them for workflows relevant to the scoped project (deploy, release, CI, environment). Note in `.agents/reports/onboarding-summary.md` or audit output when important facts live only outside the scope so remediation can copy or reference them into scoped `AGENTS.md` or `docs/`.
4. Check whether the worktree is on a dedicated onboarding branch for the current phase. If not, ask the user to create one before continuing.
5. Detect whether `.agents/`, `AGENTS.md`, `docs/onboarding-checklist.md`, or prior audit reports already exist within the target scope.
6. If existing agent instructions are present, treat them as user-owned state. Do not overwrite or replace them blindly.
7. Explain the default posture: **Phase 1 is assessment-first.**
8. Explain that higher-risk actions such as env replacement, CI changes, deployment changes, and commits require an explicit approval checkpoint even after remediation begins.
9. Tell the user that the goal is not to "finish onboarding" abstractly. The goal is to prove specific capabilities with evidence.

### Prepare Directories

Ensure the following paths exist inside the target scope:

```bash
mkdir -p .agents/reports .agents/reports/completed docs
touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep
```

Ensure these docs exist inside the target scope. If they do not, copy them from code-mint before continuing:

- `docs/framework.md`
- `docs/outcomes.md`
- `docs/onboarding-checklist.md`
- `docs/skills-status.md` when the repo wants the compatibility view

Initialize `.agents/reports/onboarding-summary.md` if it does not exist. Use it as the running memo for:

- scope and `N/A` decisions
- current outcome statuses
- human decisions and approvals taken so far
- blockers or manual follow-ups
- which proof should happen next
- the next recommended PR or phase

Initialize `.agents/code-mint-status.json` if it does not already exist. Copy it from the code-mint template and set `scope` to the onboarding scope path relative to the git root (`.` for repo-root onboarding, or the subdirectory path for scoped onboarding). This file is committed to git and serves as the machine-readable fingerprint for cross-repo scanning.

Ensure `.gitignore` keeps the directories but ignores generated report files:

```gitignore
.agents/reports/*
!.agents/reports/.gitkeep
!.agents/reports/completed/
.agents/reports/completed/*
!.agents/reports/completed/.gitkeep
```

If the git repository uses a **single root `.gitignore`** and the onboarding scope is **not** the repository root, add the same patterns there with a path prefix for the scope, or place a `.gitignore` under the scope directory.

Use this structure:

```markdown
# Onboarding Summary
**Repository:** [name]
**Current Phase:** [Phase 1 / Phase 2 / Phase 3 / Phase 4 / Phase 5]

## Outcomes
- Validate Current State: [Not Started / In Progress / Proven / Blocked / N/A]
- Navigate: [Not Started / In Progress / Proven / Blocked / N/A]
- Self-Test: [Not Started / In Progress / Proven / Blocked / N/A]
- Smoke Path: [Not Started / In Progress / Proven / Blocked / N/A]
- Bug Reproduction: [Not Started / In Progress / Proven / Blocked / N/A]
- SRE Investigation: [Not Started / In Progress / Proven / Blocked / N/A]

## Current Priorities
- [top priority]

## Decisions And Approvals
- [decision or approval]

## Blockers
- [blocker]

## Next Step
- [next proof or manual follow-up]

## Next PR
- [recommended branch or PR scope]
```

### Scope the Onboarding

Start with discovery, not taxonomy. Before you ask about detailed skill applicability, ask 1-2 freeform questions that uncover the real onboarding goal. Good starting prompts:

1. **What do you want an agent to be able to do in this repo that it cannot do reliably today?**
2. **When a new developer or agent tries to onboard, where does the process usually go off-script?**

Use the repository context to infer likely scope, then summarize your understanding back to the user before asking narrower follow-up questions. Example:

```text
Here is my current read: you want better codebase legibility, a trustworthy test path, and a safe smoke path that an agent can use to confirm the repo is actually working. I think that means navigation, self-test, and runtime proof are definitely in scope; SRE investigation may depend on whether operational tooling exists.
```

Only after that summary should you confirm applicability for the remaining areas:

1. **Cloud infrastructure** - Does this project deploy to AWS, GCP, Azure, or another cloud provider?
2. **Monitoring tools** - Does the team use Datadog, Grafana, New Relic, or another monitoring tool?
3. **Environment variables** - Does the project use `.env` files or environment variable configuration?
4. **Runtime bootstrapping** - Does the project need help getting from a clean checkout to a safe local runtime or staging-like equivalent?
5. **Companion repositories** - Does this project have related repositories the agent needs to know about?

Mark any inapplicable outcome or supporting area as `N/A` in `docs/onboarding-checklist.md`. Skip those steps later.

Instead of forcing the user into a fixed path immediately, ask about preferences that shape the recommendation:

- How much change is welcome right now versus later?
- Which proof matters most first: navigation, tests, smoke path, bug reproduction, or SRE investigation?
- Are there any risky areas the user wants kept read-only for now?

Then recommend the next phase sequence in plain language and ask for confirmation.

## Phase 1: Assess The Current State

Run applicable auditors to understand the current state. These are non-destructive, read-only assessments that produce reports to `.agents/reports/`.

### Step 1.1: Legibility Audit (Always Run)

**Skill:** `legibility--auditor`
**Output:** `.agents/reports/legibility--auditor-audit.md`

### Step 1.2: Test Readiness Audit (Always Run)

**Skill:** `autonomy--test-readiness-auditor`
**Output:** `.agents/reports/autonomy--test-readiness-auditor-audit.md`

### Step 1.3: Environment Audit (If Applicable)

**Skill:** `autonomy--env-auditor`
**Output:** `.agents/reports/autonomy--env-auditor-audit.md`

### Step 1.4: Runtime Audit (If Applicable)

**Skill:** `autonomy--runtime-auditor`
**Output:** `.agents/reports/autonomy--runtime-auditor-audit.md`

### Step 1.5: SRE Tooling Audit (If Applicable)

**Skill:** `autonomy--sre-auditor`
**Output:** `.agents/reports/autonomy--sre-auditor-audit.md`

### Parallelism Rule

Once scope is known, Phase 1 auditors may run in parallel because they are read-only. Do not parallelize creator or remediation work in later phases unless the scopes are clearly independent and the user approves.

When auditors run in parallel, **do not** have each auditor update `.agents/code-mint-status.json` individually — concurrent writes to the same file lose data. Instead, defer all fingerprint updates to the single "After Phase 1" step below, which writes all outcome statuses in one pass.

### After Phase 1

Update `docs/onboarding-checklist.md` with:

- `Validate Current State` = `Proven` when the baseline reports and summary exist and the summary captures what is working, blocked, risky, and next to prove
- each remaining outcome marked `Not Started`, `In Progress`, `Blocked`, or `N/A` based on the audit findings
- the next proof to pursue

Update `.agents/code-mint-status.json`: set `onboarded_at` to today's ISO date, and update each outcome's `status` and `date` to match the checklist.

Present a concise summary to the user:

```text
Phase 1 Assessment Complete.

Validate Current State: Proven - [one-line summary]
Navigate: [status] - [one-line summary]
Self-Test: [status] - [one-line summary]
Smoke Path: [status] - [one-line summary]
Bug Reproduction: [status] - [one-line summary]
SRE Investigation: [status] - [one-line summary]

Recommended next proof: [single next outcome]
```

Then stop and ask for approval before making changes.

Persist the Phase 1 summary into `.agents/reports/onboarding-summary.md`, including:

- outcome statuses and `N/A` decisions
- top priorities from the audit reports
- whether the user chose assessment-only, selective remediation, or full guided onboarding
- any explicit approvals or unresolved blockers
- the next proof or manual step
- the recommended next PR phase

## Phase 2: Prove Navigation

This phase makes the codebase navigable enough for the agent to explain where work belongs and what user-facing behavior depends on each module.

### Step 2.1: Create Root AGENTS.md

**Skill:** `legibility--enhancer`
**Input:** `.agents/reports/legibility--auditor-audit.md`

**Root** means the onboarding scope root: the repository root when onboarding the full repo, or the scoped project directory when onboarding a monorepo package.

Walk through the root `AGENTS.md` collaboratively. Capture:

- project purpose and tech stack
- exact build, test, lint, and run commands
- major directories and what they do
- project-wide conventions
- approval boundaries and related repositories when relevant

### Step 2.2: Create Subdirectory AGENTS.md Files

Prioritize the 3-5 highest-value modules that need local context. Capture:

- module purpose
- UX intent
- key files
- local conventions
- gotchas and edge cases

### Proof Criteria For `Navigate`

Mark `Navigate` as `Proven` only when:

1. the root `AGENTS.md` exists and is accurate
2. priority modules have localized guidance where needed
3. the agent can explain where a sample task should happen and that grounded answer is recorded in the checklist

Record the proof in `docs/onboarding-checklist.md`.

## Phase 3: Prove Self-Test And Smoke Path

Use the audit reports from Phase 1 to remediate the next approved gap.

### Step 3.1: Self-Test

**Skill:** `autonomy--test-readiness-creator`
**Input:** `.agents/reports/autonomy--test-readiness-auditor-audit.md`

Focus on establishing the smallest trustworthy targeted test path for a high-value module or behavior.

### Proof Criteria For `Self-Test`

Mark `Self-Test` as `Proven` only when:

1. the exact targeted test command or test target is documented
2. the scope it covers is clear
3. the pass/fail signal is recorded in `docs/onboarding-checklist.md`

### Step 3.2: Environment Setup (If Applicable)

**Skill:** `autonomy--env-creator`
**Input:** `.agents/reports/autonomy--env-auditor-audit.md`

Before any `.env` rewrite:

1. Confirm whether a local `.env` may contain the only surviving copy of any secret.
2. Create a timestamped backup first.
3. Validate the generated output before promoting it into place.
4. If validation fails, keep the prior `.env` as the active file and surface recovery instructions.

### Step 3.3: Smoke Path (If Applicable)

**Skill:** `autonomy--runtime-creator`
**Input:** `.agents/reports/autonomy--runtime-auditor-audit.md`

Build setup scripts, local service flows, and smoke-check documentation only after the user approves those changes. The agent handles scripting; the human provides infrastructure context and confirms approval-gated decisions.

### Proof Criteria For `Smoke Path`

Mark `Smoke Path` as `Proven` only when:

1. the runtime target is clear
2. one trusted, non-destructive smoke path is documented
3. prerequisites, exact steps, stop conditions, and a concrete success signal are recorded as evidence

## Phase 4: Prove Bug Reproduction And Investigation

### Step 4.1: Bug Reproduction

Use `autonomy--sre-agent` or the local debugging workflow to turn one real bug report into a deterministic repro. Favor a failing test when possible.

### Proof Criteria For `Bug Reproduction`

Mark `Bug Reproduction` as `Proven` only when:

1. there is a concrete bug report or observed failure
2. a deterministic repro path exists that another person or agent can rerun
3. the failing test, script, or repro recipe tied to that issue is recorded in the checklist

### Step 4.2: SRE Investigation

If the project has logs, metrics, traces, or deployment history, customize and use `autonomy--sre-agent`.

If the SRE agent needs project-local values, fill the `[CUSTOMIZE]` sections in:

- `.agents/skills/autonomy--sre-agent/SKILL.md`
- `.agents/skills/autonomy--sre-agent/references/recipes.md`

### Proof Criteria For `SRE Investigation`

Mark `SRE Investigation` as `Proven` only when:

1. the required tools are accessible
2. the agent can gather evidence from at least one operational source
3. the investigation note contains a ranked hypothesis and clear next actions, or the outcome is recorded as `N/A` with a reason

If the repository truly has no operational tooling, mark the outcome `N/A`.

## Phase 5: Verify And Activate

Re-run all applicable auditors to confirm improvements.

### Step 5.1: Preserve Baseline

Before re-running auditors, archive Phase 1 reports so before/after comparison remains possible:

```bash
for f in .agents/reports/*-audit.md; do
  [ -f "$f" ] && cp "$f" ".agents/reports/completed/$(basename "$f" .md)-baseline-$(date +%Y-%m-%d).md"
done
```

### Step 5.2: Re-run Auditors

Run each applicable auditor again and compare the new results against the Phase 1 baseline.

### Step 5.3: Update Tracking

1. Update `docs/onboarding-checklist.md` with final statuses, evidence, and dates.
2. Update `.agents/code-mint-status.json`: set `last_validated` to today's ISO date, and update all outcome statuses and dates to their final values.
3. Optionally update `docs/skills-status.md` if the repo wants the compatibility view.
4. Archive completed audit reports to `.agents/reports/completed/`.
5. Update `.agents/reports/onboarding-summary.md` with what changed, what remains manual, unresolved risks, and the next recommended skill or proof.
6. Ask whether the user wants to create a commit. Do not commit automatically.

## Activate Ongoing Skills

Once the onboarding outcomes are proven, these ongoing skills become more useful:

| Skill | When To Use |
|---|---|
| `clarity--ticket-writer` | When a PM or engineer has a new feature request or bug report |
| `autonomy--sre-agent` | When investigating production issues, errors, or incidents |
| `legibility--auditor` | Quarterly, to audit documentation coverage as the codebase evolves |
| `autonomy--test-readiness-auditor` | When test infrastructure changes or new modules are added |
| `meta--skill-creator` | When you need to create new project-specific skills |

### Recommended Cadence

- **Every PR:** If the PR changes module boundaries, conventions, or key behavior, update relevant `AGENTS.md` files.
- **Monthly:** Refresh the smallest critical test paths and smoke-path evidence.
- **Quarterly:** Re-run legibility and test-readiness audits to catch drift.
- **As needed:** Run `autonomy--sre-agent` for real failures and `clarity--ticket-writer` for new work.

The durable progress checklist lives in `docs/onboarding-checklist.md`. Do not create a second parallel checklist in chat.
