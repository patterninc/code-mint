# Agent Readiness Checklist

Detailed scoring criteria for each of the eight **readiness dimensions** used only by this audit. These are not the same as the three framework pillars (Legibility, Autonomy, Clarity) in `docs/framework.md`. Score each dimension from Level 0 (Ad Hoc) to Level 4 (Optimized).

This rubric supports the **`Navigate`** north-star outcome in `docs/outcomes.md` by measuring how well the repo is mapped for agent navigation and related hygiene.

---

## Dimension 1: Documentation

| Level | Criteria |
|---|---|
| 0 | No AGENTS.md anywhere. No inline documentation of intent. |
| 1 | Root AGENTS.md exists but is incomplete or inaccurate. |
| 2 | Root AGENTS.md covers project overview, tech stack, and build commands. Some sub-directories have AGENTS.md files. |
| 3 | Directories that exceed the legibility complexity rubric have AGENTS.md files. Both "how" and "why" are documented. Directory map in root is accurate. |
| 4 | Progressive disclosure fully implemented. Hidden assumptions surfaced. UX intent documented alongside every user-facing module. Agent can navigate the entire codebase using only AGENTS.md files. |

## Dimension 2: Build System

| Level | Criteria |
|---|---|
| 0 | No build instructions. Agent cannot determine how to compile or start the project. |
| 1 | Build instructions exist but are incomplete (missing dependencies, environment variables, or platform-specific steps). |
| 2 | Build commands are documented and work from a clean checkout. Dependencies are pinned. |
| 3 | Single-command build. CI/CD pipeline documented. Agent can build and run locally without human intervention. |
| 4 | Deterministic builds. Containerized dev environment. Agent can spin up the full stack autonomously. |

## Dimension 3: Testing

| Level | Criteria |
|---|---|
| 0 | No test suite. Agent cannot verify its own changes. |
| 1 | Some tests exist but are flaky, slow (>5s per suite), or incomplete. No clear instructions on how to run them. |
| 2 | Test suite runs reliably. Test commands documented. Unit tests cover core logic. |
| 3 | Fast unit tests (<200ms per test). Integration tests with isolated environments. Agent can run tests autonomously and interpret results. |
| 4 | Full TDD workflow feasible. Agent can write failing tests, implement fixes, and verify. Visual/snapshot tests for UI. Edge case coverage audited. |

## Dimension 4: Style & Validation

| Level | Criteria |
|---|---|
| 0 | No linting or formatting configuration. |
| 1 | Linter exists but is not enforced. Inconsistent code style across the codebase. |
| 2 | Linting and formatting configured and documented. Single-file validation commands available. |
| 3 | Pre-commit hooks enforce style. Agent can lint individual files without running full project builds. |
| 4 | Style rules are documented in AGENTS.md with rationale. Naming conventions explicit. Auto-fix available for common violations. |

## Dimension 5: Dev Environment

| Level | Criteria |
|---|---|
| 0 | No setup instructions. Agent cannot determine required tools or versions. |
| 1 | Partial setup docs. Missing environment variables, API keys, or infrastructure dependencies. |
| 2 | Setup instructions work from a clean machine. Required tools and versions documented. |
| 3 | Single-command environment setup (e.g., `make setup`, `docker-compose up`). All secrets are sourced from documented vaults or env provisioning scripts (preferred — e.g., a script that pulls secrets from SSM/Vault and writes a local `.env` file). If `.env.example` exists, it is non-secret and clearly secondary to the provisioning script. |
| 4 | Ephemeral, reproducible environments. Agent can spin up isolated sandboxes for experimentation. Sub-second cold starts. |

## Dimension 6: Code Quality

| Level | Criteria |
|---|---|
| 0 | No architectural constraints. Circular dependencies, mixed concerns, no module boundaries. |
| 1 | Some module boundaries exist but are not enforced. Dependency direction is inconsistent. |
| 2 | Clear module boundaries. Dependency direction generally respected. Code review standards exist. |
| 3 | Dependency direction mechanically enforced (Types → Config → Repo → Service → Runtime → UI). Architectural constraints documented in AGENTS.md. |
| 4 | Automated architectural drift detection. Refactoring agents triggered on violations. Golden rules enforced via CI. |

## Dimension 7: Observability

| Level | Criteria |
|---|---|
| 0 | No logging or monitoring. Agent cannot diagnose failures. |
| 1 | Basic logging exists but is inconsistent. No structured log format. |
| 2 | Structured logging in place. Agent can inspect logs to diagnose errors. |
| 3 | Metrics, traces, and structured logs available. Agent can correlate errors across services. Log format documented. |
| 4 | Full observability stack. Agent can autonomously investigate production issues using telemetry. Error budgets and SLOs documented. |

## Dimension 8: Security

| Level | Criteria |
|---|---|
| 0 | No security documentation. Secrets hardcoded or undocumented. |
| 1 | Basic secret management exists. No explicit permission boundaries for agents. |
| 2 | Secrets managed through environment variables, vaults, or env provisioning scripts. Agent permission boundaries partially defined. |
| 3 | Least-privilege policies documented. High-risk actions gated behind human approval. Dependency scanning in CI. |
| 4 | Full agent authorization model. Token vaults with scoped access. Security-sensitive operations require explicit approval workflows. Supply chain risk monitored. |
