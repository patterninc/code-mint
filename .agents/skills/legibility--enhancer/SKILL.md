---
name: legibility--enhancer
description: Collaboratively walks a human through creating high-quality AGENTS.md files throughout a codebase, implementing progressive disclosure architecture. Use when creating or updating AGENTS.md files, onboarding a repository to agent-first practices, or remediating findings from legibility--auditor. Do not use when evaluating existing documentation coverage (use legibility--auditor) or when creating agent skills (use meta--skill-creator).
---

# Legibility Enhancer

Walk through the process of creating high-quality AGENTS.md files with a human collaborator. AI-generated AGENTS.md files created without human input miss implicit assumptions and UX intent — the most valuable parts. This skill ensures those assumptions become explicit.

## Prerequisites

If a legibility audit report exists at `.agents/reports/legibility--auditor-audit.md`, read it first and prioritize directories flagged as missing AGENTS.md files. If no report exists, begin with Step 1.

## Collaboration Rules

Use a workshop pattern, not a questionnaire dump.

1. Inspect the repository before asking questions that the codebase can answer.
2. Follow the anti-pedantry rule from `docs/framework.md`: never ask a question if the answer can be reasonably inferred from repository structure, standard tooling, or existing docs.
3. Prefer short feedback loops:
   - inspect
   - draft
   - ask for correction
   - revise
4. Ask freeform questions when you need domain knowledge, UX intent, hidden assumptions, or historical context.
5. After each major section, show the human a concise draft of what you plan to write and ask what is wrong, missing, or overstated.

## Step 1: Gather Project Context

If the working scope is **not** the git repository root (monorepo or package-only onboarding), read the repository root `README.md` and any `README.md` files on the path from the repo root to the scope before drafting. Use them to capture repo-wide deploy, release, CI, and environment workflows that apply to this package. Fold relevant commands into the scoped root `AGENTS.md`, or add explicit references (for example: "Deploy: see repository root `README.md`, section …") so an agent working only in the scope still has an actionable path.

Before writing anything, inspect the repository and draft your best provisional answers for:

1. The repository's primary purpose
2. The tech stack (languages, frameworks, databases, infrastructure)
3. The exact commands to build, test, lint, and run the project
4. The major modules/directories and what each does
5. Any visible project-wide conventions (naming, file organization, dependency direction)

Then present that draft to the human and ask for corrections, not just raw restatement. Example:

```text
Here is my draft read of the repo: [summary]. I still need your help with the parts the code cannot tell me confidently, especially UX intent, hidden constraints, and which related repositories actually matter in day-to-day work. What is inaccurate or missing?
```

After that draft review, work with the human to answer any unresolved questions:

1. What is this repository's primary purpose?
2. What is the tech stack (languages, frameworks, databases, infrastructure)?
3. What are the exact commands to build, test, lint, and run the project?
4. What are the major modules/directories and what does each do?
5. Are there any project-wide conventions (naming, file organization, dependency direction)?
6. **Are there companion repositories the agent needs to know about?** Especially:
   - Infrastructure-as-Code repos (e.g., `{repo-name}-tf` for Terraform)
   - Shared library repos
   - Frontend/backend counterparts
   - Documentation or design system repos
   For each, record the repository name, what it contains, and its clone URL. These go in the "Related Repositories" section of the root AGENTS.md.

   **Proactive discovery:** If a VCS host CLI is available (e.g., `gh` for GitHub, `glab` for GitLab), search for companion repos automatically. GitHub example:
   ```bash
   # Detect the current repo's GitHub org and name
   REPO_SLUG=$(gh repo view --json nameWithOwner -q '.nameWithOwner')
   ORG=$(echo "$REPO_SLUG" | cut -d/ -f1)
   REPO_NAME=$(echo "$REPO_SLUG" | cut -d/ -f2)

   # Search for IaC and companion repos by naming convention
   gh repo list "$ORG" --limit 100 --json name,url \
     --jq ".[] | select(.name | test(\"${REPO_NAME}\")) | \"\(.name) \(.url)\""
   ```
   Present any matches to the user and ask which are relevant.

## Step 2: Create the Root AGENTS.md

