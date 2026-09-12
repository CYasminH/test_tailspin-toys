---
description: 'Comment philosophy, documentation standards, and code formatting conventions'
applyTo: '**/*.{ts,tsx,js,astro,css}'
---

# Coding Standards & Documentation Guidelines

This document establishes clear, consistent coding standards across the Tailspin Toys codebase. Clear documentation and intentional comments make the codebase easier to understand, maintain, and extend — for both humans and AI assistants like Copilot.

## Comment Philosophy

### Comment Intent, Not Mechanics

Write comments that explain **why** code exists and the reasoning behind non-obvious decisions, not what the code does. Readers can understand *what* by reading the code itself.

**❌ Poor comment (restates the code):**
```typescript
// Increment the counter
counter++;

// Check if the rating is not null
if (rating !== null) {
  // Calculate the average
  total += rating;
}
```

**✅ Good comment (explains reasoning):**
```typescript
// Increment counter to track how many games have star ratings
counter++;

// Only include explicit ratings; null indicates the title didn't yield a deterministic rating
if (rating !== null) {
  // Sum ratings for the average calculation
  total += rating;
}
```

### When to Comment

- **Non-obvious logic or decisions** — explain the reasoning if it's not immediately clear from the code
- **Trade-offs or constraints** — document why a particular approach was chosen over alternatives
- **Complex algorithms or formulas** — explain the steps and intent
- **Workarounds or edge cases** — explain why the workaround exists and what it solves
- **Integration points** — document dependencies between modules or expected calling patterns

### When NOT to Comment

- Don't restate what the code obviously does
- Don't comment every line of straightforward code
- Don't leave comments that describe old code or outdated behavior
- Outdated comments are bugs — update or delete them in the same commit that touches the related code

### Example: Good and Bad Comments

**❌ Bad:**
```typescript
// Sort games by title
const sorted = games.sort((a, b) => a.title.localeCompare(b.title));
```

**✅ Good:**
```typescript
// Ensure deterministic ordering across static builds — ordered by title so the page output is consistent
const sorted = games.sort((a, b) => a.title.localeCompare(b.title));
```

## Documentation Standards

### TypeScript/JavaScript: TSDoc/JSDoc for Exported Functions

Every exported function in `db/` and `src/lib/` must have a TSDoc/JSDoc comment describing its purpose, parameters, and return value. This ensures clarity for callers and keeps the testing patterns visible.

**Format:**
```typescript
/**
 * Brief description of what the function does.
 * 
 * Additional context if the behavior is non-obvious.
 * @param paramName - Description of the parameter and its significance
 * @returns Description of the return value and when it might be null/undefined
 */
export async function myFunction(paramName: string): Promise<string | null> {
  // implementation
}
```

**Example from the data layer:**
```typescript
/**
 * Retrieve all games ordered alphabetically by title.
 * 
 * Used at build time by pages to fetch the full game catalog.
 * @param db - The injectable database client (allows tests to use in-memory DBs)
 * @returns Array of all games in deterministic order
 */
export async function getAllGames(db: Database): Promise<Game[]> {
  const rows = await baseGamesQuery(db).orderBy(asc(games.title));
  return rows.map(mapGame);
}
```

**Example with complex parameters:**
```typescript
/**
 * Create a Drizzle database client from a local SQLite connection.
 * 
 * Resolves the given URL to a file path, creates the directory if needed,
 * and bridges the async Drizzle adapter to Node's synchronous SQLite driver.
 * @param url - Local file URL (e.g., 'file:tailspin.db' or ':memory:' for tests)
 * @returns Configured Drizzle client ready for queries
 */
export function createDatabase(url: string = process.env.DATABASE_URL ?? DEFAULT_DATABASE_URL): Database {
  return createDatabaseConnection(url).db;
}
```

### Astro Components: Props Interface Documentation

Each reusable `.astro` component must document its `Props` interface so the component API is self-explanatory. Include descriptions for each prop, type constraints, and defaults.

**Format:**
```astro
---
interface Props {
  /** Description of what this prop controls. */
  propName: string;
  /** Optional prop with a default; explain what happens if omitted. */
  optional?: boolean;
  /** When to use this variant and what it controls visually. */
  variant?: 'solid' | 'gradient';
}
---
<!-- component template -->
```

