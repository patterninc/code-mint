# Skill Development Guide

How to create, evaluate, and maintain skills in the code-mint framework.

---

## What is a Skill

A skill is a directory containing a `SKILL.md` manifest and optional reference files. It encodes a repeatable procedure that an agent can load on demand. Skills reduce prompt spaghetti by moving stable workflows into versioned, reusable bundles.

```
skill-name/
├── SKILL.md              # Required — frontmatter + instructions
└── references/           # Optional — detailed docs, templates, examples
    └── detailed-guide.md
```

Skills are stored in `.agents/skills/` and automatically discovered by compliant AI coding agents.

---

## Naming Convention

Skills in this repository use a category prefix with double-hyphen separator:

```
{category}--{skill-name}
```

**Categories:**

| Prefix | Purpose |
|---|---|
| `meta--` | Onboarding and skill-library management: creating, evaluating, and maintaining skills (aligned with `AGENTS.md`). |
| `autonomy--` | Skills that increase the agent's ability to operate independently. |
| `legibility--` | Skills that make the codebase understandable to agents. |
| `clarity--` | Skills for collaborative planning and executable tickets once core onboarding is in place. |

**Rules:**
- Lowercase letters, numbers, and hyphens only
- Max 64 characters
- The `--` separator must appear exactly once (between category and skill name)

---

## SKILL.md Structure

Every skill requires YAML frontmatter followed by markdown instructions.

```markdown
---
name: category--skill-name
description: Brief description of what this skill does. Use when [trigger scenarios]. Do not use when [negative examples].
---

# Skill Title

[Instructions, steps, checklists, templates]
```

### Frontmatter Fields

| Field | Constraints | Purpose |
|---|---|---|
| `name` | Max 64 chars, lowercase/numbers/hyphens | Unique identifier |
| `description` | Max 1024 chars | Routing logic — determines when the agent invokes this skill |

### Writing Effective Descriptions

The description is the agent's decision boundary. It must answer:

- What are the outputs and success criteria?
- When should the agent use this skill?
- When should the agent NOT use this skill?

Include a concrete "Use when / Don't use when" block. Negative examples reduce misfires — without them, similar-looking skills can drop triggering accuracy by ~20%.

**Good:**
```
Audit whether a repository's .env configuration allows an agent to load all environment variables from scratch. Use when evaluating a new repo's agent-readiness or when env loading fails. Do not use when .env files already load correctly and the goal is to add new variables.
```

**Bad:**
```
Helps with environment variables.
```

---

## Drafting A Skill

Before writing a new skill, infer as much as you can from the current conversation, repository state, and existing examples. Ask only for the project facts the codebase cannot reveal confidently.

Capture these decisions before you draft:

1. **Category:** which family the skill belongs to (`meta--`, `autonomy--`, `legibility--`, or `clarity--`)
2. **Purpose:** the concrete task the skill performs
3. **Trigger boundary:** when the skill should fire and what similar-looking situations should route elsewhere
4. **Output:** what artifact or change the skill is expected to produce
5. **Project-local facts:** what belongs in `[CUSTOMIZE]` blocks versus reusable instructions
6. **Validation:** what signal proves the skill worked, or what kind of human review is required
7. **Existing asset reuse:** whether this should improve an existing skill instead of creating a new overlapping one

Prefer improving an existing skill over adding a near-duplicate. Stable skill IDs are easier for users and agents to learn, and they reduce routing drift.

---

## Authoring Principles

### Conciseness

The context window is shared with conversation history, code, and other skills. Every token competes for space. The agent is already capable — only add context it does not already have.

- SKILL.md should be under 500 lines.
- Move detailed reference material to `references/` files (loaded only when needed).
- Keep references one level deep: `SKILL.md` links into `references/`; avoid chains where one reference file links to another. Deeply nested references may be partially read.
- **Exception:** A skill may link to another skill’s `references/` file when one canonical guide should stay in a single place (for example `autonomy--runtime-creator` pointing at `autonomy--runtime-auditor/references/smoke-test-guide.md`). Prefer one home for shared content to avoid drift.

### Progressive Disclosure

Put essential instructions in SKILL.md. Put detailed benchmarks, checklists, templates, and examples in reference files.

```markdown
## Detailed Criteria
See [references/readiness-checklist.md](references/readiness-checklist.md) for scoring criteria.
```

Templates and examples inside skills are essentially free when the skill is not invoked — they only consume tokens when the agent actually loads the skill.

### Inline Customization Blocks

`[CUSTOMIZE]` sections are a supported pattern when a skill needs project-local values that cannot be generalized away. Good examples include:

