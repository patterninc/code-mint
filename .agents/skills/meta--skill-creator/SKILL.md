---
name: meta--skill-creator
description: Best practices and template for creating new agent skills in the code-mint framework. Use when creating a new skill, authoring SKILL.md files, or structuring skill directories. Do not use when editing an existing skill's logic (edit directly instead) or when creating AGENTS.md files (use legibility--enhancer).
---

# Skill Creator

Create new skills that follow code-mint conventions. This skill guides you through the full lifecycle: gathering requirements, writing the SKILL.md, creating reference files, and verifying the result.

## External Reference

For broader skill-authoring guidance, see Anthropic's `skill-creator`:
https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md

Use the general lessons from that guide, but prefer code-mint conventions over product-specific tooling or Claude-only mechanics.

## Step 1: Gather Requirements

Before writing anything, extract what you can from the current conversation, existing files, and any examples the user already gave you. Ask only for the missing pieces.

Then answer these questions:

1. **Category:** Which category does this skill belong to?
   - `meta--` — Onboarding and skill-library management (including authoring new skills)
   - `autonomy--` — Agent independence (env, runtime, test readiness, SRE tooling)
   - `legibility--` — Codebase understandability
   - `clarity--` — Collaborative planning and high-quality executable tickets (post-core onboarding)

2. **Purpose:** What specific task does this skill perform?

3. **Trigger scenarios:** When should an agent invoke this skill? When should it NOT?

4. **Auditor or Creator?** If this is part of a pair:
   - Auditors evaluate and produce a report to `.agents/reports/{name}-audit.md`
   - Creators consume an auditor report and remediate findings
   - See the Auditor/Creator Pattern section below

5. **Output:** What artifact does this skill produce? (report, code changes, documentation, etc.)

6. **Domain knowledge:** What does the agent need to know that it wouldn't already know?

7. **Customization needs:** Which project-local facts belong in explicit `[CUSTOMIZE]` blocks, and which instructions should remain fully reusable?

8. **Validation style:** Should this skill be validated with objective checks, human review, or both?

9. **Existing asset reuse:** Is this a new skill, or should it preserve and improve an existing skill's name, structure, and reference files?

### Discovery Rules

- Infer purpose, triggers, outputs, and constraints from conversation context before interviewing the user.
- Ask about edge cases, failure modes, and competing trigger scenarios early enough to avoid rewriting later.
- Prefer realistic user phrasing over abstract examples when discussing how the skill should be invoked.
- If the user wants to improve an existing skill, preserve the skill name unless they explicitly want a rename.

---

## Step 2: Create the Directory

```
.agents/skills/{category}--{skill-name}/
├── SKILL.md              # Required
└── references/           # Optional — only if needed
    └── detailed-guide.md
```

**Naming rules:**
- Lowercase letters, numbers, and hyphens only
- Max 64 characters total
- Double-hyphen `--` separates category from skill name
- Use descriptive names: `autonomy--env-auditor` not `autonomy--env`

---

## Step 3: Write the SKILL.md

### Frontmatter

```yaml
---
name: {category}--{skill-name}
description: {What this skill does}. Use when {trigger scenarios}. Do not use when {negative examples}.
---
```

**Description rules:**
- Max 1024 characters
- Write in third person ("Evaluates...", "Creates...", not "I evaluate..." or "You can use this to...")
- Include both WHAT (capabilities) and WHEN (triggers)
- Include at least one "Do not use when" clause
- Be specific enough that the agent can distinguish this skill from similar ones
- Put trigger guidance in the description, not scattered through the body

### Body Structure

Follow this skeleton. Omit sections that don't apply.

```markdown
# {Skill Title}

{One-sentence summary of what this skill does and why.}

## Step 1: {First Action}
{Instructions}

## Step 2: {Next Action}
{Instructions}

## Output
{What artifact is produced and where it goes}

## Detailed Criteria
Link to a real file under `references/` (for example `references/readiness-checklist.md`). Do not use placeholder markdown links that resolve to non-existent paths.
```

**Body rules:**
- Under 500 lines
- Use numbered steps for sequential workflows
- Use checklists (`- [ ]`) for evaluation criteria
- Provide exact commands where possible (agents should be able to copy-paste)
- State assumptions rather than asking questions (anti-pedantry rule)
- One term per concept throughout — no mixing "endpoint" / "route" / "URL"
- If the skill needs project-local facts, isolate them in explicit `[CUSTOMIZE]` blocks
- Prefer explaining why an instruction matters over piling on rigid MUST/NEVER language
- If repeated examples imply the same deterministic helper, promote it into `scripts/` or `references/` instead of making the agent reinvent it each run

---

## Step 4: Draft Usage Tests

Before finalizing the skill, draft a small set of realistic prompts:

1. **Trigger prompts (2-3):** prompts a real user would say when this skill should fire
2. **Near-miss prompts (1-2):** prompts that share vocabulary or context but should NOT trigger this skill
3. **Edge-case prompts (optional):** prompts that stress the fragile parts of the workflow

Use these prompts to review:

- whether the description is specific enough to trigger in the right situations
- whether the instructions handle realistic user phrasing rather than only tidy examples
- whether the skill is overfit to one example instead of a reusable pattern

### Validation Guidance

- For objective skills, define what artifact, command result, or pass/fail signal would prove the skill works.
- For subjective skills, prefer human review with concrete example outputs over fake precision.
- Do not force a rigid scoring system onto skills whose quality depends mostly on judgment.

---

## Step 5: Write Reference Files (If Needed)

Move detailed content to `references/` when:
- The SKILL.md would exceed 500 lines without it
- The content is a detailed checklist, scoring rubric, or template
- The content is only needed for specific sub-tasks within the skill

