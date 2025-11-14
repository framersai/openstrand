# OpenStrand Platform Specification

**Last updated:** November 2025  
**Status:** Recursive strand groundwork shipped; approvals/placeholder UX in active rollout.

This specification describes the production architecture, recursive strand data model, approval workflows, and user experience surfaces that power OpenStrand. It replaces the legacy FastAPI notes and reflects the TypeScript monorepo (`openstrand-app`, `@framers/openstrand-teams-backend`, `@framers/openstrand-sdk`).

---

## 1. High-Level Architecture

```mermaid
graph LR
    A[Next.js App<br/>openstrand-app] -- REST / WS --> B[@framers/openstrand-teams-backend<br/>Fastify + Prisma]
    B -- Prisma --> C[(PostgreSQL / PGlite)]
    B -- BullMQ --> D[(Redis)]
    B -- SDK Types --> E[@framers/openstrand-sdk]
```

- **Frontend (`openstrand-app`)**  
  Next.js 14 App Router, React 18, Zustand stores, Tailwind + Radix UI. Server actions consume the SDK and respect feature flags exposed through `/meta/capabilities`.
- **Backend (`@framers/openstrand-teams-backend`)**  
  Fastify 4 with typed routes. Prisma 5 schema models strands, links, hierarchies, approvals, visibility caches. BullMQ orchestrates cascades and background jobs.
- **Shared SDK (`@framers/openstrand-sdk`)**  
  Type-safe fetch client, retryable requests, SSE helpers, and the canonical TypeScript model definitions.
- **Storage**  
  PostgreSQL 14+ (Supabase/RDS) for production; PGlite for local/offline. Redis optional for queues/visibility invalidation.

### Environment configuration

- A single `.env` file at the repository root feeds both the Next.js app and the Fastify backend (`env-loader` falls back to package-level `.env` only when present). There is no longer a Python service or per-package `.env` to maintain.
- `.env.example` documents provider keys, billing toggles, Swagger exposure, and queue controls. Recursive strand tables, visibility caches, and placeholder defaults are provisioned via Prisma migrations rather than environment overrides—see `docs/STRAND_HIERARCHY_PLAN.md` for operational details.
- Deployment scripts should copy `.env.example` to `.env`, adjust secrets, and commit environment-specific overrides outside of version control.

---

## 2. Recursive Strand Data Model

| Model | Purpose | Key Columns |
|-------|---------|-------------|
| `Strand` | Core knowledge object (note, dataset, visualization, etc.) | `strandType`, `classification`, `noteType`, `placeholderBehavior`, `analysisSnapshot`, `ownerId`, `createdBy`, `updatedBy` |
| `StrandLink` | Unified conceptual/structural/placeholder edges | `sourceId`, `targetId`, `linkType`, `scopeId`, `provenance`, `weight`, `metadata`, `justification`, `createdBy` |
| `StrandHierarchy` | Enforces per-scope DAGs + single primary parent | `strandId`, `scopeId`, `parentId`, `depth`, `position`, `isPrimary`, `path`, `provenance` |
| `StrandVisibilityCache` | Snapshot of inherited visibility & placeholder status | `strandId`, `scopeId`, `audience`, `isVisible`, `isPlaceholder`, `inheritedFrom`, `metadata` |
| `StrandStructureRequest` | Approval workflow for structure edits | `scopeId`, `strandId`, `parentId`, `type`, `status`, `requestedBy`, `reviewedBy`, `payload`, `justification`, `resolutionNote` |

**Link semantics**

- `STRUCTURAL`: parent/child relationships managed per scope.
- `CONCEPTUAL`: knowledge graph references with optional weights/metadata.
- `PLACEHOLDER`: virtual nodes shown to unauthorized viewers; resolved during approvals.
- `REFERENCE` / `DERIVED`: lightweight edges for provenance and reuse.

**Visibility**

- Root strands declare visibility (`private`, `team`, `unlisted`, `public`) and placeholder behaviour (`inherit`, `placeholder-only`, `hidden`).
- Cascades recompute on structure changes, permission grants, or approvals. Results cached in `StrandVisibilityCache` and mirrored to clients via `/strands/:id`.
- Placeholder preferences may be overridden per scope via `/meta/placeholders`.

---

## 3. API Surface

All routes return the standard envelope `{ success, data, meta }`. Highlights:

| Route | Description |
|-------|-------------|
| `GET /api/v1/strands` | Paginated listing with filters (`type`, `noteType`, `scopeId`, `visibility`, `teamId`, `search`). |
| `POST /api/v1/strands` | Create strand + optional structural placement (`scopeId`, `parentId`, `position`) and conceptual relationships. |
| `PUT /api/v1/strands/:id` | Update metadata; cascades hierarchy when parents change. |
| `DELETE /api/v1/strands/:id` | Soft delete (future); currently immediate delete for OSS. |
| `POST /api/v1/strands/:id/relationships` | Upsert conceptual link with justification + scope metadata. |
| `DELETE /api/v1/strands/:id/relationships` | Remove conceptual link (requires write or link permission). |
| `POST /api/v1/strands/:id/structure/requests` | Submit structure change (add child, reparent, reorder, replace placeholder, remove link). |
| `GET /api/v1/strands/:id/structure/requests` | List pending/resolved requests for a strand and scope. |
| `POST /api/v1/strands/structure/requests/:requestId/{approve|reject|cancel}` | Resolve approvals with optional resolution note. |
| `GET /api/v1/strands/:id/graph` | Lightweight conceptual graph for exploratory UI (limited depth, respects visibility). |
| `GET /api/v1/meta/capabilities` | Feature flag manifest (recursive strands, approvals, placeholders, AI tiers). |
| `PUT /api/v1/meta/placeholders` | Manage placeholder icon/text defaults (team scoped). |

