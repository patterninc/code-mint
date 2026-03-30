# Runtime Smoke-Test Guide

Use this guide when a repository needs a documented smoke path that an agent or developer can follow without improvising.

## Goal

A runtime smoke test is the smallest non-destructive sequence that proves the local runtime is meaningfully usable.

In an outcome-first onboarding flow, the smoke path is not just a nice-to-have. It is the proof artifact for the `Smoke Path` outcome.

A good smoke path:

- is safe to run in a local or isolated environment
- verifies something more meaningful than "the process started"
- has explicit prerequisites
- has a clear pass signal
- says when to stop and ask for help

## Discovery Questions

If the repository does not answer these directly, ask the developer in freeform form:

1. What is the smallest runtime check your team actually trusts?
2. Which services must be up before that check is meaningful?
3. What readiness signals show those services are usable?
4. Which env values are required, and which can be replaced locally?
5. Are migrations, seed data, or a dev account required?
6. Would the smoke path trigger external side effects?
7. Which step is usually flaky or poorly documented today?

## Recommended Shape

Prefer this order:

1. Start local dependencies
2. Verify dependency readiness
3. Start the app or service
4. Run one app-level confidence check
5. Record the success evidence

## Proof Template

Record the smoke-path evidence in `docs/onboarding-checklist.md` using a note like:

```markdown
### Outcome: Smoke Path
- Status: Proven
- Date: [YYYY-MM-DD]
- Exact Check: [command or browser walkthrough]
- Evidence: [screenshot, curl output, log path, or short note]
- What Passed: [smallest trusted confidence signal]
- What Is Still Missing: [manual steps, flaky dependencies, approval gates]
- Next Action: [single next improvement]
```

Do not mark `Smoke Path` as `Proven` until the evidence exists.

## Smoke Patterns By Runtime Type

### API Service

- Dependencies: database, cache, queue, or mock service
- App check: health endpoint plus one read-only request
- Success evidence: HTTP 200, expected response shape, and no fatal startup errors

Example:

```text
1. docker compose up -d postgres redis
2. curl -f http://localhost:3000/healthz
3. curl -f http://localhost:3000/api/products?limit=1
```

### Web App

- Dependencies: API, database, mock auth, or asset pipeline
- App check: load one core page in the browser
- Success evidence: page renders, critical element appears, no fatal console or server errors

Example:

```text
1. docker compose up -d postgres
2. npm run dev
3. Open http://localhost:3000
4. Confirm the dashboard heading or login screen renders
```

### Worker Or Background Service

- Dependencies: queue, database, scheduler, or blob store
- App check: start the worker and verify it connects to required dependencies
- Success evidence: startup log line, queue consumer ready signal, or health endpoint if one exists

### Multi-Service Stack

- Dependencies: start in dependency order
- App check: verify each dependency first, then run one end-to-end check through the main entrypoint
- Success evidence: all required services healthy and one user-visible or API-visible flow succeeds

## Safe Vs Unsafe Smoke Tests

Safe smoke tests:

- use local containers, mocks, seed data, or isolated dev resources
- avoid destructive writes unless they are clearly ephemeral
- avoid billing, email, SMS, webhooks, or third-party side effects
- avoid shared staging state unless the team explicitly approves it

Stop and ask for approval if the only available smoke path would:

- modify shared data or long-lived environments
- send external notifications
- hit real payment, auth, or third-party callbacks
- require production-like secrets that are not intended for agent use
- run migrations or seeds whose safety is unclear

## What To Record

A documented smoke path should include:

- prerequisites
- startup order
- exact commands or browser steps
- expected readiness signals
- expected pass signal
- "do not run unless" notes for risky steps
- what remains manual if the path is only partially autonomous
