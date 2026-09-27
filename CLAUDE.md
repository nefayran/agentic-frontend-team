# Frontend RFP Training — Orchestration Guide

This project is a **training environment** where product managers learn to build a quality Next.js frontend end-to-end from an RFP. The PM acts as the orchestrator and invokes specialized role-agents one stage at a time.

> **Golden rule for the PM:** never let one role do another role's work. If the engineer starts inventing requirements, stop and go back to the BA. If the designer makes architectural decisions, stop and go back to the architect.

---

## The Pipeline

```
RFP
 │
 ▼
[business-analyst] ── requirements.md, user-stories.md, acceptance-criteria.md
 │
 ▼
[architect]       ── architecture.md, data-model.md, api-contract.md
 │
 ▼
[designer]        ── design.md, wireframes, design-tokens.md, a11y-notes.md
 │
 ▼
[nextjs-engineer] ── working code in /app, components, state, data layer
 │
 ▼
[qa-engineer]     ── e2e/*.spec.ts, test-plan.md, all tests green
 │
 ▼
[code-reviewer]   ── review-report.md, quality checklist signed off
```

All artifacts for an RFP live under `docs/<rfp-slug>/`. Code lives in the standard Next.js tree at the repo root (one repo per RFP).

---

## Role Router — When to Call Whom

| Trigger | Call | Why |
|---|---|---|
| New RFP, nothing scoped yet | `business-analyst` | Turn RFP prose into testable requirements |
| Requirements exist, no tech decisions | `architect` | Pick stack details, draw data model, define API |
| Architecture exists, no UI defined | `designer` | Wireframes, component inventory, a11y plan |
| Design exists, no code | `nextjs-engineer` | Implement against the design + contract |
| Code exists, no tests | `qa-engineer` | E2E test plan + Playwright suite |
| Tests green, before "ship" | `code-reviewer` | Final quality gate against the checklist |
| Mid-stage: requirement looks wrong | back to `business-analyst` | Never patch a bad requirement downstream |
| Mid-stage: design impossible to build | back to `designer` (with architect consult) | Don't let the engineer redesign silently |

**Do not skip stages.** If a PM is tempted to go RFP → engineer directly, that's the anti-pattern this training exists to eliminate.

---

## Definition of Done — Per Stage

Each stage is **only complete** when its artifacts exist AND pass these checks. The next role refuses to start otherwise.

### Business Analyst — DoD
- [ ] `docs/<rfp>/requirements.md` — functional + non-functional, numbered
- [ ] `docs/<rfp>/user-stories.md` — INVEST format, prioritized (P0/P1/P2)
- [ ] `docs/<rfp>/acceptance-criteria.md` — Given/When/Then per story
- [ ] Every story is testable (no "should be fast", instead "LCP < 2.5s on 4G")
- [ ] Out-of-scope list is explicit

### Architect — DoD
- [ ] `docs/<rfp>/architecture.md` — chosen stack, key trade-offs, folder structure
- [ ] `docs/<rfp>/data-model.md` — entities, relations, validation rules
- [ ] `docs/<rfp>/api-contract.md` — endpoints or server actions, request/response shapes
- [ ] Decisions reference specific requirement IDs from BA stage
- [ ] State management strategy declared (server state vs. client state)

### Designer — DoD
- [ ] `docs/<rfp>/design.md` — flows, screen inventory, component list (reuse shadcn where possible)
- [ ] Wireframes for every P0 user story (ASCII / markdown / linked images all OK)
- [ ] `docs/<rfp>/tokens/` — three-tier DTCG token files (`primitive.json`, `semantic.json`, `component.json`, `README.md`) covering all 8 categories (color, spacing, typography, radius, shadow, z-index, motion, breakpoints); light + dark themes defined
- [ ] `docs/<rfp>/a11y-notes.md` — focus order, ARIA roles, keyboard map per screen
- [ ] Empty / loading / error states defined for every data-driven screen
- [ ] Every spatial/color/typography decision in design refers to a token name, not a literal value

