---
name: quality-checklist
description: Comprehensive quality checklist for a Next.js frontend going from 0 to production-ready. Used primarily by the code-reviewer agent as a final audit, and as a self-check by the engineer before handing off to QA. Walk every item, mark pass/fail, and cite file:line for any failure.
---

# Quality Checklist

Walk this top-to-bottom. For every item, mark **✅ pass** / **❌ fail** / **N/A** with a one-line note. A failure is not a "todo for later" — it goes into the review report with a severity.

---

## 1. Build & Type Safety

- [ ] `pnpm build` succeeds with no errors and no warnings.
- [ ] `pnpm tsc --noEmit` is clean.
- [ ] TypeScript strict mode enabled (`strict: true`, `noUncheckedIndexedAccess: true` recommended).
- [ ] Zero `any` in the codebase (grep `: any` and `as any`).
- [ ] Zero `@ts-ignore` / `@ts-expect-error` without an inline justification comment.
- [ ] All public component props have explicit TypeScript interfaces.
- [ ] All exported functions have explicit return types where non-trivial.

## 2. Data Layer

- [ ] Every API response is parsed through a Zod schema before use.
- [ ] Schemas live in `lib/schemas/` and are reused between server and client.
- [ ] TanStack Query keys follow a documented convention (e.g. `['entity', id, params]`).
- [ ] No server state is duplicated into Zustand or React context.
- [ ] Mutations invalidate the correct query keys (or use optimistic updates correctly).
- [ ] No `fetch` calls outside the data layer (`lib/api/`).

## 3. State Management

- [ ] Server state → TanStack Query only.
- [ ] URL state (filters, pagination, tabs) → `useSearchParams`, not local state.
- [ ] Client-only shared state → Zustand store in `stores/` with typed selectors.
- [ ] Local-only state → `useState` / `useReducer`, no global store overuse.
- [ ] Forms use React Hook Form + Zod resolver.

## 4. Rendering & Performance

- [ ] Server components by default; `"use client"` only at the smallest necessary leaf.
- [ ] Images use `next/image` with explicit width/height or `fill`.
- [ ] Fonts use `next/font`.
- [ ] Heavy client components are dynamically imported.
- [ ] No obvious request waterfalls (parallelize with `Promise.all` or parallel server components).
- [ ] LCP < 2.5s on a throttled mid-tier mobile profile (Lighthouse).
- [ ] CLS < 0.1.
- [ ] INP < 200ms on primary interactions.
- [ ] Bundle size sanity-checked (`@next/bundle-analyzer` or build output) — no unexpected 500kb+ vendor chunks.

## 5. UI States

For every data-driven screen verify ALL of:

- [ ] **Empty** — clear copy + recovery action.
- [ ] **Loading** — skeleton (preferred) or spinner; never blank screen.
- [ ] **Error** — message + retry; no silent failure.
- [ ] **Success** — primary content.
- [ ] **Partial / paginated** — works as designed for partial data.

## 6. Accessibility (WCAG 2.1 AA)

- [ ] Every interactive element is a real `<button>`, `<a>`, or input — no `<div onClick>`.
- [ ] Form fields have visible labels associated via `htmlFor` / `aria-labelledby`.
- [ ] Icon-only buttons have `aria-label` or visually hidden text.
- [ ] Focus is always visible (`focus-visible` styles).
- [ ] Tab order matches visual order.
- [ ] Escape closes overlays/modals; focus returns to the trigger.
- [ ] Color contrast: text ≥ 4.5:1, UI components ≥ 3:1.
- [ ] No information conveyed by color alone.
- [ ] Live regions used for async updates that aren't focused (toasts, validation errors).
- [ ] Headings are hierarchical (one `h1` per page, no skipped levels).
- [ ] Page has a meaningful `<title>` and `<meta name="description">`.
- [ ] Images have `alt` (empty `alt=""` for decorative).

## 7. Forms

- [ ] Validation runs on blur and on submit, not on every keystroke (unless designed otherwise).
- [ ] Server-side validation mirrors client (same Zod schema).
- [ ] Errors are announced to assistive tech (`aria-describedby` + live region).
- [ ] Submit button shows pending state and is disabled during submission.
- [ ] Successful submission gives clear feedback (toast, redirect, or inline confirmation).

## 8. Errors & Edge Cases

- [ ] Every async path has an error handler — no empty `catch` blocks.
- [ ] Network errors render an actionable UI (retry button).
- [ ] App has at least one `error.tsx` boundary at the route or layout level.
- [ ] `not-found.tsx` exists for invalid routes/resources.
- [ ] No `console.error` / `console.warn` on the happy path.

