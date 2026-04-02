---
name: legibility--auditor
description: Audits a repository's documentation coverage and agent-readiness by evaluating AGENTS.md placement, progressive disclosure quality, and UX intent documentation. Use when evaluating repository structure, auditing documentation coverage, assessing agent-readiness, or onboarding a codebase to agent-first practices. Do not use when creating or editing AGENTS.md files directly (use legibility--enhancer instead).
---

# Legibility Auditor

Evaluate how thoroughly a repository is mapped for autonomous agent navigation and produce an Agent Readiness Report. A well-mapped repository enables an agent to understand both *how* something works (technical implementation) and *why* it exists (user experience intent). This skill identifies where that mapping is missing, incomplete, or misleading. The findings support the **`Navigate`** outcome defined in `docs/outcomes.md`.

## Step 1: Crawl the Repository Structure

1. List all directories and subdirectories.
2. For each directory, record: directory path, estimated lines of code, whether an AGENTS.md file exists.
3. Identify the root AGENTS.md and note its contents.
4. If the git repository root is **above** the onboarding scope (monorepo or package-only onboarding): locate and read `README.md` at the repository root and any `README.md` on the path from the repo root to the scope (when present). Summarize repo-wide operational notes (deploy, release, CI, smoke paths, environments) that apply to the scoped package.

## Step 2: Evaluate Root AGENTS.md

**Scope:** Evaluate the root `AGENTS.md` for the onboarding target (the git repository root when onboarding the entire repository, or the scoped project directory when onboarding a monorepo package or subdirectory).

Check against these criteria:

- [ ] **Exists** at the onboarding scope root
- [ ] **Project overview** is present and accurate
- [ ] **Tech stack** is explicitly declared (languages, frameworks, databases)
- [ ] **Build/test/lint commands** are listed with exact shell commands
- [ ] **Signpost index** maps subdirectories to their purpose
- [ ] **Conventions** are stated: naming rules, dependency direction, error handling patterns
- [ ] **Length** is under 100 lines (concise, not exhaustive)

## Step 3: Evaluate Subdirectory Coverage

For every directory that appears complex enough to justify local guidance, evaluate whether a localized `AGENTS.md` should exist. Use this rubric:

- **Soft size signal:** Around 300+ lines of code is a prompt to evaluate the directory, not an automatic requirement.
- **Complexity signals:** A subdirectory `AGENTS.md` is usually warranted if the directory has two or more of the following:
  - Multiple responsibilities or entrypoints
  - User-facing failure modes or UX promises
  - Non-obvious business rules or local conventions
  - Repeated agent/human touchpoints
  - Historical gotchas, edge cases, or hidden assumptions
- **Simple-directory exception:** If a directory is straightforward, low-risk, and follows root conventions cleanly, document that rationale and do not require a local file yet.

For each directory that the rubric flags as needing local guidance:

- [ ] A localized AGENTS.md exists
- [ ] The AGENTS.md documents the module's **technical purpose** (how)
- [ ] The AGENTS.md documents the module's **UX intent** (why — what user-facing behavior depends on this module)
- [ ] Known gotchas, edge cases, or non-obvious constraints are documented
- [ ] The file is under 50 lines

**Precedence check:** Verify that subdirectory files are self-contained enough to guide an agent without the root file (subdirectory `AGENTS.md` files override, not merge with, parent files). That rule is about **which `AGENTS.md` wins at runtime**; it does not excuse skipping parent README files during this audit.

**Monorepo / scoped-scope check:** If operational documentation (deploy, release, smoke, production runbooks) appears only in a parent or repository-root `README.md` and is missing from the scoped `AGENTS.md` or from a `README.md` under the onboarding scope, record that as a **finding** (documentation gap), not as optional context.

## Step 4: Identify Hidden Assumptions

Look for implicit knowledge that is not documented anywhere:

- **Scoped scope:** When the onboarding target is not the git repository root, treat workflows documented only in parent or root README files (see Step 1) as hidden assumptions for the scoped package until they are copied, summarized, or explicitly referenced in scoped `AGENTS.md` or docs.
- Configuration values that only make sense with external context
- Business rules embedded in code without explanation
- Naming conventions that are followed but never stated
- Dependency direction between layers that is respected by convention but not documented or enforced
- Duplicated logic that should live in a single source of truth
- Error handling patterns that are inconsistent or undocumented
- API contracts between modules that exist only in developers' heads

## Step 5: Score Across Readiness Dimensions

Score the repository across eight **readiness dimensions** (not the three framework pillars in `docs/framework.md`) using the detailed checklist in [references/readiness-checklist.md](references/readiness-checklist.md). Each dimension is scored as:

- **Level 0 — Ad Hoc:** No documentation or structure. Agent would fail.
- **Level 1 — Emerging:** Partial coverage. Agent needs significant human guidance.
- **Level 2 — Defined:** Adequate coverage. Agent can handle routine tasks.
- **Level 3 — Standardized:** Strong coverage. Agent can handle most tasks autonomously.
- **Level 4 — Optimized:** Comprehensive. Agent operates as a first-class contributor.

## Output

Ensure the report directory exists: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Write the report to `.agents/reports/legibility--auditor-audit.md` using this structure:

```
# Legibility Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Readiness Level:** [0-4]

## Summary
[2-3 sentence overview]

## Dimension Scores
| Dimension | Score | Key Finding |
|---|---|---|
| Documentation | [0-4] | [finding] |
| Build System | [0-4] | [finding] |
| Testing | [0-4] | [finding] |
| Style & Validation | [0-4] | [finding] |
| Dev Environment | [0-4] | [finding] |
| Code Quality | [0-4] | [finding] |
| Observability | [0-4] | [finding] |
| Security | [0-4] | [finding] |

## Missing AGENTS.md Files
[Directories that need AGENTS.md files, ordered by priority]

## Hidden Assumptions Found
[Undocumented assumptions with recommended documentation locations]

## Findings
### [Finding Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [what exists now]
- **Required State:** [what should exist]
- **Recommended Action:** [specific step]
- **Next Skill / Step:** [e.g., Run `legibility--enhancer` for this module]

## Next Steps
Run `legibility--enhancer` to collaboratively remediate findings.
```

After writing the report, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `navigate` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.

## Detailed Criteria

See [references/readiness-checklist.md](references/readiness-checklist.md) for scoring criteria for each of the eight readiness dimensions.
