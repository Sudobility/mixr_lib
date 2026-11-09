# MIXR Lib

> **Git policy — never auto-commit or auto-push.** Leave your work in the working tree.
> Run `git commit`, `git push`, `gh pr create`, or `scripts/push_all.sh` **only when the user
> explicitly asks in that turn**. Approval for an earlier change does not carry forward, and
> finishing a task is not permission to commit it.

Business logic library for MIXR with shared utilities, types, and hooks. ESM-only (`"type": "module"`), consumed by the MIXR frontend.

**Package**: `@sudobility/mixr_lib` — private, scoped, published to **GitHub Packages** (`npm.pkg.github.com`, see `.npmrc`), not public npm. License BUSL-1.1.

## Tech Stack

- **Language**: TypeScript
- **Runtime**: Bun
- **Package Manager**: Bun (do not use npm/yarn/pnpm for installing dependencies)
- **Build**: TypeScript compiler (tsc)
- **Test**: Vitest + React Testing Library + happy-dom

## Project Structure

```
src/
├── index.ts              # Public exports
├── business/
│   └── index.ts          # initializeMixrLib, MIXR_LIB_VERSION
├── hooks/
│   └── index.ts          # React hooks
├── types/
│   └── index.ts          # Re-exports from mixr_types
├── utils/
│   └── index.ts          # Utility functions
├── test/
│   └── setup.ts          # Vitest setup (cleanup, localStorage mock)
└── __tests__/
    ├── business/
    ├── hooks/
    ├── types/
    └── utils/
```

Each `src/<area>/index.ts` is a single barrel module; `src/index.ts` re-exports the public surface. `_resetMixrLib()` in `business/index.ts` is `@internal`, for tests only.

## Commands

```bash
bun run build        # tsc -p tsconfig.build.json + scripts/fix-esm-imports.mjs
bun run build:watch  # tsc --watch
bun run clean        # Remove dist/
bun run test:unit    # Run Vitest once   (NOTE: no `test` script exists)
bun run test:watch   # Run Vitest in watch mode
bun run lint         # Run ESLint (src, .ts/.tsx)
bun run typecheck    # tsc --noEmit
bun run format       # Format with Prettier
bun run verify       # typecheck + lint + test:unit + build
```

## Exports

### Business Logic
- `initializeMixrLib(client: MixrClient)` - Initialize the library with a client instance
- `isMixrLibInitialized()` - Check if the library has been initialized
- `getMixrClient()` - Get the stored MixrClient instance
- `MIXR_LIB_VERSION` - Current version string

### Types (re-exported from mixr_types)
All entity types, request/response types, and constants from `@sudobility/mixr_types`.

### Utilities
- `formatMixrData(data)` - JSON pretty-printer (handles circular references)
- `validateMixrInput(input)` - Basic non-empty string validation (trims whitespace)
- `validateDisplayName(name)` - Display name length validation
- `validateStarRating(stars)` - Star rating range validation (1-5, integer)
- `validateReviewText(review)` - Review text length validation

### Recipe Helpers
- `formatIngredientList(ingredients)` - Format ingredients with amounts as readable strings
- `formatRecipeSteps(steps)` - Number recipe steps (e.g., "1. Muddle mint...")
- `formatRatingDisplay(aggregate)` - Format rating aggregate as display string
- `getIngredientCount(recipe)` - Count ingredients in a recipe
- `getStepCount(recipe)` - Count steps in a recipe
- `getEquipmentNames(recipe)` - Extract unique equipment names from a recipe

### Constants
- `MIN_STAR_RATING`, `MAX_STAR_RATING` - Star rating range (1-5)
- `MAX_REVIEW_LENGTH` - Maximum review text length (2000)
- `MIN_DISPLAY_NAME_LENGTH`, `MAX_DISPLAY_NAME_LENGTH` - Display name constraints (3-50)
- `DEFAULT_PAGE_LIMIT`, `MAX_PAGE_LIMIT` - Pagination defaults (20, 100)
- `DEFAULT_RATING_SORT` - Default rating sort order ("newest")

### Hooks
- `useMixrLibPlaceholder(initialValue)` - Example hook pattern

## Usage

```typescript
import {
  initializeMixrLib,
  formatMixrData,
  validateMixrInput,
  MIXR_LIB_VERSION,
} from '@sudobility/mixr_lib';

import type { Equipment, Recipe, Mood } from '@sudobility/mixr_lib';
```

## Peer Dependencies

(from `package.json` — keep this list in sync with it)

- `@sudobility/di` ^1.5.85
- `@sudobility/mixr_types` ^0.0.31
- `@sudobility/types` ^1.9.78
- `@tanstack/react-query` >= 5.0.0
- `react` >= 18.0.0
- `zustand` >= 5.0.0

## Architecture

This library sits between the client (API hooks) and the app (UI):

```
mixr (frontend)
    └── mixr_lib (this package - business logic, types, utils)
        └── mixr_client (API hooks)
            └── mixr_types (types)
```

## Publishing

Publishing is automated by CI (`.github/workflows/ci-cd.yml`, which calls the reusable `johnqh/workflows` unified-cicd with `publish-target: github`) on pushes to `main`/`develop`. Manual: `bun run prepublishOnly` (clean + build) then publish to GitHub Packages. `zustand`, `@tanstack/react-query`, and `@sudobility/di` are declared peer deps but not yet imported anywhere in `src/` (only `@sudobility/mixr_client`, `@sudobility/mixr_types`, `@sudobility/types`, and `react` are actually used).

## Gotchas

- **`MIXR_LIB_VERSION`** in `src/business/index.ts` is a hardcoded string (`'0.0.14'`) and is **stale** vs `package.json` (`0.0.52`); its "kept in sync" comment is aspirational. Nothing auto-updates it.
- **ESM post-build rewrite is mandatory**: `tsc` uses `moduleResolution: "bundler"` and emits extensionless relative imports that Node's ESM loader rejects. `scripts/fix-esm-imports.mjs` rewrites them to explicit `.js` paths after every build — never ship `dist/` without it.
- **`tsconfig.build.json` relaxes strictness** (`noUnusedLocals`, `exactOptionalPropertyTypes`, `noUncheckedIndexedAccess`, etc. off) vs the strict `tsconfig.json` used by `typecheck`. Code can build but fail `typecheck`.
- **Tests need a GitHub Packages auth token** to install the `@sudobility/*` peer deps (`.npmrc`).

## Git Workflow

- Do not use feature branches for code changes. Always stay on the current branch.
