# Contributing to Code-Mint

Thank you for your interest in improving code-mint. This document explains how to propose changes and what to expect during review.

## How to Contribute

1. **Fork the repository** and create a feature branch from `main`.
2. **Make your changes** following the conventions below.
3. **Open a pull request** with a clear description of what changed and why.

## What We Accept

- Improvements to existing skills, templates, and documentation
- New skills that follow the auditor/creator pairing model
- Bug fixes in templates or broken references
- Expanded worked examples for additional tooling stacks

## Conventions

- **Skill IDs are stable.** Do not rename an existing skill ID unless there is a strong compatibility reason. See `AGENTS.md` for this policy.
- **No deprecated alias stubs:** This repository ships only the skill directories under `.agents/skills/` in this tree. If you copied older alias folders from a prior code-mint bundle, point agents at the canonical skill IDs listed in `docs/outcomes.md` and remove the obsolete paths.
- **`[CUSTOMIZE]` blocks stay generic.** Skills ship with worked examples (e.g., Datadog, AWS, GitHub CLI) but must not assume a specific vendor as the only option.
- **Outcome-first language.** Describe what the user or agent will be able to prove, not just what procedure runs.
- **Progressive disclosure.** Root docs stay concise; detailed guidance lives in skill references or `docs/`.

## Skill Development

See `docs/skill-development.md` for the canonical guide on creating and maintaining skills, including the auditor/creator pattern, frontmatter format, validation expectations, and how to revise existing skills without creating overlap.

## Documentation change checklist (maintainers)

**Single source for outcome vocabulary:** `docs/outcomes.md` is the canonical glossary for outcome names, proof criteria, and primary skill mappings. When you change any of that, edit `docs/outcomes.md` first, then update pointers and checklists in `README.md`, `docs/framework.md`, `docs/onboarding-checklist.md`, `docs/skills-status.md`, `.agents/code-mint-status.json`, and `.agents/skills/meta--onboarding/SKILL.md` as needed so they stay aligned.

After substantive edits to `README.md`, onboarding docs, or the copy bundle, quickly confirm:

- A new reader can find the six outcomes and `docs/onboarding-checklist.md` without hunting.
- The quick-start copy commands still match the paths listed in the repository map, including `TARGET_REPO` / `TARGET_SCOPE` and monorepo `.gitignore` notes.
- Cross-links between `README.md`, `docs/outcomes.md`, and `docs/adoption-guide.md` still resolve.

### Markdown link check (CI)

GitHub Actions runs [Lychee](https://github.com/lycheeverse/lychee) on `**/*.md` for pushes to `main` and for pull requests. Skill templates must not contain clickable placeholder URLs (for example fake `references/{file}.md` links); use backticks or real paths so the check passes.

The workflow retries failed HTTP requests a few times to reduce flakes. If CI still fails on an **external** URL (for example a standards site that is temporarily down or rate-limits bots), confirm the link in a browser, then either wait and re-run the workflow or replace the link with a stable alternative—do not merge known-dead URLs.

## Security

For security-sensitive reports (unsafe guidance, credential handling, or similar), follow `SECURITY.md`. Do not post exploit details in public issues before maintainers have had a chance to review.

## Review Process

Pull requests are reviewed for clarity, accuracy, and alignment with the outcome-first design. Expect feedback on:

- Whether the change preserves the `[CUSTOMIZE]` pattern for vendor-specific content
- Whether skill IDs and cross-references remain consistent
- Whether the change improves agent performance without adding unnecessary complexity

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0 that covers this project. See the `LICENSE` file for details.
