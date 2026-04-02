# Code-Mint

Code-mint is the source repository for harness-engineering documentation, skills, and templates (portable reference material for target repos). It is not an executable product. Do not attempt to build, test, or run this repository.

## Editing Priorities

When working in this repo:

1. Keep `README.md` as the single human-facing entry point.
2. Keep this file agent-facing and concise.
3. Preserve stable skill IDs unless there is a strong compatibility reason to rename them.
4. Treat `docs/onboarding-checklist.md` as the canonical progress tracker and `docs/outcomes.md` as the outcome glossary.
5. Update cross-links when adding or moving onboarding docs so the repo map stays trustworthy.

## Repository Map

| Path | Purpose |
|---|---|
| `README.md` | Human-facing overview, quick start, and onboarding journey |
| `AGENTS.md` | Concise agent-facing instructions for this repository (this file) |
| `.agents/skills/` | Portable skills copied into target repositories |
| `.agents/rules/` | Guidance on project-specific persistent rules |
| `.agents/code-mint-status.json` | Machine-readable onboarding fingerprint (committed) |
| `docs/framework.md` | Founding principles and mechanical standards |
| `docs/outcomes.md` | North-star onboarding outcomes and evidence model |
| `docs/onboarding-checklist.md` | Canonical outcome checklist template |
| `docs/adoption-guide.md` | Detailed operator workflow for onboarding a target repo |
| `docs/skills-status.md` | Compatibility matrix from skills to outcomes |
| `docs/skill-development.md` | How to create and maintain skills |
| `.agents/skills/legibility--enhancer/references/` | Root and subdirectory `AGENTS.md` templates |

## Skill Categories

| Category | Meaning |
|---|---|
| `meta--` | Onboarding and skill-library management |
| `legibility--` | Repo mapping and `AGENTS.md` creation |
| `autonomy--` | Self-verification, runtime readiness, env setup, and operational tooling (including `autonomy--sre-*`) |
| `clarity--` | Collaborative planning and executable tickets (post-core onboarding) |

## Working Model

Code-mint is intentionally outcome-first:

- the user should quickly understand what the onboarding will unlock
- progress should be visible in a checklist the user can revisit later
- proof should be recorded as reports, smoke paths, failing tests, or investigation notes

## Standards

Use the exact filename `AGENTS.md`. Subdirectory `AGENTS.md` files override parent files rather than merging with them.
