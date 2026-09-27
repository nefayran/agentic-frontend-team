---
name: component-segregation
description: How to organize React components in a Next.js 15 App Router project — colocation + feature-based segregation with a clear UI / feature / shared boundary. Defines where every component lives, when to promote a component up the hierarchy, and the rules for server/client boundaries. Used by the engineer to keep the codebase navigable and by the reviewer to flag misplacements.
---

# Component Segregation

A large frontend rots when components are dumped into a flat `components/` folder. This skill defines the **layered architecture** every RFP in this training uses.

The model combines two ideas from current best practice:

1. **Colocation** — code that changes together lives together.
2. **Feature-based layering with a UI primitive base** — a hybrid of Feature-Sliced Design and Atomic Design.

---

## The Layers

```
app/                       ← routes (App Router)
└── <route>/
    ├── page.tsx
    ├── layout.tsx
    ├── loading.tsx
    ├── error.tsx
    └── _components/       ← ROUTE-LOCAL components (not reusable)
                             leading underscore = private to this route

features/                  ← FEATURE modules (cross-route, domain-bound)
└── <feature>/
    ├── components/        ← feature-specific components
    ├── hooks/
    ├── api/               ← TanStack Query hooks, fetchers
    ├── schemas/           ← Zod schemas for this feature
    ├── stores/            ← Zustand stores for this feature
    └── types.ts

components/
├── ui/                    ← shadcn primitives (Button, Input, Dialog, ...)
└── shared/                ← cross-feature, non-primitive shared components
                             (e.g. PageHeader, EmptyState, ErrorBoundary)

lib/                       ← cross-cutting, non-React utilities
├── utils/                 ← cn(), formatters, date helpers
├── api/                   ← shared fetcher, API client config
└── auth/                  ← cross-feature auth helpers
```

---

## The Promotion Rule (Where Does a Component Live?)

Decide bottom-up:

| Used by... | Lives in |
|---|---|
| Exactly one route | `app/<route>/_components/` |
| Two or more routes within one feature | `features/<feature>/components/` |
| Multiple features, not a primitive | `components/shared/` |
| Multiple features, primitive UI atom | `components/ui/` (shadcn) |

**Promote, don't pre-place.** Always start at the lowest level (route-local). Move up only when a second consumer appears. Never put a component in `shared/` "just in case".

**Demote when applicable.** If a `shared` component ends up used by only one feature after refactoring, push it down.

---

## What Belongs in Each Layer

### `components/ui/` — Primitives

- shadcn-generated components only.
- No business logic. No data fetching.
- No reference to features, entities, or domain language.
- Props are purely presentational.
- Examples: `Button`, `Input`, `Card`, `Dialog`, `Dropdown`, `Tooltip`, `Skeleton`.

### `components/shared/` — Reusable composites

- Compose UI primitives.
- May know about generic concepts ("page", "user avatar shape") but **not feature-specific entities**.
- No data fetching from a specific endpoint.
- Examples: `PageHeader`, `EmptyState`, `LoadingSpinner`, `ConfirmDialog`, `Avatar`, `CopyButton`.

### `features/<feature>/components/` — Feature components

- Know the feature's entities (`Post`, `Order`, `Patient`).
- May call feature hooks (`usePosts`, `useCreateOrder`).
- May trigger feature stores or server actions.
- Examples (a "posts" feature): `PostList`, `PostCard`, `PostEditor`, `PostFilters`.

### `app/<route>/_components/` — Route-local

- Used by exactly one route.
- Often pages a single feature, with route-specific glue (page layout, route-specific empty state copy).
- Examples: `PostsPageHeader` (specific to `/posts` route), `OnboardingStepThree`.

### `features/<feature>/` (non-component files)

- `api/` — TanStack Query hooks (`usePosts`, `useCreatePost`), server fetchers.
- `schemas/` — Zod schemas (`PostSchema`).
- `hooks/` — feature hooks not tied to a single component.
- `stores/` — Zustand stores scoped to this feature.
- `types.ts` — inferred + auxiliary types.

---

## Server vs Client Components

Independent of the segregation rules above:

