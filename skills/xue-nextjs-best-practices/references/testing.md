# Testing Strategy

Test the architectural boundary at the same level where the behavior lives.

## Service Unit Tests

Service tests verify business behavior without rendering Next.js routes.

- Mock or inject external clients when practical.
- Cover success, validation failure, authorization failure, and important edge cases.
- Keep database-backed tests integration-style if the service depends heavily on SQL behavior.

Example target:

```text
src/features/billing/__tests__/services.test.ts
```

## Public API Integration Tests

Test what other parts of the app are allowed to use: the feature public API.

```text
src/features/billing/__tests__/public-api.test.ts
```

Useful checks:

- Required exports exist.
- Exported service behavior works through the public entrypoint.
- Internal helpers are not exported accidentally.

## Server Action Tests

Action tests should focus on edge behavior:

- Input validation.
- Auth and authorization checks.
- Calls to the intended service.
- Revalidation, redirect, or returned action state.

Avoid duplicating service business-logic tests in action tests.

## Boundary Tests

When the repo enforces feature boundaries with ESLint, verify the lint command fails on illegal deep imports. This can be done with a fixture or by ensuring CI runs the configured lint rule.

Illegal from outside a feature:

```ts
import { createCheckoutSession } from "@/features/billing/services";
```

Legal:

```ts
import { createCheckoutSession } from "@/features/billing";
```

## Route Tests

Routes should need fewer tests when they mostly compose feature APIs. Add route tests when route behavior includes permissions, redirects, metadata, loading/error states, or user-visible integration behavior.
