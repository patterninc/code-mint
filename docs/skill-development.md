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

For high-leverage skills such as onboarding, include scenario coverage:

- Empty repo or repo with minimal docs
- Repo with existing `.agents/` or `AGENTS.md`
- Assessment-only flow
- Partial remediation flow
- Existing project-local `[CUSTOMIZE]` values that must be preserved

Track these test prompts alongside the skill. Every real-world failure becomes a new test case.

---

## Maintenance

Skills rot when the underlying systems change and the instructions do not.

- **PR rule:** Any PR that changes a workflow, tool, or convention covered by a skill must update the relevant SKILL.md.
- **Version the skill:** Update the skill when the procedures it encodes change.
- **Track status:** After running an auditor or creator skill, update `docs/onboarding-checklist.md` with the evidence and current status. Optionally refresh `docs/skills-status.md` if the repository keeps the compatibility view.
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
