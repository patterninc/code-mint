# Test Readiness Standards Reference

Detailed benchmarks and criteria for the Test Readiness Auditor.

---

## Speed Benchmarks

### Unit Tests

The target for agentic iteration is **<200ms per individual test**. This allows an agent to run a targeted test, observe the result, and iterate within a single reasoning cycle.

| Metric | Target | Acceptable | Unacceptable |
|---|---|---|---|
| Single test execution | <200ms | <500ms | >500ms |
| Full unit suite | <30s | <2min | >2min |
| Per-file test run | <5s | <15s | >15s |

Common causes of slow unit tests:
- Database connections in tests that should be pure logic
- Network calls that should be mocked
- Filesystem operations that should use in-memory alternatives
- Sleep statements or artificial delays
- Heavy setup or teardown that could be shared or simplified

### Integration Tests

| Metric | Target | Acceptable | Unacceptable |
|---|---|---|---|
| Single integration test | <5s | <15s | >30s |
| Full integration suite | <5min | <15min | >15min |
| Environment spin-up | <30s | <2min | >5min |

### End-to-End Tests

| Metric | Target | Acceptable | Unacceptable |
|---|---|---|---|
| Single E2E test | <30s | <1min | >2min |
| Full E2E suite | <15min | <30min | >30min |

---

## Isolation Standards

### Unit Test Isolation

Unit tests must have **zero external dependencies**:
- No database connections, use in-memory alternatives or mocks
- No network calls, use stubs or dependency injection
- No filesystem reads or writes, use in-memory filesystems or temp directories with cleanup
- No shared mutable state between tests

### Integration Test Isolation

Integration tests must use **dedicated, ephemeral environments**:
- Test databases created per suite, then destroyed
- Mock services for third-party APIs
- No reliance on external services being available
- Container-based isolation where possible

### Concurrency Safety

Tests must be safe to run in parallel:
- No shared database rows between test cases
- No port conflicts
- No reliance on global state or singletons
- No ordering dependencies between tests

---

## Coverage Priorities

When evaluating coverage gaps, prioritize by risk to autonomous agent work:

| Priority | Area | Rationale |
|---|---|---|
| Critical | Authentication and authorization | Agent changes here can create security vulnerabilities |
| Critical | Payment and financial logic | Incorrect behavior has direct business impact |
| High | Data validation and sanitization | Edge cases here cause data corruption |
| High | API contracts (request or response shapes) | Breaking changes cascade across consumers |
| Medium | UI state management | Bugs are visible but usually recoverable |
| Medium | Error handling paths | Untested error paths produce cryptic failures |
| Lower | Cosmetic or display logic | Low risk, easily caught in review |

---

## Test-First Workflow Requirements

For an agent to follow a test-first workflow, the repository must provide:

1. **Naming conventions:** The agent must be able to infer where to create a test file based on the source file path. Document the pattern, such as `src/foo.ts` to `tests/foo.test.ts`.
2. **Test utilities:** Factories, fixtures, and helpers must be documented or discoverable. An agent should not have to reverse-engineer how to create test data.
3. **Mocking patterns:** The preferred mocking approach must be consistent. Document whether the project uses dependency injection, module mocking, or test doubles.
4. **Single-test execution:** The agent must be able to run exactly one test without running the full suite. Document the command, such as `npx vitest run -t "test name"`.
5. **Assertion style:** Document whether the project uses `expect`, `assert`, or a custom assertion library, and the preferred style for new tests.

---

## CI Integration Checklist

| Requirement | Details |
|---|---|
| Tests run on every PR | Not just on merge to main |
| Results are parseable | Structured output, such as JUnit XML or JSON, is preferred over raw logs |
| Failure blocks merge | Tests are gates, not advisories |
| Flaky test handling | A quarantine mechanism exists so flaky tests do not block unrelated PRs |
| Coverage reporting | Coverage thresholds are enforced where applicable |
| Timeout configuration | Reasonable timeouts prevent hung test suites from blocking CI |