**Reference file rules:**
- Keep references one level deep (`SKILL.md` links to `references/`; avoid chains where one reference links to another).
- **Exception:** Linking to another skill’s `references/` file is allowed when sharing one canonical guide (for example the runtime creator linking to the runtime auditor’s smoke-test guide). Prefer a single home for shared content.
- Each reference file should be self-contained.
- Name files descriptively: `testing-standards.md`, `readiness-checklist.md`, `ticket-formats.md`

---

## Step 6: Verify

Run through this checklist before finalizing:

- [ ] Name is `{category}--{skill-name}`, lowercase/hyphens only, under 64 chars
- [ ] Description includes WHAT, WHEN, and at least one DO NOT USE WHEN
- [ ] Description is under 1024 characters
- [ ] SKILL.md body is under 500 lines
- [ ] Instructions use consistent terminology throughout
- [ ] Sequential workflows use numbered steps
- [ ] Commands are exact and copy-pasteable
- [ ] Reference files (if any) follow the depth rules above (including the cross-skill exception when applicable)
- [ ] If this is an auditor: output report path is `.agents/reports/{name}-audit.md`
- [ ] If this is a creator: reads from `.agents/reports/{auditor-name}-audit.md` and archives after completion
- [ ] Report instructions mention that `.agents/reports/` stays in Git via `.gitkeep`, while generated report files are ignored by `.gitignore`
- [ ] Project-local facts are isolated in `[CUSTOMIZE]` blocks rather than scattered through the file
- [ ] You have at least one explicit invocation prompt, one implicit trigger prompt, and one negative-control prompt for this skill
- [ ] The skill has been reviewed against 2-3 realistic user prompts, not just tidy examples
- [ ] The description has been checked against near-miss prompts that should not trigger it
- [ ] The instructions explain key reasoning where that improves generalization
- [ ] Repeated deterministic work has been promoted into `scripts/` or `references/` where appropriate
- [ ] If improving an existing skill, the current name and reusable assets have been preserved unless the user approved a rename

---

## Step 7: Iterate

Good skills are usually discovered through a small feedback loop, not written perfectly in one pass.

Use this cycle:

1. Draft the skill structure and description.
2. Review it against realistic trigger and near-miss prompts.
3. Check whether the instructions solve a reusable pattern rather than one narrow example.
4. Revise based on feedback, missing edge cases, or confusing trigger behavior.
5. Repeat until the skill is clear, lean, and reliable.

### Iteration Rules

- Generalize from feedback instead of hard-coding one example's quirks.
- Remove instructions that are not pulling their weight.
- Watch for repeated helper logic across examples; bundle it once rather than regenerating it repeatedly.
- If the user prefers a lightweight collaborative pass instead of formal evaluation, keep the loop conversational but still test the skill against realistic prompts.

---

## Auditor/Creator Pattern

When creating a paired auditor and creator:

### Auditor Skills

- Evaluate the current state against defined criteria
- Produce a structured report at `.agents/reports/{skill-name}-audit.md`
- Report format: Summary, itemized findings with severity (Critical/High/Medium/Low), recommended actions
- Include: `Next Skill / Step`
- End with: "Run `{creator-skill-name}` to remediate findings."
- Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the relevant outcome status and date after completion. Optionally refresh `docs/skills-status.md` if the repository keeps the compatibility view.

### Creator Skills

- Begin by reading the auditor report from `.agents/reports/{auditor-name}-audit.md`
- If no report exists, instruct the user to run the auditor first
- Walk through findings collaboratively with the user (do not auto-generate without human input where domain knowledge is required)
- Add approval checkpoints before any supervised, destructive, or hard-to-reverse action
- After remediation, archive the report to `.agents/reports/completed/{name}-audit-{YYYY-MM-DD}.md`
- Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the relevant outcome status and date after completion. Optionally refresh `docs/skills-status.md` if the repository keeps the compatibility view.

---

## Common Patterns

### Evaluation Pattern (Auditors)

```markdown
## Step 1: Discover
{Gather facts about the current state}

## Step 2: Evaluate
{Check against criteria}

## Step 3: Score/Report
{Produce structured output}
```

### Remediation Pattern (Creators)

```markdown
## Step 1: Read Audit Report
{Load findings from .agents/reports/}

## Step 2: Prioritize
{Order findings by severity}

## Step 3: Remediate
{Walk through each finding with the user}

## Step 4: Verify
{Confirm fixes address findings}

## Step 5: Archive
{Move report to completed/}
```

### Customization Pattern

```markdown
## Project-Local Customization

Fill these `[CUSTOMIZE]` blocks collaboratively with the user:

- [CUSTOMIZE: project-local fact]
- [CUSTOMIZE: project-local fact]
```

### Workflow Pattern (Investigation And Clarity Skills)

```markdown
## Step 1: Gather Context
{Collect evidence}

## Step 2: Analyze
{Form hypotheses or triage}

## Step 3: Execute
{Take action}

## Step 4: Verify
{Confirm the action succeeded}
```

### Improvement Pattern (Existing Skills)

```markdown
## Step 1: Read Current Skill
{Understand the current purpose, triggers, and assets before editing}

## Step 2: Preserve Stable Identity
{Keep the skill name and reusable structure unless a rename is explicitly approved}

## Step 3: Test Against Real Prompts
{Use realistic trigger and near-miss prompts to find confusion or gaps}

## Step 4: Revise
{Improve the description, instructions, scripts, or references}

## Step 5: Re-verify
{Confirm the revised skill is clearer, leaner, and less overfit}
```