Follow the template in [references/agents-md-guide.md](references/agents-md-guide.md). Include:

1. **One-paragraph project overview.** What this repository is and what it does.
2. **Tech stack declaration.** Explicit list — do not assume the agent knows.
3. **Build/test/lint commands.** Exact shell commands, not descriptions. An agent should be able to copy-paste and run them.
4. **Signpost index.** A table mapping each major directory to a one-line description of its purpose.
5. **Key conventions.** Only project-wide rules: naming conventions, dependency direction, error handling patterns. Module-specific rules go in subdirectory files.

Before finalizing the file, show the human a concise draft of:

- the one-paragraph project overview
- the commands table
- the directory map
- the project-wide conventions you believe belong in the root file

Ask the human what should be corrected, trimmed, or moved out of the root file. Favor concise, high-signal output over exhaustive inventory.

### Verify the Root File

- Is it under 100 lines? If not, move details to subdirectory files or reference docs.
- Can an agent that reads only this file understand where to look for any given task?
- Are the build/test commands accurate? Run them to confirm.

## Step 3: Create Subdirectory AGENTS.md Files

### When to Create One

Use this standard decision rubric:

- **Soft size signal:** Around 300+ lines of code is a prompt to evaluate the directory, not a hard threshold.
- **Create a subdirectory `AGENTS.md` when the directory has two or more of these signals:**
  - Multiple responsibilities or entrypoints
  - Conventions that differ from the root
  - User-facing behavior that requires UX intent documentation
  - Hidden assumptions, historical gotchas, or non-obvious constraints
  - An agent working in this directory would need context not present in the root file
- **Do not force one yet** if the directory is dead simple, low-risk, and follows root conventions cleanly. In that case, note the rationale and move on.

### Extracting UX Intent

This is the hardest and most valuable part. Ask the human collaborator:

1. What happens to the user if this module is down or slow?
2. What user-visible behavior would change if this module's output changed?
3. What implicit promises does this module make? (e.g., "search results appear within 200ms," "uploaded files are never lost")
4. What edge cases have caused user-facing bugs before?
5. What would a new engineer need to know to avoid breaking the user experience?

Encourage narrative answers and concrete examples. If the human is unsure, ask for one real incident, one failure mode, or one "easy way to break this module" story instead of pushing for abstract wording.

Document the answers directly in the subdirectory AGENTS.md under a "UX Intent" heading, but first compress them into a short draft summary and confirm it with the human.

### What to Include

1. **Module purpose (how).** What this module does technically.
2. **UX intent (why).** What user-facing behavior depends on this module.
3. **Key files.** A brief map of the most important files and their roles.
4. **Local conventions.** Any rules that apply only within this module.
5. **Gotchas.** Non-obvious constraints, edge cases, or things that have broken before.

After drafting a subdirectory file, ask:

```text
Here is the 5-line version I would give to a new engineer or agent working in this module. What is wrong, missing, or too generic?
```

Revise until the summary reflects real module behavior rather than template filler.

### Precedence Rule

Subdirectory `AGENTS.md` files **override** parent files — they do not merge. An agent working in a subdirectory will only see the nearest AGENTS.md file. This means:

- Subdirectory files must be self-contained enough to guide the agent independently
- If a root-level convention applies in a subdirectory, repeat it or reference it explicitly
- Do not assume the agent has read the root file when working in a subdirectory

**Authoring vs runtime:** When *creating* or *updating* scoped `AGENTS.md` files, use parent and repository-root README files to **populate** scoped content (commands, links, short summaries). At **runtime**, the nearest `AGENTS.md` still wins; the authoring step is how you bring parent README facts into scope so the scoped file stays self-contained.

## Step 4: Verify and Archive

1. Review all created AGENTS.md files for accuracy and completeness.
2. Confirm all build/test commands work by running them.
3. Summarize which directories received local guidance, which were intentionally deferred as simple, and why.
4. If a legibility audit report was used, archive it to `.agents/reports/completed/legibility--auditor-audit-{YYYY-MM-DD}.md`.
5. Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `navigate` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.

## Detailed Templates

See [references/agents-md-guide.md](references/agents-md-guide.md) for root and subdirectory AGENTS.md templates with examples.
