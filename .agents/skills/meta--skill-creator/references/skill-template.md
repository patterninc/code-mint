# Skill Template

Copy this template when creating a new skill. Replace all `{PLACEHOLDER}` values.

---

## SKILL.md Template

```markdown
---
name: {category}--{skill-name}
description: {What this skill does in one sentence}. Use when {specific trigger scenarios}. Do not use when {negative examples — when NOT to use this skill}.
---

# {Skill Title}

{One-sentence summary of purpose.}

## Step 1: {First Action}

{Clear instructions. Use numbered sub-steps for complex procedures.}

1. {Sub-step}
2. {Sub-step}

## Step 2: {Next Action}

{Instructions. Include exact commands where applicable:}

```bash
{exact command the agent should run}
```

## Project-Local Customization

Fill these `[CUSTOMIZE]` blocks collaboratively when the skill needs project-specific facts:

- [CUSTOMIZE: project-local fact]
- [CUSTOMIZE: project-local fact]

## Step 3: {Evaluation or Verification}

Check against these criteria:

- [ ] {Criterion 1}
- [ ] {Criterion 2}
- [ ] {Criterion 3}

## Output

{Describe what artifact this skill produces and where it goes.}

{For auditors:}
Write the report to `.agents/reports/{skill-name}-audit.md` using the format defined in `docs/skill-development.md`.

Ensure the report directory exists: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Include `Next Skill / Step`.

{For creators:}
Read the audit report from `.agents/reports/{auditor-name}-audit.md`. After remediation, archive to `.agents/reports/completed/`.
Add approval checkpoints before destructive or hard-to-reverse actions.

## Detailed Criteria

Link to a real file under `references/` (for example `references/readiness-checklist.md`) for {description of what the reference contains}. Replace the filename with your actual reference; avoid placeholder links that break link checkers.
```

Create a minimal prompt test set alongside the skill:

- Explicit invocation prompt
- Implicit trigger prompt
- Negative-control prompt

---

## Directory Structure Template

```
.agents/skills/{category}--{skill-name}/
├── SKILL.md
└── references/           # Include only if SKILL.md would exceed 500 lines
    └── {descriptive-name}.md
```

---

## Description Examples

**Auditor:**
```yaml
description: Audit whether a repository's environment variable configuration allows an agent to load all required .env variables from scratch. Use when evaluating a new repository's agent-readiness or when environment loading fails during agent setup. Do not use when .env files already load correctly and the goal is to add new individual variables.
```

**Creator:**
```yaml
description: Create the capacity for an agent to load all environment variables from scratch, based on the output of autonomy--env-auditor. Use when an env audit report exists and environment loading needs to be remediated. Do not use when no audit report exists (run autonomy--env-auditor first) or when the goal is debugging a single missing variable.
```

**Autonomy (operational investigation):**
```yaml
description: Structured root-cause analysis workflow for investigating active or historical infrastructure issues, runtime errors, and unexpected behavior. Use when encountering a runtime error, production incident, test failure, or investigating a bug report. Do not use when the issue is a known feature request or cosmetic change.
```
