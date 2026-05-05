# Drizzle/Postgres Guidance

Use this only when the repo already uses Drizzle/Postgres or the user requests that stack. Do not introduce Drizzle/Postgres as a default dependency for every Next.js app.

## Connection Location

Keep the database client in infrastructure, usually `src/lib/db.ts` or the repo's existing equivalent. In development, protect against connection churn caused by hot reloads by reusing a global client when the selected Postgres driver supports it.

Use the repo's existing driver conventions. Drizzle can be paired with different Postgres clients, so avoid mixing driver styles unless the task is explicitly a migration.

## Feature Schemas

When a feature owns a table, keep its schema close to the feature:

```text
src/features/billing/schema.ts
src/features/accounts/schema.ts
```

Provide a central aggregation file only where tooling needs one:

```ts
// src/lib/db/schema.ts
export * from "@/features/accounts/schema";
export * from "@/features/billing/schema";
```

Treat this aggregation file as infrastructure for Drizzle tooling, not as permission for app code to bypass feature APIs.

## Query Ownership

Put feature-owned queries in `services.ts` or a feature-local repository file if the service becomes too large.

```text
src/features/billing/
  services.ts
  repository.ts
  schema.ts
```

Keep repository helpers feature-private unless there is a strong reason to expose them through `index.ts`.

## Transactions

Use transactions for multi-step writes that must succeed or fail together:

```ts
await db.transaction(async (tx) => {
  await createInvoice(tx, input);
  await decrementCredits(tx, input.accountId);
});
```

Use explicit isolation for sensitive workflows when supported by the configured driver and Drizzle version:

- Billing, balances, credits, inventory, and entitlement changes often need stricter isolation.
- Prefer idempotency keys for externally triggered payment or webhook flows.
- Keep webhook verification at the delivery edge, then call feature services with verified data.

## Cross-Feature Data

If one feature needs another feature's data, prefer calling the other feature's public service instead of importing its table or repository directly. Importing another feature's schema is acceptable in central migration/schema aggregation files.
