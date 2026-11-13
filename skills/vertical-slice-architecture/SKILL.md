---
name: vertical-slice-architecture
description: Prefer feature-centric architecture with Vertical Slice for AI-assisted systems while avoiding dogmatic over-engineering. Use when defining project structure, deciding architecture fit, refactoring layered code, or implementing slices that blend cleanly with Domain-Driven Design, Hexagonal Architecture, and layered conventions. Host repo conventions (AGENTS.md/CLAUDE.md, ADRs, architecture gates) always take precedence over this skill's example layouts.
---

# Vertical Slice Architecture

## Overview

Use Vertical Slice as the default architecture bias for AI-assisted development. Optimize for change isolation, clear ownership, and deterministic implementation boundaries. Apply DDD and Hexagonal ideas where they add clarity, not ceremony.

Treat this as a default, not a law. Choose simpler structure when scope is tiny or short-lived. When a repo adopts this bias, record the decision in its own decision log (`docs/tech/ADR.md`, `adr/`, or equivalent) — the skill is not the policy source for any specific project.

## Host Conventions Win

This skill provides reasoning, boundaries, and a workflow — not a mandatory layout. The host repo's own rules (AGENTS.md, CLAUDE.md, its ADRs, and any architecture gate such as `mago guard` or deptrac) are always authoritative for the concrete shape of a slice. Before creating files:

- If the repo already defines a slice layout, mirror the closest existing slice exactly — same folder depth, same file roles, same vocabulary. Do not introduce roles the repo does not use (no `command`/`repository` files where the convention is Controller + Request + Handler).
- If the repo has no architecture conventions, the structures below are reasonable defaults.
- When this skill and a repo convention disagree, the repo wins and the disagreement is worth an ADR, not a silent override.

## Default Position

Prefer Vertical Slice first because:

- Reduce blast radius by localizing changes per feature/use case.
- Reduce ambiguity by mapping code to business behavior.
- Improve review quality by keeping request flow in one slice.
- Improve testability with slice-level seams and explicit dependencies.
- Improve migration safety by moving one workflow at a time.

## Run Fit Check

Before adopting or enforcing slice, check fit:

- Use slice by default when product lifespan, domain complexity, or team size is medium to high.
- Start simpler when app is tiny, mostly CRUD, or intentionally disposable.
- Add Hexagonal boundaries early when external integrations change frequently.
- Add deeper DDD modeling only where business rules are dense and volatile.

If most checks are weak, keep structure minimal and revisit after real complexity appears.

## Implement Vertical Slice

1. Read the host's conventions (AGENTS.md/CLAUDE.md, existing slices, architecture gates) and mirror the closest existing slice before inventing structure.
2. Define one use case as one slice boundary.
3. Place request mapping, validation, orchestration, domain behavior, and persistence coordination inside that slice.
4. Keep cross-slice dependencies explicit and minimal.
5. Keep dependency direction inward toward slice application/domain logic.
6. Write tests at slice boundaries first, then inside critical rules.
7. Extract shared code only after repeated, proven duplication.

Use a structure similar to:

```text
# Minimal shape (typical MVC / server-rendered app)
features/
  orders/
    create/
      CreateOrderController   # endpoint + validation + orchestration
      CreateOrderRequest
      create-order.view

# Richer shape (complex domain, many use-case collaborators)
features/
  orders/
    create/
      endpoint
      command
      handler
      domain
      repository
      tests
```

The slice boundary is the feature/use-case folder. Its internal file roles are a host-convention decision (see Host Conventions Win): a single controller file is a valid slice, and so is a full command/handler/domain set.

## Mix Patterns Safely

Apply combinations deliberately:

- Vertical Slice + DDD: Keep ubiquitous language and aggregate rules close to the slice use case.
- Vertical Slice + Hexagonal: Define ports at slice boundaries and adapters near each slice, not in one global adapter sink.
- Vertical Slice + Layered conventions: Allow small local layers inside each slice, but avoid global horizontal feature code buckets.

Read `references/mixed-patterns.md` for detailed boundaries, dependency rules, and folder options.

## Prevent Common Failures

Watch for:

- Slice theater: folders look vertical but logic is still centralized and tangled.
- Cargo-cult DDD: entities and value objects without real domain language.
- Adapter explosion: premature interfaces for stable dependencies.
- Hidden monolith layer: overgrown `shared/` that bypasses slice boundaries.

Read `references/anti-patterns.md` for symptom -> risk -> fix guidance.

## Adopt Incrementally

Do not do big-bang rewrites. Migrate progressively:

1. Start new features as slice.
2. Move one high-change legacy flow at a time.
3. Add architecture guardrails in code review and automated checks.
4. Measure cycle time, defect rate, and coupling trends.

Read `references/adoption-playbook.md` for a staged rollout.

## Verification Checklist

Confirm before closing architecture work:

- Each new behavior maps to a clear slice.
- Dependencies point inward and avoid cross-slice leakage.
- Shared code exists only for proven reuse.
- Ports and adapters exist where volatility justifies them.
- Tests cover slice boundary behavior and key domain rules.
- Naming matches domain language and use-case intent.
