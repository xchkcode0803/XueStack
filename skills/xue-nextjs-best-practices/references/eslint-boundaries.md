# ESLint Boundary Rules

Use linting to prevent feature modules from becoming implicit shared folders. Adapt examples to the repo's ESLint version and import alias.

## Simple `no-restricted-imports`

Block cross-feature deep imports while allowing public feature imports:

```js
// eslint.config.mjs
export default [
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      "no-restricted-imports": [
        "error",
        {
          patterns: [
            {
              group: [
                "@/features/*/components/*",
                "@/features/*/services",
                "@/features/*/actions",
                "@/features/*/schema",
                "@/features/*/types",
                "@/features/*/internal/*"
              ],
              message:
                "Import from the feature public API, e.g. '@/features/billing'."
            }
          ]
        }
      ]
    }
  }
];
```

If the repo uses legacy `.eslintrc`, apply the same rule shape under `rules`.

## Local Exceptions

The simple pattern can also flag imports inside a feature if absolute aliases are used internally. Prefer relative imports inside the same feature:

```ts
import { getInvoices } from "./services";
```

If the repo strongly prefers aliases everywhere, add per-feature overrides instead of weakening the global boundary.

## Stronger Option

For larger apps, use `eslint-plugin-boundaries` or `eslint-plugin-import` `no-internal-modules` to express layered architecture. Keep the same policy:

- `app` may import feature public APIs.
- A feature may import shared UI and infrastructure.
- A feature may import another feature only through its public API.
- Shared `components`, `hooks`, and `lib` must not import app routes or feature internals.

## Boundary Test

Add at least one lint fixture or documented failing import when the architecture rule is important to the repo. The goal is to verify that an illegal import such as this fails CI:

```ts
import { chargeCustomer } from "@/features/billing/services";
```
