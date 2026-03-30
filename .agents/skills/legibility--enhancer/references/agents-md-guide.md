# AGENTS.md Templates and Examples

Use this guide when creating or reviewing `AGENTS.md` files with `legibility--enhancer`.

## Authoring Principles

- Root files should stay lean: usually 50-100 lines.
- Subdirectory files should stay focused: usually 20-50 lines.
- Prefer concise, high-signal instructions over exhaustive inventory.
- Remove sections or rows that do not apply. Never leave `[CUSTOMIZE]` markers behind.
- Draft from repository evidence first, then ask the human for UX intent, hidden assumptions, and corrections.
- When this `AGENTS.md` governs a scoped directory inside a monorepo (not the git repository root), read the repository root `README.md` and any `README.md` on the path to the scope before drafting, and pull deploy-, CI-, or environment-relevant facts into the scoped file or explicit cross-references (same posture as `legibility--enhancer` Step 1).

## Canonical Root Structure

The root file should contain:

1. Project name and one-paragraph overview
2. Scope statement telling the agent which directory the file governs
3. Tech stack
4. Commands
5. Directory map
6. Project-wide conventions
7. Related repositories
8. Optional infrastructure context
9. Optional agent-permission boundaries and merge rules

Use the root template in [root-agents-md-template.md](root-agents-md-template.md) as the canonical structure.

### Required Vs Optional Root Content

- Required:
  - project overview
  - scope statement
  - tech stack
  - commands that the agent will actually need
  - directory map
  - project-wide conventions
- Optional, include only when useful:
  - related repositories
  - infrastructure context
  - agent permissions
  - merge rules

## Root AGENTS.md Template