All endpoints enforce RBAC via `StrandPermission` (user/team principals) with capability arrays (`read`, `write`, `link`, `structure`, `manage`).

---

## 4. Background Jobs & Cascades

| Queue | Job | Trigger | Outcome |
|-------|-----|---------|---------|
| `structure-cascade` | Propagate visibility + placeholder resolution down the DAG | Structure approvals, hierarchy edits, permission changes | Recomputes `StrandVisibilityCache`, raises webhook events |
| `scope-integrity` | Validate DAG invariants per scope | Hourly or manual | Detects cycles, orphaned placeholders, conflicting parents |
| `link-reindex` | Refresh search/analytics caches after conceptual link edits | Link create/update/delete | Rebuilds search vectors + analytics aggregates |

Jobs emit structured logs and metrics (queue depth, success/failure counts) for dashboards. Hooks planned for Datadog/New Relic.

---

## 5. Frontend Experience

### Strand Workspace

- **Structure Explorer**: tree view showing structural parents/children per scope, with placeholder badges and visibility tooltips.
- **Conceptual Links Panel**: showcases weighted references, provenance, and `add/remove` actions gated by link permissions.
- **Approval Drawer**: reviewers see pending requests, diff preview (parent chain + placeholder summary), and one-click approve/reject with notes.
- **Placeholder Preview**: unauthorized viewers see sanitized placeholders (custom text/icon) while approvals are pending.

### Dashboard

- Updated dataset inspector & strand overview to surface hierarchy placement, visibility cascades, and outstanding approvals.
- Status tiles highlight cascading jobs, queue depth, and recent approvals for the current team scope.

### Account & Billing

- Unified auth controls: landing header exposes sign-in/sign-up for cloud deployments; authenticated users see a profile/billing switcher everywhere.
- Profile page subscription card surfaces current plan, renewal date, trial state, portal actions, and cancellation scheduling via the new `useSubscription` hook.
- Billing workspace reuses the typed `PlanGrid` component with plan-aware CTAs; paid tiers bounce to Stripe/LemonSqueezy checkout while current plans route to the customer portal.
- SDK + `api.ts` expose strongly typed helpers (`getSubscription`, `changeSubscriptionPlan`, `getBillingPortal`) so frontend, CLI, and automation share one contract.
- Operators enable billing by setting the `BILLING_PROVIDER` block in `.env` (`STRIPE_*` or `LEMONSQUEEZY_*` keys plus `FRONTEND_URL` for redirect callbacks). Leave it unset to ship the free Community build.

### Landing Page & Marketing Copy

- Hero and features highlight recursive hierarchies, placeholder-driven sharing, approval workflows, and DAG-safe reuse.
- Prompt chaining references now call out structure-aware contexts instead of the legacy notebook narrative.
- The landing navigation consolidates documentation into a single dropdown with links to tutorials, the live OpenAPI reference (`/docs` when `ENABLE_SWAGGER=true`), and SDK typings so operators can jump straight to implementation details.

---

## 6. Documentation & Tutorials

- `docs/Collaborative-Zettelkasten.md` walks through recursive hierarchy modelling, placeholder etiquette, and approval best practices.
- `docs/INTELLIGENT_DATA_PIPELINE.md` illustrates how schema intelligence feeds structure requests and placeholder defaults.
- Tutorials include pen-and-paper modelling instructions, manual migration checklists, and sample approval flows.
- Generated docs (`docs/generated/*`) updated during CI to reflect new Prisma schema comments and SDK signatures.

---

## 7. Rollout & Observability

- Feature flags: `recursiveStrands`, `structureApprovals`, `placeholderVisibility`.
- Metrics: `structure_cascade_duration_ms`, `placeholder_render_count`, `structure_request_backlog`.
- Alerts: queue depth, cascade failure rate, approval SLA breaches.
- Migration strategy: SQL migration adds new tables, TypeScript backfill migrates relationships + hierarchies, dry-run + conflict reports (`migration_reports/structure_conflicts.json`).

---

## 8. Future Enhancements

- Real-time collaboration on structure requests (WebSocket subscriptions).
- Scoped placeholder themes + overrides per audience.
- Automatic suggestion of structure changes driven by schema intelligence and usage analytics.
- GraphQL surface for partner integrations (read-only to start).
- Branching DAG views for scenario planning / knowledge modelling.

Keep this document current whenever schema, APIs, or UX flows evolve. It is the single source of truth for engineers, designers, and documentation contributors.