## 9. Security Basics

- [ ] No secrets in client bundle (grep for env vars, ensure server-only ones don't start with `NEXT_PUBLIC_`).
- [ ] User input is never rendered as raw HTML (no `dangerouslySetInnerHTML` without sanitization).
- [ ] External links use `rel="noopener noreferrer"` when `target="_blank"`.
- [ ] Auth-gated routes are guarded server-side, not only by hiding UI.

## 10. Testing

- [ ] Each P0 user story has at least one Playwright spec.
- [ ] Each critical flow has at least one negative-path test.
- [ ] Tests use accessible queries (`getByRole`, `getByLabel`) — minimal `data-testid` usage.
- [ ] No `page.waitForTimeout` — uses `expect(...).toBeVisible()` style waits.
- [ ] `pnpm test:e2e` is green locally.

## 11. Design Tokens (Tokenized System Integrity)

- [ ] `tokens/primitive.json`, `tokens/semantic.json`, `tokens/component.json` exist; primitives and semantics non-empty.
- [ ] All 8 categories covered: color, spacing, typography, radius, shadow, z-index, motion, breakpoints.
- [ ] Every semantic token references a primitive (or a documented semantic).
- [ ] Semantic tokens named by **role** (`color.text.primary`), never by **appearance** (`color.dark-gray`).
- [ ] Light + dark themes redefine **only the semantic tier**.
- [ ] `globals.css` exposes semantic tokens via `@theme inline`; primitives held as CSS vars.
- [ ] `grep -RnE 'bg-(red|blue|green|gray|slate|zinc|neutral|stone|amber|yellow|lime|emerald|teal|cyan|sky|indigo|violet|purple|fuchsia|pink|rose)-[0-9]+' app components features` returns **zero** hits.
- [ ] `grep -RnE '\[#?[0-9a-fA-F]{3,8}\]|\[[0-9]+px\]' app components features` returns **zero** hits (arbitrary value classes).
- [ ] No inline `style={{ color: ... }}` / `style={{ padding: ... }}` for static values.

## 12. Component Segregation

- [ ] `components/ui/` contains only shadcn primitives — no domain language, no data fetching.
- [ ] `components/shared/` contains only cross-feature composites — knows no specific entities.
- [ ] Every feature lives under `features/<name>/` with the documented sub-structure.
- [ ] Route-local components live under `app/<route>/_components/` (leading underscore).
- [ ] No cross-feature imports (grep `from '@/features/X'` from inside `features/Y/`).
- [ ] `components/ui/*` does not import from `features/*` (primitive depends on domain — forbidden).
- [ ] No flat dumping ground: `components/` has only `ui/` and `shared/` as children.
- [ ] `"use client"` is at the smallest necessary leaf — audit each occurrence and confirm a `useState` / `useEffect` / event handler / browser API justifies it.
- [ ] Each feature exposes a clear public surface (`index.ts` or explicit exports).

## 13. Code Hygiene

- [ ] Comments only explain non-obvious WHY, never WHAT.
- [ ] No dead code, no commented-out blocks.
- [ ] Imports sorted/grouped consistently.
- [ ] Component file names match the default export.
- [ ] No `console.log` left in the codebase.
- [ ] No `TODO` / `FIXME` without an owner and a linked issue.

## 14. Documentation Coherence

- [ ] Every P0 story in `user-stories.md` has matching code and tests.
- [ ] `api-contract.md` matches what the code actually calls.
- [ ] `data-model.md` matches the Zod schemas in `lib/schemas/`.
- [ ] `design.md` screens map to routes in `app/`.
- [ ] Deviations from the architect's plan are documented in `architecture.md`.

---

## How to Use This Checklist

**As reviewer:** create a copy in the review report and mark every item. Anything failed becomes a finding with severity:

- Section 1, 2 (top half), 6 (interactive elements, labels), 8 (silent failures), 9 → **P0** if failed.
- Section 11 (hardcoded values, missing tiers), 12 (cross-feature imports, primitives polluted with domain) → **P0** — system rot starts here.
- Section 4 (perf budgets), 5 (missing states), 6 (polish), 10 (negative paths) → **P1**.
- Section 12 (`"use client"` placement, segregation polish), 11 (component-tier usage discipline) → **P1**.
- Section 13 (hygiene), 14 (doc drift) → **P2** unless drift hides a real bug.

**As engineer (self-check):** run through this before declaring engineer DoD complete. Most "QA found a bug" moments come from sections 5, 6, and 8.
