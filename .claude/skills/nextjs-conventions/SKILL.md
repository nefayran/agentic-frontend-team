---
name: nextjs-conventions
description: Conventions for building Next.js 15 (App Router) apps in this training environment. Covers folder structure, server vs client components, data fetching, state, forms, styling, and common patterns. Load this before writing any product code.
---

# Next.js Conventions

These are the conventions every engineer in this training follows. Deviation requires the architect's written approval in `architecture.md`.

**Companion skills (load alongside this one):**
- `component-segregation` — where every component lives and how layers connect.
- `design-tokens` — three-tier token system + Tailwind v4 wiring.

---

## Folder Structure

```
<rfp-slug>/
├── app/                          # routes (App Router)
│   ├── <route>/
│   │   ├── page.tsx              # server component by default
│   │   ├── layout.tsx
│   │   ├── loading.tsx           # route-level skeleton
│   │   ├── error.tsx             # route-level error boundary
│   │   └── _components/          # route-local components (underscore = private)
│   ├── api/                      # route handlers (only if not server actions)
│   ├── layout.tsx                # root layout
│   ├── not-found.tsx
│   └── globals.css               # token CSS variables + Tailwind v4 @theme
├── features/                     # FEATURE modules (cross-route, domain-bound)
│   └── <feature>/
│       ├── components/           # feature-specific components
│       ├── hooks/
│       ├── api/                  # TanStack Query hooks + fetchers
│       ├── schemas/              # Zod schemas
│       ├── actions/              # server actions (if any)
│       ├── stores/               # Zustand stores
│       ├── types.ts
│       └── index.ts              # public surface of the feature
├── components/
│   ├── ui/                       # shadcn primitives ONLY
│   └── shared/                   # cross-feature composite components
├── lib/
│   ├── utils/                    # cn(), formatters, date helpers
│   ├── api/                      # shared fetcher + API client config
│   └── auth/                     # cross-feature auth helpers
├── tokens/                       # DTCG JSON source (compiled into globals.css)
├── e2e/                          # Playwright specs
├── playwright.config.ts
├── tailwind.config.ts            # may be minimal — most config lives in @theme in CSS
├── tsconfig.json
└── package.json
```

Naming: kebab-case for folders and files, PascalCase for component exports, camelCase for hooks and utilities.

**Promotion rule** (see `component-segregation` for full detail):
- 1 route uses it → `app/<route>/_components/`
- 2+ routes in same feature → `features/<feature>/components/`
- 2+ features, composite → `components/shared/`
- 2+ features, UI primitive → `components/ui/` (shadcn)

---

## Server vs Client Components

- **Default to server.** A new `page.tsx` or component is a server component unless proven otherwise.
- Mark a component `"use client"` only when it needs:
  - `useState` / `useReducer` / `useEffect`
  - event handlers (onClick, onChange) on the rendered element
  - browser APIs (window, localStorage)
  - third-party client-only libraries
- Push `"use client"` to the smallest leaf possible. Don't wrap an entire page; wrap the interactive subtree.
- Server components can import client components freely. The reverse passes serializable props only.

---

## Data Fetching

### Reads (server components)

```ts
// app/posts/page.tsx
import { getPosts } from '@/lib/api/posts.server';

export default async function PostsPage() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}
```

The server fetcher (`*.server.ts`) parses the response with Zod and throws on schema failure.

### Reads (client components)

Use TanStack Query. Query keys are arrays starting with the entity name:

```ts
// lib/api/posts.ts
export const postsKeys = {
  all: ['posts'] as const,
  list: (filter: Filter) => [...postsKeys.all, 'list', filter] as const,
  detail: (id: string) => [...postsKeys.all, 'detail', id] as const,
};

export function usePosts(filter: Filter) {
  return useQuery({
    queryKey: postsKeys.list(filter),
    queryFn: () => fetchPosts(filter), // parses with Zod inside
  });
}
```

### Writes

Use **server actions** by default. Route handlers (`app/api/.../route.ts`) only when an external client (mobile, third-party) calls the same endpoint.

```ts
// lib/actions/posts.ts
'use server';

import { CreatePostSchema } from '@/lib/schemas/post';

export async function createPost(formData: FormData) {
  const parsed = CreatePostSchema.parse(Object.fromEntries(formData));
  // ...
  revalidatePath('/posts');
}
```