### Next.js Engineer — DoD
- [ ] App builds (`pnpm build`) with no TypeScript errors
- [ ] Every P0 user story is reachable and functional in the running app
- [ ] No console errors/warnings on the happy path
- [ ] Loading and error states match what designer specified
- [ ] No `any`, no `@ts-ignore`, no unused exports
- [ ] Code follows `nextjs-conventions`, `design-tokens`, and `component-segregation` skills
- [ ] Tokens compiled into `globals.css`; zero hardcoded color/spacing/radius literals in `components/` and `app/`
- [ ] Component segregation enforced: primitives in `components/ui/`, feature code in `features/<x>/`, route-local in `app/<route>/_components/`

### QA Engineer — DoD
- [ ] `docs/<rfp>/test-plan.md` — what is covered, what is intentionally not
- [ ] One Playwright spec per P0 user story under `e2e/`
- [ ] `pnpm test:e2e` runs locally and passes
- [ ] At least one negative-path test per critical flow (error, validation, empty state)

### Code Reviewer — DoD
- [ ] `docs/<rfp>/review-report.md` — issues found, severity, fixed/deferred
- [ ] `quality-checklist` skill executed end-to-end and signed off
- [ ] No P0 issues remain open

---

## Skills Index — When to Load What

| Skill | Loaded by | Purpose |
|---|---|---|
| `design-tokens` | designer (author), engineer (compile), reviewer (audit) | DTCG three-tier system, OKLCH primitives, Tailwind v4 `@theme inline` wiring |
| `component-segregation` | engineer (build), reviewer (audit) | App Router colocation + feature/shared/ui layers, promotion rule, import boundaries |
| `nextjs-conventions` | engineer (build), reviewer (audit) | Folder structure, RSC, data layer, state, forms, styling |
| `quality-checklist` | reviewer (full audit), engineer (self-check) | Final 14-section quality gate with P0/P1/P2 severity mapping |

## Quality Bar (Applied at Reviewer Stage)

Use the `quality-checklist` skill for the full version. Headlines:

- **Performance:** LCP < 2.5s, INP < 200ms, CLS < 0.1 on a mid-tier mobile profile
- **Accessibility:** WCAG 2.1 AA — keyboard navigable, visible focus, contrast ≥ 4.5:1, semantic HTML
- **Type safety:** TS strict, zero `any`, all API boundaries validated with Zod
- **State discipline:** server state in TanStack Query, client state in Zustand or local; no overlap
- **Errors:** every async path has loading + error UI; no silent failures
- **Tests:** every P0 story covered by at least one Playwright spec

---

## Default Tech Stack (Don't Re-Litigate)

- **Framework:** Next.js 15 (App Router) + TypeScript (strict)
- **Styling:** Tailwind CSS + shadcn/ui
- **Server state:** TanStack Query
- **Client state:** Zustand (only when local state is insufficient)
- **Validation:** Zod (shared between server and client)
- **Forms:** React Hook Form + Zod resolver
- **Unit tests:** Vitest + React Testing Library
- **E2E:** Playwright
- **Package manager:** pnpm

The architect MAY justify deviations in `architecture.md` with a written trade-off — never silently.

---

## Project Structure (Per RFP)

```
<rfp-slug>/
├── docs/<rfp-slug>/        # artifacts from BA, architect, designer, QA, reviewer
├── app/                    # Next.js App Router
├── components/
│   ├── ui/                 # shadcn primitives
│   └── <feature>/          # feature components
├── lib/
│   ├── api/                # client/server data layer
│   ├── schemas/            # Zod schemas
│   └── utils/
├── stores/                 # Zustand stores
├── e2e/                    # Playwright specs
├── CLAUDE.md               # this file (training guide)
└── package.json
```

---

## How a PM Uses This

1. Pick an RFP from `rfp-set.md`.
2. Create `docs/<rfp-slug>/` and drop the RFP excerpt in `rfp.md`.
3. Invoke the **business-analyst** agent. Review its output critically — push back on vague stories.
4. Only when BA DoD is fully green, invoke the **architect**. Repeat.
5. Continue down the pipeline. Never skip a role.
6. After the **code-reviewer** signs off, the RFP is "shipped" for training purposes.

**The point of the exercise is the discipline of the handoff**, not the final product. A PM who skips stages and gets a working app has learned nothing.