```markdown
# [CUSTOMIZE: Project Name]

[CUSTOMIZE: One-paragraph description of what this repository does. Be specific and operational, not promotional.]

Apply these instructions to [CUSTOMIZE: full path to the repository root or scoped project/directory this AGENTS.md governs]. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

[CUSTOMIZE: Keep this file lean. Remove optional rows or sections that do not apply.]

## Tech Stack

- **Language:** [CUSTOMIZE]
- **Framework:** [CUSTOMIZE]
- **Database:** [CUSTOMIZE]
- **Infrastructure:** [CUSTOMIZE]
- **Testing:** [CUSTOMIZE]

## Commands

[CUSTOMIZE: Include exact commands. Remove rows that do not apply. The smoke-test row should name the smallest safe runtime check.]

| Action | Command |
|---|---|
| Install dependencies | `[CUSTOMIZE]` |
| Start local dependencies | `[CUSTOMIZE: optional]` |
| Build | `[CUSTOMIZE: optional]` |
| Run dev server | `[CUSTOMIZE: or main local startup command]` |
| Smoke test runtime | `[CUSTOMIZE: e.g., after services are up, curl http://localhost:3000/healthz]` |
| Run all tests | `[CUSTOMIZE]` |
| Run single test file | `[CUSTOMIZE: optional]` |
| Lint | `[CUSTOMIZE]` |
| Lint single file | `[CUSTOMIZE: optional]` |

## Directory Map

| Directory | Purpose |
|---|---|
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |

## Conventions

- [CUSTOMIZE: project-wide rule]
- [CUSTOMIZE]
- [CUSTOMIZE]

## Related Repositories

[CUSTOMIZE: Optional. Include only if companion repositories matter.]

| Repository | Purpose | Clone URL |
|---|---|---|
| [CUSTOMIZE] | [CUSTOMIZE] | [CUSTOMIZE] |

## Infrastructure

[CUSTOMIZE: Optional. Describe local runtime target, IaC visibility, CI, and deployment boundaries.]

- Local runtime target: [CUSTOMIZE]
- IaC visibility: [CUSTOMIZE]
- CI pipeline: [CUSTOMIZE]
- Deployment: [CUSTOMIZE]

## Agent Permissions

[CUSTOMIZE: Optional but recommended when autonomy boundaries matter.]

- **Autonomous:** [CUSTOMIZE]
- **Supervised:** [CUSTOMIZE]
- **Restricted:** [CUSTOMIZE]

### Merge Rules

Agents may merge PRs when the branch is unprotected or all required approvals and checks have passed. Never:

- Use `--admin` to override a blocked merge
- Force-push to `main` or `dev`
- Skip CI checks
```

## Subdirectory AGENTS.md Template

Create a subdirectory file when a module has meaningful UX impact, non-obvious rules, repeated gotchas, or needs context beyond the root `AGENTS.md`.

Use the subdirectory template in [subdirectory-agents-md-template.md](subdirectory-agents-md-template.md) as the canonical structure.

### Required Vs Optional Subdirectory Content

- Required:
  - module name
  - scope statement
  - UX intent
  - key files
- Optional:
  - local conventions
  - gotchas

If a section adds no value, remove it instead of padding it with generic text.

```markdown
# [CUSTOMIZE: Module Name]

[CUSTOMIZE: One-sentence description of what this module does technically.]

Apply these instructions to [CUSTOMIZE: full path to the directory this AGENTS.md governs]. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

## UX Intent

[CUSTOMIZE: What does the user experience when this module works correctly? What breaks if it fails or changes incorrectly? What promises does it make?]

## Key Files

| File | Role |
|---|---|
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |

## Local Conventions

[CUSTOMIZE: Optional. Repeat or reference any root rule that still matters here because subdirectory files override parent files.]

## Gotchas

[CUSTOMIZE: Optional. Known edge cases, failure modes, or easy ways to break the module.]
```

## Example: Root AGENTS.md for an E-Commerce App

```markdown
# ShopFast

A headless e-commerce API serving the ShopFast storefront. It handles catalog, cart, checkout, and order management for the storefront and internal operations tooling.

Apply these instructions to `/workspace/shopfast-api`. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

## Tech Stack

- **Language:** TypeScript 5.4
- **Framework:** Express 4.x
- **Database:** PostgreSQL 16 (primary), Redis 7 (sessions/cache)
- **Infrastructure:** Docker Compose (local), AWS ECS (production)
- **Testing:** Vitest, Supertest

## Commands

| Action | Command |
|---|---|
| Install dependencies | `npm install` |
| Start local dependencies | `docker compose up -d postgres redis` |
| Build | `npm run build` |
| Run dev server | `npm run dev` |
| Smoke test runtime | `curl -f http://localhost:3000/healthz` |
| Run all tests | `npm test` |
| Run single test file | `npx vitest run tests/cart/cart.test.ts` |
| Lint | `npm run lint` |
| Lint single file | `npx eslint src/api/cart.ts` |

## Directory Map

| Directory | Purpose |
|---|---|
| `src/api/` | REST API route handlers and middleware |
| `src/services/` | Business logic for cart, pricing, inventory, and checkout |
| `src/repositories/` | Database queries via Kysely |
| `src/models/` | Type definitions and Zod schemas |
| `src/config/` | Environment config and feature flags |
| `tests/` | Test suites mirroring `src/` |

## Conventions

- Dependency direction: Models → Config → Repositories → Services → API. Never import upstream.
- All API responses use `{ data, error, meta }`.
- Database access only through repository modules.
- All monetary values are stored as integer cents, never floats.
- Feature flags are checked in the service layer, not route handlers.

## Related Repositories

| Repository | Purpose | Clone URL |
|---|---|---|
| `shopfast-tf` | Terraform infrastructure for ECS, RDS, ElastiCache, ALB, IAM, and networking | `git@github.com:acme/shopfast-tf.git` |
| `shopfast-storefront` | Next.js frontend that consumes this API | `git@github.com:acme/shopfast-storefront.git` |

## Infrastructure

- Local runtime target: Docker Compose provides PostgreSQL and Redis locally.
- IaC visibility: Terraform lives in `shopfast-tf` and can be inspected read-only.
- CI pipeline: GitHub Actions runs lint, test, and build on every PR.
- Deployment: Merges to `main` auto-deploy to staging.

## Agent Permissions

- **Autonomous:** Read files, run tests, run linters, create branches.
- **Supervised:** Change API contracts, modify CI, open PRs.
- **Restricted:** Run production migrations, deploy to production, modify payment-provider credentials.
```

## Example: Subdirectory AGENTS.md for a Checkout Module

```markdown
# Checkout Module

Orchestrates cart validation, payment processing, inventory reservation, and order creation.

Apply these instructions to `/workspace/shopfast-api/src/services/checkout`. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

## UX Intent

This module directly controls the purchase experience. If it fails silently, users may lose their cart or be charged without receiving an order. Every checkout error must be clear and actionable, and the flow should stay fast enough that users do not abandon the purchase.

## Key Files

| File | Role |
|---|---|
| `checkout.service.ts` | Orchestrates the full checkout flow |
| `payment.adapter.ts` | Wraps the payment provider SDK |
| `inventory.lock.ts` | Reserves inventory during checkout with TTL |
| `order.repository.ts` | Persists completed orders |

## Local Conventions

- All payment operations are idempotent and must use idempotency keys.
- Inventory locks expire after 10 minutes if checkout is not completed.
- Repeat the root error-handling rule here: user-facing failures must stay actionable.

## Gotchas

- The payment provider webhook can arrive before the checkout API call returns.
- If Redis is down, checkout must fail gracefully instead of proceeding without an inventory lock.
- Currency conversion belongs in the pricing service, not this module.
```