- Infrastructure-as-Code repository locations
- AWS profile and region mappings
- Service names, log groups, and deployment workflow names
- Project-specific commands or directory names

Rules for using `[CUSTOMIZE]` blocks:

- Keep them explicit and easy to spot.
- Keep them narrowly scoped to project-local facts, not general workflow logic.
- Fill them collaboratively during onboarding or setup workflows.
- Preserve local values during upstream skill updates.
- Do not scatter project-specific edits throughout the skill when a small marked block will do.

### Degrees of Freedom

Match specificity to the task's fragility:

| Freedom Level | When to Use | Example |
|---|---|---|
| **High** (text instructions) | Multiple valid approaches | Code review guidelines |
| **Medium** (templates/pseudocode) | Preferred pattern with acceptable variation | Report generation |
| **Low** (exact scripts/commands) | Fragile operations, consistency critical | Database migrations |

---

## Auditor/Creator Pattern

Many skills come in pairs: an **auditor** that evaluates the current state, and a **creator** that remediates based on the auditor's findings.

### Handoff Protocol

1. **Auditor writes a structured report** to `.agents/reports/{skill-name}-audit.md` in the target repo. The report uses a consistent format: summary, itemized findings with severity, and recommended actions.
2. **Creator reads that report as its primary input.** If no report exists, the creator instructs the user to run the auditor first.
3. **After the creator finishes, the report is archived** to `.agents/reports/completed/` with a date suffix:
   ```bash
   mv .agents/reports/{skill-name}-audit.md .agents/reports/completed/{skill-name}-audit-$(date +%Y-%m-%d).md
   ```

The reports directory itself should stay in version control, but generated report files should not. Use `.gitkeep` files to preserve `.agents/reports/` and `.agents/reports/completed/`, and add this ignore pattern to the target repo's `.gitignore`:

```gitignore
.agents/reports/*
!.agents/reports/.gitkeep
!.agents/reports/completed/
.agents/reports/completed/*
!.agents/reports/completed/.gitkeep
```

### Auditor/Creator Pairs in This Repository

| Auditor | Creator |
|---|---|
| `autonomy--env-auditor` | `autonomy--env-creator` |
| `autonomy--runtime-auditor` | `autonomy--runtime-creator` |
| `autonomy--test-readiness-auditor` | `autonomy--test-readiness-creator` |
| `legibility--auditor` | `legibility--enhancer` |

`autonomy--sre-auditor` is intentionally not paired with a creator. Its remediation path routes to `autonomy--sre-agent`, an operational investigation tool rather than a remediation workflow. The auditor verifies tooling access; the agent uses that access to investigate real issues.

### Report Format

Auditor reports should follow this structure:

```markdown
# [Skill Name] Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Status:** [Pass / Partial / Fail]

## Summary
[2-3 sentence overview of findings]

## Findings

### [Finding 1 Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [What exists now]
- **Required State:** [What should exist]
- **Recommended Action:** [Specific remediation step]

## Next Steps
Run `{creator-skill-name}` to remediate findings.
```

At minimum, every audit report should also make the next handoff obvious:

- **Next Skill / Step:** What should run next, or what manual decision is needed

---

## Status Fingerprint

Every onboarded repository contains a committed `.agents/code-mint-status.json` file that serves as the machine-readable fingerprint of code-mint onboarding. It is an index for quick scanning, not a replacement for the detailed evidence in `docs/onboarding-checklist.md`.

### Template

```json
{
  "code_mint": "1.0",
  "scope": ".",
  "onboarded_at": null,
  "last_validated": null,
  "outcomes": {
    "validate_current_state": { "status": "Not Started", "date": null },
    "navigate": { "status": "Not Started", "date": null },
    "self_test": { "status": "Not Started", "date": null },
    "smoke_path": { "status": "Not Started", "date": null },
    "bug_reproduction": { "status": "Not Started", "date": null },
    "sre_investigation": { "status": "Not Started", "date": null }
  }
}
```

### Update Rule

Whenever a skill updates `docs/onboarding-checklist.md`, also update `.agents/code-mint-status.json` with the relevant outcome's current `status` and `date`.

**Status values** must be copied exactly from the checklist Status Key — `Not Started`, `In Progress`, `Blocked`, `Proven`, or `N/A`. Case and spacing matter; cross-repo scanners use strict string equality.

**Dates** must use date-only ISO format: `YYYY-MM-DD`. Never use datetime strings, locale dates, or timestamps.

**For `N/A` outcomes**, `date` remains `null`. Scanners should not assume every non-`Not Started` row has a non-null date.

