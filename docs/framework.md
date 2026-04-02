# Harness Engineering Foundations

Harness engineering is the discipline of designing environments, repo knowledge, and feedback loops so an AI agent can execute work reliably. The work shifts from "write the code by hand" to "make the intended behavior legible, verifiable, and recoverable."

This repository is organized around three pillars: **Legibility** (navigation), **Autonomy** (verification and operational tooling within safe boundaries), and **Clarity** (executable plans and tickets once the core loop is proven). The six outcomes in `docs/outcomes.md` are the concrete proof model.

## Principle 1: Legibility — Make The Codebase AI-Legible

If the agent cannot discover the structure, purpose, and boundaries of the repository from in-repo artifacts, it will guess. Durable repo knowledge matters more than chat explanations.

### What Legibility Requires

- a short root `AGENTS.md` that acts as a map, not an encyclopedia
- subdirectory `AGENTS.md` files for complex or high-risk modules
- references and examples that live in-repo rather than in people's heads
- explicit UX intent for modules that affect user-visible behavior

### Progressive Disclosure

Structure context in layers:

- **Root `AGENTS.md`**: project overview, commands, directory map, conventions
- **Subdirectory `AGENTS.md`**: local purpose, UX intent, key files, gotchas
- **Reference docs**: deeper examples, history, schemas, and guides

### Maintenance Rule

Any change to structure, key commands, conventions, or module boundaries should update the relevant `AGENTS.md` files. Stale guidance is worse than missing guidance because it teaches the agent the wrong thing confidently.

## Principle 2: Autonomy — Make Self-Verification Possible

An agent that can only write code is not enough. It needs trusted ways to tell whether the repository is working now, whether a change passed the right tests, and whether a runtime path is meaningfully usable. Autonomy here includes tests and smoke paths, and—where applicable—using operational CLIs for evidence; it does not require full parity with production.

### What Self-Verification Requires

- baseline audits that describe the current state before remediation
- fast targeted test execution for the code that changed
- isolated or safe environments for verification
- one trusted, non-destructive smoke path
- explicit approval boundaries for higher-risk actions

### The Required Proof Loops

The user should be able to point to concrete evidence for each north-star outcome. Names, proof criteria, and primary skills are defined only in [`docs/outcomes.md`](outcomes.md); in recommended order they are: `Validate Current State`, `Navigate`, `Self-Test`, `Smoke Path`, `Bug Reproduction`, and `SRE Investigation`.

### Agent Authorization Tiers

Document three permission tiers in the root `AGENTS.md`:

1. **Autonomous**: safe read, test, and local verification steps
2. **Supervised**: actions that require review before they take effect
3. **Restricted**: actions that require explicit approval every time

Never leave these boundaries implicit.

## Principle 3: Clarity — Plans And Tickets That Survive Contact With The Codebase

Once the repo is legible and self-verifiable, debugging workflows and ticket refinement multiply the value of the rest of the system.

### What Clarity Requires

Clarity compounds **after** the core outcomes in `docs/outcomes.md` are in place. It is not a substitute for proving `Self-Test`, `Bug Reproduction`, or `SRE Investigation`; use the evidence paths those outcomes establish for tests, repros, and operational signals.

- reproduce-before-fix discipline for bugs (fixes tie back to evidence the repo can rerun)
- a workshop pattern for refining vague requests into executable tickets (`clarity--ticket-writer`)
- a habit of turning lessons learned into repo-local documentation or rules

### Ticket quality matters

Work tickets are part of the harness. A vague ticket produces vague output. The goal is not to ask more questions than necessary, but to ask the few questions that surface missing edge cases, assumptions, and success criteria.

## Outcome Map

Outcome names, proof criteria, and primary skill mappings are defined in `docs/outcomes.md`. Track progress and evidence in `docs/onboarding-checklist.md`. `.agents/code-mint-status.json` provides a machine-readable index of outcome statuses for cross-repo scanning.

## Cross-Cutting Standards

These standards keep the repo legible as agent throughput increases.

### Dependency Direction

Dependencies should flow one way:

```
Types → Config → Repository → Service → Runtime → UI
```

Agents are fast enough to create structural drift quickly. Clear dependency boundaries reduce that risk.

### Extend Before Creating

Before creating a new file, the agent should:

1. look for an existing home for the logic
2. check local `AGENTS.md` guidance
3. place any new file in the correct module structure

### Single Source Of Truth

Keep types, config, business rules, and database access in one clear layer each. Duplicated logic creates contradictions that both humans and agents will keep reinforcing.

### Error Handling

- user-facing errors should be clear and actionable
- internal failures should include enough context to debug
- error handling patterns should be consistent and documented
- swallowed errors and generic catch-all behavior should be treated as drift

## Mechanical Enforcement

Documentation alone is not enough. The best harnesses gradually promote important guidance into tooling:

- `AGENTS.md` for discoverability
- rules for persistent context
- linters for naming and architecture constraints
- structural tests for dependency boundaries and coverage expectations
- recurring cleanup work that catches drift before it spreads

When prose keeps getting ignored, encode the constraint directly into the system.