---

## Validation (Zod)

- Every external boundary (API response, form input, server action input, URL params) is parsed.
- Schemas live in `lib/schemas/` and are imported by both client and server.
- Infer TS types from the schema, don't duplicate:

```ts
export const PostSchema = z.object({ id: z.string(), title: z.string() });
export type Post = z.infer<typeof PostSchema>;
```

---

## State Management

Decision tree:

1. Does this state come from the server? → **TanStack Query**.
2. Should it survive a page refresh / be linkable? → **URL search params**.
3. Is it shared across routes or deeply nested? → **Zustand store** in `stores/`.
4. Otherwise → **local `useState` / `useReducer`**.

Never store server data in Zustand. Never read the same value from two sources.

---

## Forms

React Hook Form + Zod resolver. shadcn `Form` components wrap RHF cleanly.

```tsx
const form = useForm<z.infer<typeof Schema>>({
  resolver: zodResolver(Schema),
  defaultValues: { ... },
});

async function onSubmit(values: z.infer<typeof Schema>) {
  await createPostAction(values);
}
```

- Validate on blur + submit by default.
- Show inline errors associated via `aria-describedby`.
- Disable submit button while pending; show pending state.

---

## Styling

- **Tailwind v4** with the CSS-first `@theme inline` directive. Most config lives in `app/globals.css`, not `tailwind.config.ts`.
- **All design values come from tokens** — see the `design-tokens` skill. Compile `docs/<rfp>/tokens/` into CSS custom properties at `:root` (light) and `.dark`, and expose them to Tailwind via `@theme inline`.
- Consume semantic utilities only: `bg-background`, `text-foreground`, `bg-primary`, `rounded-md`. **Never** hardcoded values:
  - ❌ `bg-blue-500`, `text-[#1A73E8]`, `mt-[18px]`, `rounded-[10px]`
  - ✅ `bg-primary`, `text-foreground`, `mt-4`, `rounded-md`
- shadcn primitives live in `components/ui/` and consume the same tokens — never override their colors with literal Tailwind palette classes.
- Use `cn()` from `lib/utils/cn.ts` for conditional classes.
- No inline `style` except for genuinely dynamic values (e.g. computed transforms).

---

## UI States — Always Five

For every data-driven screen, build all five:

1. **Empty** — first-time user / no results.
2. **Loading** — skeleton matching the loaded layout.
3. **Error** — friendly message + retry.
4. **Success** — primary content.
5. **Partial** — paginated / streamed / partially loaded.

If the design spec doesn't define one of these, **stop and ask the designer** — don't guess.

---

## Accessibility

- Always real semantic elements: `<button>`, `<a href>`, `<input>`, `<label>`.
- Form labels associated via `htmlFor` (RHF + shadcn `FormLabel` handles this).
- Icon-only buttons: include visually hidden text or `aria-label`.
- Focus management: when a modal opens, focus the first interactive element; on close, return focus to the trigger.
- Never `outline: none` without an equivalent `:focus-visible` style.

---

## Error Handling

- Every async path: try/catch with a user-visible error (toast or inline) — never swallow.
- Every route segment that fetches data has an `error.tsx`.
- Use `notFound()` from `next/navigation` for missing resources.
- Log unexpected errors (server-side) with enough context to debug.

---

## Performance Defaults

- `next/image` for every image, with explicit dimensions or `fill`.
- `next/font` for every font — never `<link>` Google Fonts.
- Dynamic import (`next/dynamic`) for heavy client-only components (charts, editors, maps).
- Avoid client-side `useEffect` for data fetching — prefer server components or TanStack Query.
- Cache server fetches with `fetch(..., { next: { revalidate: N } })` where appropriate.

---

## Testing Hooks for the Engineer

- Prefer accessible queries; that means accessible markup is free.
- If a Playwright test requires a `data-testid`, it usually means the markup isn't semantic — fix the markup first.

---

## What This Skill Doesn't Cover

- Specific business logic (it's per-RFP, lives in the BA/architect/designer artifacts).
- Deployment (out of scope for training).
- Authentication strategy (architect picks per RFP).

When in doubt, read the artifacts under `docs/<rfp-slug>/` — they are the source of truth, this skill is just the style guide.
