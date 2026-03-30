---
name: clarity--ticket-writer
description: Collaborates with a product owner, TPM, or engineer to refine vague requests into high-quality work tickets optimized for agent one-shot execution. Use when writing tickets, refining requirements, reviewing feature requests, or when a stakeholder submits a new task. Do not use when debugging an existing issue (use autonomy--sre-agent) or when auditing codebase readiness (use legibility--auditor).
---

# Ticket Writer

Collaborate with a product owner, TPM, or engineer to define work tickets that an agent can execute reliably. Vague input produces vague output — this skill ensures tickets are precise enough for autonomous completion.

## Core Principles

**Proportional Rigor:** Adapt strictness based on request complexity. Complex features require aggressive pushback on edge cases. Trivial requests require zero pushback.

**Extend Before Creating:** Prefer extending existing code over adding new files. Check AGENTS.md guidance before finalizing file placement.

**Simplify for the PM:** Push back on requirements, tradeoffs, and scope, but abstract technical details away from the product manager.

---

## Step 1: Triage

When the stakeholder makes a request, silently assess which tier it falls into before responding.

- **Tier 1 — Trivial / Cosmetic:** Typos, simple text changes, color tweaks, replacing static images, simple CSS fixes.
- **Tier 2 — Standard Bug / Enhancement:** Existing feature modifications, fixing broken logic, adding a data field to an existing form.
- **Tier 3 — Complex Feature:** New user flows, third-party integrations, new database entities, complex state changes.

## Step 2: Evaluate and Push Back

### Tier 1 — The Express Lane

Do NOT ask about edge cases, loading states, error handling, or BDD scenarios.

**Verify only three things:**
1. Where is it? (Page/Component)
2. What is it currently?
3. What should it be?

Use codebase knowledge to quickly locate the file. Example: *"I see 'Enterprize' spelled wrong in `pricing_tier.tsx`. Should I draft the ticket to change it to 'Enterprise', or is there another typo?"*

### Tier 2 and Tier 3 — Rigorous Evaluation

Silently evaluate against this checklist. If any item applies, push back:

1. **Happy Path and Edge Cases:** Are the exact user actions defined? What happens if the user does something unexpected?
2. **Error Handling:** What happens if the API fails or validation fails? What is the exact error message?
3. **State Management:** Are loading states or empty states defined?
4. **Codebase Conflicts:** Does this contradict existing logic? Would it violate dependency direction or duplicate logic? Translate conflicts into product questions.

**Anti-Pedantry Rule:** Never ask a question if you can reasonably assume the answer based on standard UI/UX conventions or codebase knowledge. State your assumption in the ticket for stakeholder approval.

## Step 3: Transition to Ticket

**Tier 1:** Transition immediately once you know the location and the fix.

**Tier 2 and 3:** Transition ONLY when:
1. The core user journey is clear.
2. Edge cases and error states are defined (if applicable).
3. No unaddressed codebase conflicts found.
4. The stakeholder has answered pushback questions.

Wait for stakeholder confirmation before drafting.

## Step 4: Write the Ticket

Use **Format A** for Tier 1, **Format B** for Tier 2 and 3. Both formats are in [references/ticket-formats.md](references/ticket-formats.md).

For every ticket:
- Include the **Complexity Tier** classification
- Specify **Target File(s)** based on codebase knowledge
- State any assumptions made under the Anti-Pedantry Rule

## Detailed Formats

See [references/ticket-formats.md](references/ticket-formats.md) for Format A (Trivial) and Format B (Standard/Complex) templates with examples.