- **Default to server**. A new component is a server component until proven otherwise.
- `"use client"` is allowed at any layer, but **push it to the smallest leaf**.
  - ❌ Marking the whole `PostsPage` as client because the filter dropdown is interactive.
  - ✅ Mark only `<PostFilters />` (the dropdown) as client.
- Server components can compose client components freely.
- Client components receive only **serializable** props (no functions from server, no class instances).
- Suffix convention (optional, useful): `.server.ts` for server-only modules (DB calls, secrets). The runtime enforces with `import 'server-only'`.

---

## Component Anatomy (Inside a single component file)

```
features/posts/components/post-card.tsx
```

```tsx
'use client'; // only if needed; otherwise omit

import { type Post } from '@/features/posts/types';
import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils/cn';

interface PostCardProps {
  post: Post;
  onSelect?: (id: string) => void;
  className?: string;
}

export function PostCard({ post, onSelect, className }: PostCardProps) {
  return (
    <article className={cn('rounded-md border bg-card p-4', className)}>
      {/* ... */}
    </article>
  );
}
```

Rules inside a component file:

- One default-ish responsibility per file. If you need to export 3 components, consider whether they belong together.
- Props interface explicit, named `<Component>Props`.
- Use `cn()` for conditional classes.
- Consume semantic Tailwind utilities (`bg-card`, `text-foreground`), never raw colors (see `design-tokens` skill).
- No data fetching inline — call a feature hook (`usePost`) or accept data via props.

---

## Imports & Boundaries

A clean import graph enforces the layering. Allowed directions:

```
app/<route>     → features/* , components/* , lib/*
features/<x>    → components/* , lib/* , features/<x>/* (own feature only)
components/shared → components/ui , lib/utils
components/ui   → lib/utils (only)
lib/*           → lib/* (only)
```

Forbidden:
- ❌ `components/ui/*` importing from `features/*` (a primitive must not know about domain).
- ❌ `features/a/*` importing from `features/b/*` — cross-feature dependency is a smell; lift shared logic to `components/shared/` or `lib/`.
- ❌ `lib/*` importing from React components.

If you find yourself wanting a forbidden import, **the architecture is telling you something**: extract the shared piece.

---

## Naming

- Folders: `kebab-case` (`post-card/`, `user-profile/`).
- Files: `kebab-case.tsx` (`post-card.tsx`).
- React components: `PascalCase` (`PostCard`).
- Hooks: `useCamelCase` (`usePostList`).
- Schemas: `PascalCaseSchema` (`PostSchema`); inferred type `PascalCase` (`type Post = z.infer<...>`).
- Server-only modules: `*.server.ts` (optional, helpful with `import 'server-only'`).

---

## Index Files (`index.ts`) — Use Sparingly

- ✅ Use at the **feature root** to define the public API: `features/posts/index.ts` exports what other features/routes may consume.
- ❌ Do not create barrel `index.ts` files inside every folder — they fragment imports and confuse tree-shaking.

---

## Anti-Patterns (Reject in Review)

- Flat `components/` with 80 files at the same level.
- A `PostCard` living under `components/ui/` (feature component in primitive layer).
- A `Button` living under `features/auth/components/` (primitive locked inside a feature).
- A route-local component imported by another route (should be promoted to feature or shared).
- Cross-feature imports (`features/a/...` ↔ `features/b/...`).
- `'use client'` at the top of a route's `page.tsx` when only one widget is interactive.
- Components doing their own `fetch` instead of calling a feature hook.
- Inline styles or hardcoded Tailwind colors inside a component (see `design-tokens` skill).
- Index barrels at every level.

---

## Definition of Done (Segregation)

- [ ] Every component lives in the lowest layer that satisfies its consumers.
- [ ] No primitive imports from a feature; no feature imports from another feature.
- [ ] No `"use client"` higher than necessary (audit with grep).
- [ ] Every feature has a clear public surface (its `index.ts` or its component exports).
- [ ] No cross-route component duplication (promotion happened where needed).
- [ ] Route-local components live under `app/<route>/_components/` (leading underscore).
