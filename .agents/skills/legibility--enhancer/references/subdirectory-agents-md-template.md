# [CUSTOMIZE: Module Name]

[CUSTOMIZE: One-sentence description of what this module does technically.]

[CUSTOMIZE: Save this file with the exact filename `AGENTS.md`. Do not use `agents.md` or `Agents.md`. Remove this line after customization.]

Apply these instructions to [CUSTOMIZE: full path to the directory this AGENTS.md governs]. Treat paths and commands below as relative to that location unless explicitly stated otherwise.

[CUSTOMIZE: Keep this file concise and specific. Prefer a few strong statements about UX intent, key files, and gotchas over a long inventory. Remove sections that do not apply.]

## UX Intent

[CUSTOMIZE: What user-facing behavior depends on this module? Answer these questions:
- What does the user see or experience when this module works correctly?
- What breaks for the user if this module fails or changes incorrectly?
- What implicit promises does this module make? (e.g., response time, data consistency, error clarity)]

## Key Files

| File | Role |
|---|---|
| `[CUSTOMIZE]` | [CUSTOMIZE: What this file is responsible for] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |
| `[CUSTOMIZE]` | [CUSTOMIZE] |

## Local Conventions

[CUSTOMIZE: Rules specific to this module that differ from or supplement the root AGENTS.md. Because subdirectory AGENTS.md files override parent files, repeat or explicitly reference any root rules that still matter here. If there are no module-specific conventions, state "Follows root conventions" and remove this section. Examples:]

- [CUSTOMIZE: e.g., All database queries go through the repository layer, never called directly from handlers.]
- [CUSTOMIZE: e.g., All payment operations must be idempotent.]

## Gotchas

[CUSTOMIZE: Non-obvious constraints, known edge cases, or things that have broken before. If there are no known gotchas, remove this section. Examples:]

- [CUSTOMIZE: e.g., The webhook can arrive BEFORE the API call returns. Handlers must be idempotent.]
- [CUSTOMIZE: e.g., Currency conversion happens in the pricing service, not here. Do not add currency logic to this module.]
- [CUSTOMIZE: e.g., Cache invalidation requires clearing both Redis and the CDN. Clearing only one will cause stale data.]