**`Proven` means "last proven with recorded evidence"**, not a live guarantee. If the codebase drifts after proof, the fingerprint does not automatically invalidate. Use `last_validated` and periodic re-audits to detect staleness.

The `meta--onboarding` skill owns the lifecycle fields: `onboarded_at` is set when Phase 1 completes, and `last_validated` is set when Phase 5 verification runs.

### Scoping

The file lives at `.agents/code-mint-status.json` **relative to the onboarding scope root**. For repo-root onboarding this is `./.agents/code-mint-status.json`. For monorepo package onboarding at `packages/api/`, this is `packages/api/.agents/code-mint-status.json`. Skills use bare relative paths and assume the agent's working directory is the scope root.

### Create If Missing

`meta--onboarding` initializes the file during its Prepare Directories step. When a skill runs **standalone** (outside the onboarding playbook) and `.agents/code-mint-status.json` does not exist, the skill should create it from the template before updating. Set `scope` to `.` for repo-root or the appropriate path for scoped onboarding.

### Parallel Writes

When multiple skills run in parallel (for example Phase 1 auditors), do not have each skill update the JSON individually. Concurrent writes to a single JSON file lose data. Instead, defer fingerprint updates to a single step after all parallel skills complete. `meta--onboarding` handles this in its "After Phase 1" step.

### Cross-Repo Scanning

```bash
find . -name "code-mint-status.json" -exec jq '{path: input_filename, proven: [.outcomes[] | select(.status == "Proven")] | length}' {} \;
```

---

## Evaluating Skills

Define success criteria before writing the skill. Evaluate across four dimensions:

| Dimension | Question |
|---|---|
| **Outcome** | Did the task complete? Does the output meet acceptance criteria? |
| **Process** | Did the agent invoke the skill and follow the intended steps? |
| **Style** | Does the output follow the conventions specified in the skill? |
| **Efficiency** | Did the agent get there without thrashing (unnecessary commands, excessive tokens)? |

### Testing Skills

Use a small set of 10-20 test prompts to catch regressions:

- **Explicit invocation:** "Use the $skill-name skill to..." (confirms basic functionality)
- **Implicit invocation:** Natural language that should trigger the skill without naming it
- **Negative control:** Prompts that should NOT trigger the skill (catches false positives)
- **Edge case (optional):** Prompts that stress the fragile parts of the workflow

Review those prompts for three failure modes:

- the description is too vague to trigger reliably
- the instructions only handle tidy examples instead of realistic user phrasing
- the skill is overfit to one example instead of a reusable pattern

For high-leverage skills such as onboarding, include scenario coverage:

- Empty repo or repo with minimal docs
- Repo with existing `.agents/` or `AGENTS.md`
- Assessment-only flow
- Partial remediation flow
- Existing project-local `[CUSTOMIZE]` values that must be preserved

Track these test prompts alongside the skill. Every real-world failure becomes a new test case.

---

## Improving Existing Skills

When revising a skill that already exists:

1. Read the current `SKILL.md` and any linked `references/` files before editing.
2. Preserve the current skill name unless there is a strong compatibility reason to rename it.
3. Re-test the current description against realistic trigger prompts and near-miss prompts before rewriting it.
4. Prefer removing weak or duplicated instructions over adding more prose.
5. Re-verify that the revised skill is clearer, leaner, and less overfit than before.

If repeated examples keep implying the same deterministic helper, promote that helper into `scripts/` or `references/` instead of asking the agent to reinvent it on each run.

---

## Maintenance

Skills rot when the underlying systems change and the instructions do not.

- **PR rule:** Any PR that changes a workflow, tool, or convention covered by a skill must update the relevant SKILL.md.
- **Version the skill:** Update the skill when the procedures it encodes change.
- **Track status:** After running an auditor or creator skill, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the evidence and current status. Optionally refresh `docs/skills-status.md` if the repository keeps the compatibility view.
- **Periodic review:** Run auditor skills quarterly to identify drift.

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| Giant monolithic skill (>500 lines) | Saturates context, agent ignores buried instructions | Split into SKILL.md + references |
| Vague description | Agent can't decide when to invoke | Add specific triggers and negative examples |
| Multiple valid tools without a default | Agent picks randomly | Provide a default, document when to use alternatives |
| Time-sensitive instructions | Becomes stale silently | Use "current method" / "deprecated" sections |
| Mixed terminology | Agent inconsistently follows instructions | Pick one term per concept, use it everywhere |
| AI-generated AGENTS.md without human input | Misses implicit assumptions and UX intent | Always involve humans in authoring legibility docs |
