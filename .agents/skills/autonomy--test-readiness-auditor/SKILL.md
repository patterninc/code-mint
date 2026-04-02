---
name: autonomy--test-readiness-auditor
description: Evaluate whether a repository's unit, integration, end-to-end, visual, contract, and CI test setup lets an AI agent self-verify changes through fast, isolated, and well-targeted automated feedback. Use when auditing test readiness, onboarding a new project, or assessing whether the agent can safely validate its own work. Do not use when the main question is app startup or runtime bootstrapping (use autonomy--runtime-auditor) or when debugging one failing test (use autonomy--sre-agent).
---

# Test Readiness Auditor

Evaluate whether the repository's tests provide the feedback loops needed for autonomous agent work. This includes unit, integration, end-to-end, visual or snapshot, contract, and CI testing strategy, along with speed, isolation, and developer ergonomics. An agent that cannot self-verify is a liability.

## Step 1: Discover the Test Infrastructure

1. Identify the test framework or frameworks in use.
2. Locate the documented test commands in `AGENTS.md`, README, CI config, and package manager scripts.
3. Map which test types are present: unit, integration, end-to-end, visual or snapshot, contract, performance.
4. Determine whether tests can run per file, per module, or by explicit test name.

## Step 2: Evaluate Test Speed

Fast feedback is non-negotiable for agentic iteration.

| Rating | Unit Test Speed | Suite Speed |
|---|---|---|
| Excellent | <200ms per test | <30s full suite |
| Acceptable | <500ms per test | <2min full suite |
| Poor | >500ms per test | >2min full suite |
| Failing | Tests timeout or are flaky | Suite cannot complete reliably |

Check for:

- [ ] Per-file or targeted test execution is possible
- [ ] Test parallelization is configured where appropriate
- [ ] Unit tests avoid unnecessary I/O, network calls, and sleeps
- [ ] Test databases or services are fast-reset or ephemeral

## Step 3: Evaluate Test Isolation

- [ ] Unit tests have no external dependencies, such as database, network, or filesystem requirements
- [ ] Integration tests use isolated environments, such as containers, test databases, or mock services
- [ ] Tests can run concurrently without interfering with each other
- [ ] Test state is fully reset between runs
- [ ] The agent can spin up a testing sandbox autonomously with documented commands

## Step 4: Evaluate Coverage and Gaps

- [ ] Core business logic has unit test coverage
- [ ] API endpoints have integration tests where applicable
- [ ] Error paths and edge cases are tested, not just happy paths
- [ ] Error handling is consistent and observable
- [ ] UI components have visual or snapshot tests when the project warrants them
- [ ] Database migrations or schema changes have validation or rollback coverage when relevant
- [ ] Authentication and authorization flows are tested

Identify the highest-risk untested areas: modules with complex logic, external integrations, or user-facing impact that lack effective tests.

## Step 5: Evaluate Test-First Feasibility

Determine whether an agent can follow a test-first workflow:

- [ ] The framework supports writing and running a single test in isolation
- [ ] Test file naming conventions are documented or inferrable
- [ ] Test utilities, fixtures, and factories are available and documented
- [ ] Mocking or stubbing patterns are established and consistent
- [ ] An agent can create a new test file, write a failing test, and run it without human intervention

## Step 6: Evaluate CI Integration

- [ ] Tests run automatically on every PR or commit
- [ ] CI results are accessible and parseable
- [ ] Flaky test detection and quarantine is in place where needed
- [ ] Test failure blocks merge rather than acting only as advisory output

## Output

Ensure the report directory exists: `mkdir -p .agents/reports/completed && touch .agents/reports/.gitkeep .agents/reports/completed/.gitkeep`

Ensure `.gitignore` ignores generated report contents while preserving the directories with their `.gitkeep` files.

Write the report to `.agents/reports/autonomy--test-readiness-auditor-audit.md`:

```markdown
# Test Readiness Audit Report
**Repository:** [name]
**Date:** [timestamp]
**Overall Test Readiness:** [Excellent / Acceptable / Poor / Failing]

## Summary
- Agent Can Self-Verify: [Yes / Partially / No]
- Test-First Feasible: [Yes / No]

## Top Blockers
[Highest-severity blockers preventing autonomous verification]

## Human Decisions Needed
[Coverage priorities, domain-specific risk areas, or CI policy decisions]

## Safe To Automate
[Framework setup, helper creation, or low-risk test tooling improvements that can proceed immediately]

## Test Infrastructure
- Framework(s): [list]
- Test Types Present: [Unit, Integration, E2E, Visual, Contract, etc.]
- Test Command: [exact command]
- Per-File Execution: [Yes / No]

## Findings

### [Finding Title]
- **Severity:** [Critical / High / Medium / Low]
- **Current State:** [what exists]
- **Required State:** [what should exist]
- **Recommended Action:** [specific step]
- **Next Skill / Step:** [e.g., Run `autonomy--test-readiness-creator`]

## Speed Assessment
- Unit Test Speed: [Xms average]
- Full Suite Speed: [Xs]
- Rating: [Excellent / Acceptable / Poor / Failing]

## Isolation Assessment
- External Dependencies in Unit Tests: [Yes / No]
- Concurrent Execution Safe: [Yes / No]
- Autonomous Sandbox: [Yes / No — describe what's missing]

## Coverage Gaps
[Prioritized list of untested areas with risk assessment]

## Test-First Readiness
[Assessment with specific blockers if any]

## Next Steps
Run `autonomy--test-readiness-creator` to remediate findings.
```

After writing the report, update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `self_test` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.

## Detailed Criteria

See [references/testing-standards.md](references/testing-standards.md) for detailed speed benchmarks, isolation standards, coverage priorities, and CI integration checklist.
