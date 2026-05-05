---
name: xue-nextjs-best-practices
description: Apply XueStack Next.js architecture and code quality rules. Use when building, reviewing, or refactoring Next.js App Router apps, especially for feature-driven structure, domain boundaries, server actions, service layers, modular testing, Drizzle/Postgres integration when present, and anti-slop scalable code organization.
---

# Xue Next.js Best Practices

Use this skill to keep Next.js applications modular, testable, and scalable. Treat Next.js as the delivery mechanism and keep business behavior in feature modules with explicit public APIs.

## Core Workflow

1. Inspect the existing app structure before changing it.
2. Identify the feature or domain being built or changed.
3. Put route files, layouts, metadata, and high-level composition in `src/app`.
4. Put domain UI, actions, services, schemas, tests, and feature-specific helpers in `src/features/<feature>`.
5. Export only the feature contract from `src/features/<feature>/index.ts`.
6. Keep Server Actions thin: validate input, call service functions, revalidate or redirect, and return typed results.
7. Keep service functions framework-light: put business rules, persistence calls, transactions, and third-party orchestration there.
8. Add or update tests around service behavior, feature public APIs, and import boundaries when the change has architectural risk.

## Architecture Defaults

Use this shape for new Next.js App Router projects unless the repo already has a coherent alternative:

```text
src/
  app/                 # routes, layouts, loading/error files, RSC composition
  components/          # shared generic UI only
  features/            # domain modules
    <feature>/
      components/      # feature-specific UI
      actions.ts       # server actions: validation + service calls
      services.ts      # business logic and persistence orchestration
      schema.ts        # feature-owned DB schema when applicable
      index.ts         # public feature API
  hooks/               # shared React hooks only
  lib/                 # infrastructure clients and low-level adapters
```

Do not force a large refactor when working in an existing repo. Apply the architecture at the smallest useful boundary and keep local conventions where they are already consistent.

## Boundary Rules

- Import other features only through `@/features/<feature>`.
- Do not deep import from another feature's `components`, `services`, `actions`, `schema`, or internal helpers.
- Keep `index.ts` exports explicit and small. This is a feature contract, not a broad bundle barrel.
- Allow direct imports inside the same feature when they improve clarity.
- Keep global `components` generic. If a component knows domain language, move it into the owning feature.
- Keep `lib` infrastructure-focused. Do not put product workflows or domain decisions there.

## Server Rules

- Use Server Components to fetch data for initial render and compose feature UI.
- Use Server Actions for mutations and form submissions.
- Put validation at the edge of the action, then call a service.
- Use `revalidatePath`, `revalidateTag`, redirects, or typed action results intentionally after mutations.
- Authenticate and authorize Server Actions like API endpoints.
- Pass minimal serialized data from Server Components into Client Components.

## Data Rules

Use Drizzle/Postgres guidance only when the repo already uses it or the user requests it. Do not introduce a database stack just because this skill mentions one.

When Drizzle/Postgres is present:

- Keep the connection/client in `src/lib` or the repo's existing infrastructure location.
- Prefer feature-local table definitions when the feature owns the data.
- Provide a central schema aggregation point only for Drizzle tooling and migrations.
- Use transactions for multi-step writes and explicit isolation levels for sensitive flows such as billing, inventory, credits, or account balances.

## Resource Map

Read these references only when needed:

- `references/architecture.md`: detailed feature layout, responsibilities, and import examples.
- `references/eslint-boundaries.md`: ESLint patterns for preventing cross-feature deep imports.
- `references/drizzle-postgres.md`: conditional Drizzle/Postgres structure and transaction guidance.
- `references/testing.md`: test strategy for services, feature APIs, and boundaries.

## Done Criteria

Before finishing a Next.js architecture task, verify:

- Route files stay thin and mostly compose feature APIs.
- New domain behavior lives under an owning feature.
- Cross-feature imports use public `index.ts` contracts.
- Server Actions do not contain business workflows.
- Existing conventions are respected unless they conflict with modularity or testability.
- Validation steps are run or clearly reported as not run.