**Example:**
```astro
---
import type { HTMLAttributes } from 'astro/types';

interface Props extends Omit<HTMLAttributes<'button'>, 'type'> {
  /** Visual style of the button (solid fills vs. gradient backgrounds). */
  variant?: 'solid' | 'gradient';
  /** Padding scale (sm for compact, md for default). */
  size?: 'sm' | 'md';
  /** When provided, renders as an <a> tag; omit to render as a <button>. */
  href?: string;
  /** Button type when rendered as a <button> element. */
  type?: 'button' | 'submit' | 'reset';
  /** Stretch the button to fill its container width. */
  fullWidth?: boolean;
}

const {
  variant = 'solid',
  size = 'md',
  href,
  type = 'button',
  fullWidth = false,
  class: className,
  ...rest
} = Astro.props;
---
<!-- template -->
```

## TypeScript Formatting Rules

### Type Annotations

- **Explicit function signatures**: All exported functions must have explicit parameter types and return types. This is especially important in the data layer (`db/`, `src/lib/`).
  
  ```typescript
  // ✅ Good
  export async function getGameById(db: Database, id: number): Promise<Game | null> {
    // ...
  }
  
  // ❌ Avoid
  export async function getGameById(db, id) {
    // ...
  }
  ```

- **Use `type` for type-only aliases** (not `interface` for external-facing types):
  ```typescript
  export type Game = {
    id: number;
    title: string;
    description: string;
  };
  ```

- **Use `interface` for Astro Props and extensible contracts**:
  ```typescript
  interface Props {
    variant?: string;
  }
  ```

### Import/Export

- Group imports by category: external libraries, internal modules, types
  ```typescript
  // External
  import { eq, asc } from 'drizzle-orm';
  
  // Internal
  import type { Database } from './db';
  import { games, categories } from '../../db/schema';
  
  // Types
  import type { Game } from '../types/game';
  ```

- Use relative paths for same-directory or parent imports; use absolute paths from `src/` for clarity
  ```typescript
  // ✅ Clear: absolute path from src/
  import { getDatabase } from '../../lib/db';
  
  // Also acceptable: relative when in same directory
  import { mapGame } from './mappers';
  ```

### Naming

- **Exported functions/types**: descriptive, verb-first for actions (`getAllGames`, `getGameById`)
- **Private helpers**: prefix with `_` or keep in a separate scope if needed
- **Constants**: UPPER_SNAKE_CASE for module-level constants
  ```typescript
  const DEFAULT_DATABASE_URL = 'file:tailspin.db';
  const GAME_CACHE_TIMEOUT = 3600; // seconds
  ```

- **Variables**: camelCase
  ```typescript
  const gameData = { /* ... */ };
  let userInput = '';
  ```

## Keeping Comments Current

Treat outdated comments as bugs. When you modify code:

1. **Update related comments** in the same commit
2. **Delete comments** that no longer describe the current behavior
3. **Add new comments** if your change adds non-obvious logic
4. **Review comments** in nearby code to catch stale documentation

**Example: Before and After**

Before:
```typescript
// Fetch all games from the API
const rows = await db.select({ id: games.id }).from(games);
```

After (API changed to local SQLite):
```typescript
// Fetch all game IDs from the local SQLite database, ordered by title for deterministic builds
const rows = await db.select({ id: games.id }).from(games).orderBy(asc(games.title));
```

## ESLint & TypeScript Rules

The project uses **ESLint** to enforce code quality and **TypeScript** for type safety. All code must pass linting before commit.

### Running Linting

```bash
npm run lint
```

This checks all TypeScript and Astro files against the rules in `eslint.config.js`.

### Key ESLint Rules

- **No unused variables** (except those prefixed with `_`)
- **No implicit `any` types** (TypeScript strict mode)
- **Consistent naming** for imports and exports
- **No console.log in production code** (use only for debugging, remove before commit)

### Type Checking

Type checking runs separately and is required before merging:

```bash
npm run typecheck           # TypeScript 7 (tsgo) for pure TS
npm run typecheck:astro    # Astro type checking for .astro files
npm run typecheck:all      # Both of the above
```

## Documentation Checklist

Before committing code, verify:

- [ ] All exported functions in `db/` and `src/lib/` have TSDoc comments with `@param` and `@returns`
- [ ] All Astro components with `Props` interfaces have documented each prop
- [ ] Comments explain *why*, not *what*
- [ ] No comments restate the code
- [ ] All comments are current and accurate
- [ ] `npm run lint` passes without errors
- [ ] `npm run typecheck:all` passes without errors
- [ ] README is updated if behavior or API changed

## Examples by File Type

### `db/` and `src/lib/` Files (TypeScript with TSDoc)

