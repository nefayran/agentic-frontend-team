# Frontend Training RFP Set (0 → E2E)

A curated set of 15 deliberately diverse RFPs for teaching product managers how to build quality frontend products end-to-end. Each RFP highlights a distinct lesson in UI complexity, data patterns, interaction models, or infrastructure requirements.

---

## 1. TaskFlow — Kanban Board for Teams

- **Domain:** Productivity / SaaS
- **Teaches:** drag-and-drop, optimistic updates, real-time sync (WebSocket), complex client state.
- **Core:** boards → columns → cards, filters, assignees, comments, change history.
- **Challenge:** conflict resolution when multiple users edit at once.

---

## 2. MediScan — Patient Portal for a Clinic

- **Domain:** HealthTech
- **Teaches:** handling sensitive data, accessibility (WCAG AA), form validation, file uploads (PDF / DICOM lab results).
- **Core:** booking appointments, visit history, prescriptions, telemedicine video calls.
- **Challenge:** privacy requirements, action audit log.

---

## 3. StreamPulse — Analytics Dashboard for Marketers

- **Domain:** BI / Analytics
- **Teaches:** heavy tables (10k+ rows), virtualization, charting (Recharts / ECharts), filters with URL state, CSV/PDF export.
- **Core:** customizable widgets, drill-down, period comparison.
- **Challenge:** performance on large datasets.

---

## 4. GreenBasket — Grocery Delivery E-commerce

- **Domain:** E-commerce
- **Teaches:** catalog + cart + checkout, payments (Stripe), addresses/cards, promo codes, cart state across devices.
- **Core:** delivery slots, item substitutions, recommendations.
- **Challenge:** SEO (SSR/SSG), Core Web Vitals.

---

## 5. CodeReview Pro — GitHub PR Alternative

- **Domain:** DevTools
- **Teaches:** diff views, syntax highlighting, inline comments, keyboard shortcuts, information-dense UI.
- **Core:** reviews, threads, suggested changes, CI status surfaces.
- **Challenge:** monospace layout, navigating large files.

---

## 6. NestHunt — Apartment Rental Search

- **Domain:** Marketplace
- **Teaches:** map + list sync (Mapbox/Leaflet), complex filters, photo galleries, lazy loading.
- **Core:** favorites, comparison, chat with landlord, virtual tours.
- **Challenge:** mobile UX, geo-search.

---

## 7. FormForge — No-code Form Builder

- **Domain:** Low-code / Builder
- **Teaches:** drag-and-drop builder UI, dynamic JSON schema, preview, undo/redo, embed mode.
- **Core:** conditional field logic, integrations (webhook, Slack).
- **Challenge:** runtime rendering of arbitrary structures.

---

## 8. PennyJar — Personal Finance and Budgets

- **Domain:** FinTech (B2C)
- **Teaches:** transaction categorization, spending charts, savings goals, CSV import from banks.
- **Core:** multi-currency, recurring payments, budget forecast.
- **Challenge:** money math (decimal, rounding, localization).

---

## 9. LearnLoop — LMS with Interactive Lessons

- **Domain:** EdTech
- **Teaches:** video player with progress, quizzes, in-browser code playground, gamification (streaks, badges).
- **Core:** offline mode (PWA), certificates.
- **Challenge:** progress persistence, sandboxed iframes.

---

## 10. OnCallShift — Engineering On-Call Scheduler

- **Domain:** Internal tool / Ops
- **Teaches:** calendar views (week/month), time zones, swap requests, incident escalation.
- **Core:** PagerDuty/Slack integrations, push notifications.
- **Challenge:** correctness of timezone logic.

---

## 11. VoiceNote AI — Meeting Transcriber

- **Domain:** AI product
- **Teaches:** in-browser audio recording, streaming transcription, LLM chat over meeting context, markdown rendering.
- **Core:** tags, full-text transcript search, sharing.
- **Challenge:** streaming responses (SSE), long-form content.

---

## 12. BoardRoom — Virtual Whiteboard (Miro-lite)

- **Domain:** Collaboration
- **Teaches:** Canvas/SVG, zoom/pan, multi-cursor (CRDT/Yjs), custom shapes, infinite canvas.
- **Core:** comments, templates, export.
- **Challenge:** render performance, collaborative editing.

---

## 13. PawPal — Social Network for Pet Owners

- **Domain:** Social
- **Teaches:** infinite feed, likes/comments, 24h stories, notifications, client-side feed ranking.
- **Core:** pet profiles, chats, events (walks).
- **Challenge:** optimistic UI, media uploads.

---

## 14. WarehouseOS — Mobile Interface for Warehouse Workers

- **Domain:** Industrial / B2B
- **Teaches:** mobile-first, barcode scanning (camera/BT), offline-first approach, reconnect sync.
- **Core:** receiving, inventory, transfers.
- **Challenge:** working without network, sync conflicts, large buttons (gloves).

---

## 15. GovPortal — Online Government Service Application

- **Domain:** GovTech / Public
- **Teaches:** multi-step wizards with save-and-resume, digital signatures, document uploads, application status tracking.
- **Core:** localization, accessibility, legal compliance.
- **Challenge:** strict validation, printable PDFs, very broad audience (including seniors).

---

## Coverage Matrix

| Frontend Aspect | RFPs |
|---|---|
| Real-time / collaboration | 1, 11, 12, 13 |
| Heavy data / performance | 3, 5, 12 |
| Maps / geo | 6 |
| Drag & drop | 1, 7, 12 |
| Forms & validation | 2, 7, 15 |
| Payments | 4, 8 |
| Mobile-first / PWA / offline | 9, 14 |
| Media (video / audio / canvas) | 9, 11, 12 |
| Accessibility / compliance | 2, 15 |
| AI / streaming | 11 |
| SEO / SSR | 4 |

---

## Possible Next Steps

- Expand any single RFP into a full document (goals, personas, user stories, non-functional requirements, success metrics).
- Propose a learning order by increasing complexity.
- Generate a "quality frontend" checklist that PMs apply when dissecting each RFP.
