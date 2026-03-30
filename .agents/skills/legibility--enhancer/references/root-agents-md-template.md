# [CUSTOMIZE: Project Name]

[CUSTOMIZE: One-paragraph description of what this repository does. Be specific — this is not a marketing blurb. State what the system does, who uses it, and what its primary responsibilities are.]

Apply these instructions to [CUSTOMIZE: full path to the repository root or scoped project/directory this AGENTS.md governs]. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

[CUSTOMIZE: Keep this file lean. Include only project-wide guidance that an agent needs often. Remove optional rows or sections that do not apply instead of leaving placeholder text behind.]

## Tech Stack

- **Language:** [CUSTOMIZE: e.g., TypeScript 5.x, Python 3.12, Go 1.22]
- **Framework:** [CUSTOMIZE: e.g., Next.js 14, FastAPI, Gin]
- **Database:** [CUSTOMIZE: e.g., PostgreSQL 16, MongoDB 7, Redis 7]
- **Infrastructure:** [CUSTOMIZE: e.g., AWS ECS, Vercel, Docker Compose, Kubernetes]
- **Testing:** [CUSTOMIZE: e.g., Vitest, pytest, Go test]

## Commands

[CUSTOMIZE: Include exact commands. Remove rows that truly do not apply. For the smoke-test row, include the smallest safe runtime check plus any short prerequisite note if needed.]

| Action | Command |
|---|---|
| Install dependencies | `[CUSTOMIZE]` |
| Start local dependencies | `[CUSTOMIZE: optional, e.g., docker compose up -d postgres redis]` |
| Build | `[CUSTOMIZE: optional]` |
| Run dev server | `[CUSTOMIZE: or the main local startup command]` |
| Smoke test runtime | `[CUSTOMIZE: e.g., after services are up, curl http://localhost:3000/healthz]` |
| Run all tests | `[CUSTOMIZE]` |
| Run single test file | `[CUSTOMIZE: optional, e.g., npx vitest run path/to/test.ts]` |
| Lint | `[CUSTOMIZE]` |
| Lint single file | `[CUSTOMIZE: optional, e.g., npx eslint path/to/file.ts]` |

## Directory Map

| Directory | Purpose |
|---|---|
| `[CUSTOMIZE]` | [CUSTOMIZE: What this directory contains and is responsible for] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |

## Conventions

[CUSTOMIZE: List 3-7 project-wide conventions. These should be rules that apply everywhere in the codebase — not module-specific rules. Examples:]

- [CUSTOMIZE: e.g., Dependency direction: Types → Config → Repository → Service → Runtime → UI. Never import upstream.]
- [CUSTOMIZE: e.g., All API responses use the envelope pattern: `{ data, error, meta }`.]
- [CUSTOMIZE: e.g., Database access only through repository modules, never directly from handlers.]

## Related Repositories

[CUSTOMIZE: List companion repositories the agent may need for full context. Include IaC, shared libraries, frontend/backend counterparts, and documentation repos. Provide the clone URL so the agent can access them.]

| Repository | Purpose | Clone URL |
|---|---|---|
| [CUSTOMIZE: e.g., `myapp-tf`] | [CUSTOMIZE: e.g., Terraform infrastructure (ECS, RDS, Redis, IAM, networking)] | [CUSTOMIZE: e.g., `git@github.com:org/myapp-tf.git`] |
| [CUSTOMIZE: e.g., `myapp-frontend`] | [CUSTOMIZE: e.g., React frontend that consumes this API] | [CUSTOMIZE: e.g., `git@github.com:org/myapp-frontend.git`] |

[CUSTOMIZE: Remove rows that don't apply. Add rows for any repository an agent would need to reference when debugging, inspecting infrastructure, deploying, or understanding the full system. If the primary access path is read-only GitHub inspection instead of cloning, say that explicitly.]

## Infrastructure

[CUSTOMIZE: Optional but recommended when runtime or infrastructure context matters. Describe what environment is available to agents working in this repo. Clarify what can be run locally, what can be inspected read-only, and what requires human approval. Remove this section if it adds no useful information.]

- Local runtime target: [CUSTOMIZE: e.g., Docker Compose spins up PostgreSQL and Redis locally]
- IaC visibility: [CUSTOMIZE: e.g., Terraform lives in `myapp-tf`; agents may inspect it read-only via clone or `gh`]
- CI pipeline: [CUSTOMIZE: e.g., GitHub Actions runs lint, test, and build on every PR]
- Deployment: [CUSTOMIZE: e.g., Merges to main auto-deploy to staging]

## Agent Permissions

[CUSTOMIZE: Recommended whenever the repository allows command execution, infra inspection, or deployment-related work. Define what agents can do autonomously vs. what requires human approval. Remove this section only if a different file already documents the same boundaries clearly.]

- **Autonomous:** [CUSTOMIZE: e.g., Read files, run tests, run linters, create branches]
- **Supervised:** [CUSTOMIZE: e.g., Open PRs, modify API contracts, change database schemas]
- **Restricted:** [CUSTOMIZE: e.g., Deploy to production, run migrations, modify auth logic]

### Merge Rules

Agents may merge PRs when the branch is unprotected or all required approvals and checks have passed. Never:

- Use `--admin` to override a blocked merge (causes compliance violations)
- Force-push to `main` or `dev`
- Skip CI checks (`--no-verify`, etc.)

If a merge is blocked by branch policy, report the block and provide the PR URL for human review.
