# Analytics & Visualisation Layer

**Status:** Phase 1 (Strand → Loom → Weave dashboards) implemented November 2025  
**Components:** Fastify API (`/api/v1/analytics/...`), Prisma stats tables, Redis cache, Next.js PKMS dashboard panels

---

## Goals
- Surface descriptive + behavioural metrics at every hierarchy level (Strand, Loom, Weave) without leaving the PKMS dashboard.
- Keep Community Edition lightweight (single weave, implicit global loom) while unlocking multi-loom dashboards for Teams cloud deployments.
- Reuse existing dependencies (`recharts`, `d3`, `chart.js`, `three.js`) and deterministic NLP services, with lazy loading + caching to preserve TTI.

---

## Backend
- **Service:** `AnalyticsService` (Fastify) combines deterministic NLP (`DataIntelligenceService`), readability, POS tagging, sentiment, ratings, embedding coverage, and cost telemetry.
- **Routes:**  
  - `GET /api/v1/analytics/strands/:id?fresh=true`  
  - `GET /api/v1/analytics/looms/:scopeId`  
  - `GET /api/v1/analytics/weaves/:workspaceKey` (`community` or `team:ID`)
- **Persistence:**  
  - `StrandStats` – chart-ready JSON per strand.  
  - `LoomStats` – KPI + topic distribution + timelines per `StrandScope`.  
  - `WeaveStats` – roll-ups per workspace (totals, cost, usage, embedding coverage).  
  - Redis keys (`analytics:strand|loom|weave`) provide 5/10/15 minute TTL caches with `stale-while-revalidate` semantics.
- **Nightly job (optional):** Toggle `ANALYTICS_SCHEDULED_REFRESH=true` to queue BullMQ refreshes for all strands/looms/weaves.

---

## Frontend
- **Hooks:** `useStrandAnalytics`, `useLoomAnalytics`, `useWeaveAnalytics` handle fetch/cancel/refresh logic with error states.
- **Panels (PKMS dashboard):**
  - `StrandAnalyticsPanel` – entity histogram, POS donut, keyword cloud (d3 scale), readability gauges, sentiment + rating badges, embedding stats.
  - `LoomAnalyticsPanel` – total strands/tokens, vocabulary growth area chart, topic distribution pie, embedding coverage, rating aggregates.
  - `WeaveAnalyticsPanel` – workspace totals, provider spend bar chart, hourly usage, deterministic similarity scatter (hash-based preview until streaming embeddings ship), embedding coverage + rating badges.
- **Lazy loading:** `<LazyChart>` wraps each visualization with `IntersectionObserver` + skeleton fallback so heavy chart bundles load only when scrolled into view.
- **Edition behaviour:**  
  - Community: fixed `workspaceKey=community`, Loom switcher shows upgrade CTA.  
  - Teams: additional Weaves + Looms unlocked; workspace select ties into `team:${teamId}` keys.

---

## Roadmap
1. **Streaming similarity graph:** SSE endpoints feed sigma.js / deck.gl / react-three-fiber explorers progressively.
2. **Worker offloading:** UMAP, LDA, clustering run inside Web Workers via `Comlink`, streaming interim coordinates for responsive charts.
3. **Historical analytics:** `analytics_snapshots` table for executive dashboards, budgeting, and auditing.
4. **3D Explorer:** React Three Fiber “solar system” view backed by the same `/analytics/weaves` payloads with instanced rendering + AI Artisan overlays.

---

For full architectural context, see `docs/openstrand/ARCHITECTURE.md` (“Analytics & Visualization Layer” section). PKMS UX details live in `src/components/pkms/analytics/*`. Tests + TSDoc are colocated with the service and hooks.*** End Patch