```typescript
import { asc, eq } from 'drizzle-orm';
import type { Database } from './db';
import { games, categories, publishers } from '../../db/schema';
import type { Game } from '../types/game';

const gameSelection = {
  id: games.id,
  title: games.title,
  description: games.description,
  starRating: games.starRating,
  categoryId: categories.id,
  categoryName: categories.name,
  publisherId: publishers.id,
  publisherName: publishers.name,
};

type GameSelectionRow = {
  id: number;
  title: string;
  description: string;
  starRating: number | null;
  categoryId: number | null;
  categoryName: string | null;
  publisherId: number | null;
  publisherName: string | null;
};

/**
 * Map a raw database row to the app-facing Game type.
 * 
 * Handles null relationships gracefully so pages can render partial data.
 * @param row - Raw database selection
 * @returns Typed Game object with resolved relationships
 */
function mapGame(row: GameSelectionRow): Game {
  return {
    id: row.id,
    title: row.title,
    description: row.description,
    starRating: row.starRating,
    category:
      row.categoryId !== null && row.categoryName !== null
        ? { id: row.categoryId, name: row.categoryName }
        : null,
    publisher:
      row.publisherId !== null && row.publisherName !== null
        ? { id: row.publisherId, name: row.publisherName }
        : null,
  };
}

/**
 * Build the base games query with category and publisher joins.
 * 
 * Used internally by getAllGames and getGameById to avoid duplication.
 * @param db - The injectable database client
 * @returns Drizzle query (not yet executed)
 */
function baseGamesQuery(db: Database) {
  return db
    .select(gameSelection)
    .from(games)
    .leftJoin(categories, eq(games.categoryId, categories.id))
    .leftJoin(publishers, eq(games.publisherId, publishers.id));
}

/**
 * Retrieve all games ordered by title.
 * 
 * Deterministic ordering ensures static builds produce consistent output.
 * Used at build time by pages to fetch the full game catalog.
 * @param db - The injectable database client
 * @returns Array of all games in alphabetical order
 */
export async function getAllGames(db: Database): Promise<Game[]> {
  const rows = await baseGamesQuery(db).orderBy(asc(games.title));
  return rows.map(mapGame);
}

/**
 * Retrieve a single game by its ID.
 * 
 * @param db - The injectable database client
 * @param id - The game's primary key
 * @returns The Game if it exists, or null if not found
 */
export async function getGameById(db: Database, id: number): Promise<Game | null> {
  const row = await baseGamesQuery(db).where(eq(games.id, id)).get();
  return row ? mapGame(row) : null;
}
```

### Astro Components (with documented Props)

```astro
---
import type { HTMLAttributes } from 'astro/types';

/**
 * A flexible button component that can render as a <button> or <a> tag.
 * 
 * Supports multiple visual styles (solid, gradient) and sizes (sm, md).
 * Includes focus and hover states for accessibility and modern UX.
 */
interface Props extends Omit<HTMLAttributes<'button'>, 'type'> {
  /** Visual style of the button: 'solid' fills with a single color, 'gradient' uses a multi-color gradient. */
  variant?: 'solid' | 'gradient';
  /** Padding scale: 'sm' for compact spacing, 'md' for default/recommended. */
  size?: 'sm' | 'md';
  /** When provided, the button renders as an <a> tag with this href; omit to render as <button>. */
  href?: string;
  /** HTML button type (only used when href is not provided and rendered as <button>). */
  type?: 'button' | 'submit' | 'reset';
  /** When true, button stretches to fill the width of its container. */
  fullWidth?: boolean;
}

const {
  variant = 'solid',
  size = 'md',
  href,
  type = 'button',
  fullWidth = false,
  class: className,
  ...rest
} = Astro.props;

const base =
  'inline-flex items-center justify-center rounded-lg font-medium transition-all duration-300 focus:ring-2 focus:ring-blue-500 focus:outline-none';

const variants: Record<NonNullable<Props['variant']>, string> = {
  solid: 'bg-blue-600 hover:bg-blue-700 text-white',
  gradient:
    'bg-gradient-to-r from-blue-600 to-purple-600 hover:from-blue-500 hover:to-purple-500 text-white',
};

const sizes: Record<NonNullable<Props['size']>, string> = {
  sm: 'px-4 py-2 text-sm',
  md: 'px-6 py-3',
};

const classes = [base, variants[variant], sizes[size], fullWidth && 'w-full', className];
---

{
  href ? (
    <a href={href} class:list={classes} {...rest}>
      <slot />
    </a>
  ) : (
    <button type={type} class:list={classes} {...rest}>
      <slot />
    </button>
  )
}
```

## Summary

- **Comments**: Explain *why*, not *what*. Keep them current.
- **Documentation**: Export functions in `db/` and `src/lib/` need TSDoc. Astro components need Props documentation.
- **TypeScript**: Explicit types for all exported functions and parameters.
- **Linting**: `npm run lint` and `npm run typecheck:all` must pass before every commit.
- **Outdated comments**: Treat as bugs and fix or remove in the same commit.
