---
name: autonomy--test-readiness-creator
description: Builds the test setup, test utilities, and high-priority coverage needed for an agent to self-verify changes across unit, integration, end-to-end, visual, contract, and CI workflows. Use when a test readiness audit report exists and the testing setup needs improvement. Do not use when no audit report exists (run autonomy--test-readiness-auditor first) or when debugging one failing test (use autonomy--sre-agent).
---

# Test Readiness Creator

Build out a repository's testing setup so it supports autonomous agent verification, based on findings from `autonomy--test-readiness-auditor`.

**Scope:** Full test coverage is built incrementally, not in one session. During initial onboarding, focus on establishing the test framework, creating test utilities, and writing tests for the 3-5 highest-risk modules. Expand coverage in subsequent sessions.

## Prerequisites

Read the audit report at `.agents/reports/autonomy--test-readiness-auditor-audit.md`. If no report exists, instruct the user to run `autonomy--test-readiness-auditor` first.

## Step 1: Extract the Handoff Contract

Treat the audit report as the primary input contract before making changes:

1. Read `Top Blockers`, `Human Decisions Needed`, `Safe To Automate`, `Coverage Gaps`, and `Test-First Readiness`.
2. Pull out all Critical and High findings.
3. Identify which coverage priorities require human domain input before writing tests.
4. Confirm which actions are safe to automate immediately and which need approval first, especially CI or merge-blocking changes.

## Step 2: Prioritize Findings

Review the audit report and order findings by severity and impact on agent autonomy:

1. **Critical first:** Anything blocking the agent from running tests at all, such as a missing framework, broken commands, or no per-file execution.
2. **High next:** Slow tests, missing isolation, or external dependencies in unit tests.
3. **Medium:** Coverage gaps in high-risk areas such as auth, payments, or data validation.
4. **Low:** Missing snapshot tests, CI refinements, or coverage reporting.

Use the audit's `Coverage Gaps` and `Test-First Readiness` sections to decide the first 3-5 modules to tackle during onboarding.

## Step 3: Establish Test Infrastructure

If the audit found missing foundational elements, address these first:

### Test Framework Setup

1. Confirm the test framework is installed and configured.
2. Ensure a single test file can be executed in isolation. Document the exact command.
3. Configure test parallelization if not already present.

### Test Utilities

1. Create or document test factories or fixtures for common data objects.
2. Establish mocking or stubbing patterns consistent with the project's conventions.
3. Create a test helper file if one does not exist, with shared utilities.

### Test Naming Convention

Document the mapping from source files to test files:

```text
src/services/user.ts -> tests/services/user.test.ts
src/models/order.py -> tests/models/test_order.py
```

## Step 4: Build Test Suites

Work through the coverage gaps identified in the audit, collaborating with the user for domain-specific test cases.

### For Each Gap

1. **Identify the module and its purpose.** Read the `AGENTS.md` if one exists.
2. **Define test categories:**
   - **Happy path:** Does the feature work as expected under normal conditions?
   - **Edge cases:** What happens with boundary values, empty inputs, or max-length strings?
   - **Error paths:** What happens when dependencies fail, validation fails, or inputs are invalid?
   - **Regression guards:** What bugs have occurred before? Write tests that prevent recurrence.
3. **Write the tests.** Follow TDD where possible:
   - Write a failing test that describes expected behavior.
   - Confirm it fails for the right reason.
   - Implement or verify the behavior.
   - Confirm the test passes.
4. **Verify isolation:** Ensure new tests have no external dependencies in unit tests and use ephemeral environments for integration tests.

### Coverage Priority Order

| Priority | Area | Rationale |
|---|---|---|
| Critical | Authentication and authorization | Changes here create security vulnerabilities |
| Critical | Payment and financial logic | Direct business impact |
| High | Data validation and sanitization | Edge cases cause data corruption |
| High | API contracts, request shapes, and response shapes | Breaking changes cascade across consumers |
| Medium | UI state management | Visible but usually recoverable |
| Medium | Error handling paths | Untested error paths produce cryptic failures |
| Lower | Cosmetic or display logic | Low risk, easily caught in review |

## Step 5: Configure CI Integration

If the audit found CI gaps:

Before changing CI behavior, ask for explicit approval if the change will block merges, alter required checks, or materially change team workflows.

1. Ensure tests run automatically on every PR or commit.
2. Configure structured output, such as JUnit XML or JSON, for parseable results.
3. Set test failure to block merge rather than acting only as advisory output.
4. Add reasonable timeout configuration to prevent hung suites.
5. If applicable, add flaky test detection and quarantine.

## Step 6: Verify and Archive

1. Run the full test suite and confirm all new tests pass.
2. Verify per-file test execution works with the documented command.
3. Confirm test speed meets benchmarks where practical.
4. Archive the audit report to `.agents/reports/completed/autonomy--test-readiness-auditor-audit-{YYYY-MM-DD}.md`.
5. Update `docs/onboarding-checklist.md` and `.agents/code-mint-status.json` with the current `self_test` outcome status and date. Optionally update `docs/skills-status.md` if the repository keeps the compatibility view.
