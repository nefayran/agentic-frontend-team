# agentic-frontend-team

Six Claude Code agents that take a product brief (an RFP) to a tested Next.js frontend. Each agent owns one stage
and hands a written artifact to the next one, so requirements, architecture, design, code, tests and review stay
separate.

It began as a training environment for product managers. The PM runs the pipeline and keeps one rule: no agent
does another agent's job. If the engineer starts inventing requirements, the work goes back to the analyst.

## Pipeline

| Stage | Agent | Hands over |
|---|---|---|
| 1 | business-analyst | requirements.md, user-stories.md, acceptance-criteria.md |
| 2 | architect | architecture.md, data-model.md, api-contract.md |
| 3 | designer | design.md, wireframes, design-tokens.md, a11y-notes.md |
| 4 | nextjs-engineer | the app: routes, components, state, data layer |
| 5 | qa-engineer | test-plan.md and a Playwright suite |
| 6 | code-reviewer | review-report.md against the quality checklist |

Artifacts for each RFP go to `docs/<rfp-slug>/`. Every stage has a definition of done in `CLAUDE.md`, and the next
agent refuses to start until it is met.

## What's in the box

- `CLAUDE.md`: the orchestration guide. The pipeline, a router for who to call when something breaks, per-stage
  definitions of done, the default stack.
- `.claude/agents/`: the six agents.
- `.claude/skills/`: `nextjs-conventions`, `component-segregation`, `design-tokens` (W3C DTCG, three tiers) and
  `quality-checklist` (14 sections, P0 to P2).
- `rfp-set.md`: 15 practice RFPs. Each one is built around a different hard part of frontend work.

## Using it

1. Click "Use this template", or copy `CLAUDE.md` and `.claude/` into an empty repository.
2. Open the folder in Claude Code.
3. Pick an RFP from `rfp-set.md`, put it in `docs/<rfp-slug>/rfp.md`, and ask the business-analyst agent to start.
4. Go stage by stage. Read each artifact before you call the next agent, and push back on vague stories early.

The default stack is Next.js 15 (App Router, strict TypeScript), Tailwind CSS with shadcn/ui, TanStack Query,
Zustand, Zod, React Hook Form, Vitest and Playwright, with pnpm. The architect can change it, but only with a
written trade-off in `architecture.md`.

## The practice RFPs

| # | RFP | What it teaches |
|---|---|---|
| 1 | TaskFlow, a Kanban board | drag and drop, optimistic updates, real-time sync |
| 2 | MediScan, a clinic patient portal | sensitive data, WCAG AA, file uploads |
| 3 | StreamPulse, a marketing dashboard | 10k-row tables, virtualization, charts, URL state |
| 4 | GreenBasket, grocery delivery | cart and checkout, Stripe, SEO and Core Web Vitals |
| 5 | CodeReview Pro | diff views, inline comments, keyboard shortcuts |
| 6 | NestHunt, apartment search | map and list in sync, complex filters, galleries |
| 7 | FormForge, a no-code form builder | dynamic JSON schema, preview, undo and redo |
| 8 | PennyJar, personal finance | categorization, spending charts, CSV import |
| 9 | LearnLoop, an LMS | video progress, quizzes, a code playground |
| 10 | OnCallShift, on-call scheduling | calendars, time zones, swap requests |
| 11 | VoiceNote AI, a meeting transcriber | audio recording, streaming transcription, LLM chat |
| 12 | BoardRoom, a whiteboard | canvas, zoom and pan, multi-cursor with CRDTs |
| 13 | PawPal, a pet social network | infinite feed, stories, notifications |
| 14 | WarehouseOS, a warehouse app | mobile first, barcode scanning, offline sync |
| 15 | GovPortal, a government service | multi-step wizards, save and resume, signatures |

## License

MIT
