---
applyTo: "**/*.ts,**/*.tsx,**/*.jsx"
---

# TypeScript / React Standards

- **TypeScript required** for all new code. No `.js` files except build configs.
- **Formatter:** Prettier via `pnpm format`. Always run after editing.
- **Naming:** `camelCase` for variables/functions/properties/hooks. `PascalCase` for components/classes/types/interfaces/enum members. `UPPER_SNAKE_CASE` for constants.
- **File names:** `kebab-case.ts` for utilities, `PascalCase.tsx` for React components (follow directory convention).
- **Types:** Interfaces over `type` aliases for object shapes. Discriminated unions for status branching. Interfaces mirror API schemas with camelCase field names.
- **React:** Functional components with hooks only. `React.FC` only when children needed. Derived state → custom hooks. Style via CSS modules or design system.
- **Exports:** Named exports preferred. Default exports only for Next.js pages/layouts.
- **Error handling:** Never swallow errors. Use Error boundaries. `try/catch` must log or surface.
- **Null handling:** Prefer `undefined` over `null`. Use `?.` and `??`.
- **No manual casing translation.** Use framework-native serialization.
- **HTML/CSS:** Semantic HTML. Keyboard accessibility. `alt` on images. CSS Modules preferred. No `!important`.

# UI / SDK Build Checklist

## SDK Interface Sync
- When adding methods to the `I4GClient` interface in `packages/sdk/src/index.ts`, **always** add matching stubs to `createMockClient()` in `packages/sdk/src/__fixtures__/index.ts`. The build fails otherwise.
- When adding Zod schemas, add the corresponding `export type` and wire the schema into the relevant client method.

## Union Type Safety
- Before using a string literal as a component prop variant (e.g., `Badge variant=`), **check the component source** for the actual union type. Common mistake: using `"secondary"` when `BadgeVariant` only defines `"default" | "success" | "warning" | "danger" | "info"`.
- Never guess variant values — read the type definition.

## Zod Schema ↔ API Parity
- SDK Zod schemas must match what the FastAPI backend actually returns. `curl` the endpoint to verify field names and types before writing a new schema.
- After changing a Zod schema, search `packages/sdk/src/__fixtures__/` for affected field names and update test fixtures.

## Pre-Commit Verification
Before staging UI changes for merge, run in order:
1. `pnpm format` — Prettier formatting
2. `pnpm lint` — catches unused imports, undeclared vars
3. `pnpm build` — catches type errors, missing mock stubs, invalid union values

All three must pass. Do not rely on editor diagnostics alone — `pnpm build` is the source of truth.