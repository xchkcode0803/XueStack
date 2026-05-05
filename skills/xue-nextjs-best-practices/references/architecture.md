# XueStack Next.js Architecture

Use feature-driven architecture with Next.js App Router. The goal is separation of concerns: routes deliver the app, features own domains, services own behavior, and lib owns infrastructure.

## Responsibilities

`src/app`

- Route segments, layouts, metadata, loading/error files, and high-level Server Component composition.
- Initial data loading when the data is needed to render a route.
- No reusable business logic.

`src/features/<feature>`

- Domain language, domain UI, actions, services, schema, tests, and feature-local helpers.
- Public contract in `index.ts`.
- Internal files can change without breaking the rest of the app.

`src/components`

- Generic UI primitives and shared design-system components.
- No domain-specific assumptions, data fetching, or product workflow logic.

`src/lib`

- Infrastructure clients and adapters: database client, auth client, analytics client, storage client, AI SDK setup.
- No product decisions such as "create an invoice after checkout"; that belongs in a feature service.

`src/hooks`

- Shared React hooks that are genuinely cross-domain.
- Feature-specific hooks stay inside the owning feature.

## Feature Anatomy

Use this shape when creating a new feature:

```text
src/features/billing/
  components/
    invoice-list.tsx
    plan-selector.tsx
  actions.ts
  services.ts
  schema.ts
  types.ts
  index.ts
  __tests__/
    services.test.ts
    public-api.test.ts
```

Keep the shape proportional. Small features do not need every file.

## Public API Pattern

`index.ts` is the feature gatekeeper. Export only what other parts of the app should use.

```ts
export { PlanSelector } from "./components/plan-selector";
export { createCheckoutSessionAction } from "./actions";
export { getCurrentPlan } from "./services";
export type { BillingPlan } from "./types";
```

Acceptable from another feature or route:

```ts
import { PlanSelector, getCurrentPlan } from "@/features/billing";
```

Avoid from outside the feature:

```ts
import { getCurrentPlan } from "@/features/billing/services";
import { PlanSelector } from "@/features/billing/components/plan-selector";
```

Inside the same feature, direct relative imports are fine.

## App Router Pattern

Route files should compose features and keep route-specific concerns visible.

```tsx
import { getCurrentPlan, PlanSelector } from "@/features/billing";

export default async function BillingPage() {
  const plan = await getCurrentPlan();

  return <PlanSelector currentPlan={plan} />;
}
```

If a route file starts accumulating branching business rules, move those rules into a feature service.

## Server Action Pattern

Actions validate input, call services, and handle Next.js side effects.

```ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";
import { createCheckoutSession } from "./services";

const inputSchema = z.object({
  planId: z.string().min(1),
});

export async function createCheckoutSessionAction(input: unknown) {
  const parsed = inputSchema.parse(input);
  const session = await createCheckoutSession(parsed);
  revalidatePath("/billing");
  return session;
}
```

Services should not import `revalidatePath`, `redirect`, request cookies, or route params unless the repo has an explicit adapter pattern for that dependency.

## Service Pattern

Services own the business behavior and orchestrate persistence or third-party calls.

```ts
import { db } from "@/lib/db";

export async function createCheckoutSession(input: { planId: string }) {
  // Business checks, DB reads/writes, external SDK calls, and transactions live here.
}
```

Prefer dependency injection for services that need heavy external systems and are hard to test directly.
