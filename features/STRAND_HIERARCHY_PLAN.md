# Recursive Strand Initiative – Execution Plan

## Context Recap

## Status Update (Current Session)
- Frontend surfaces now expose structure request queues, inline approvals, and placeholder editing via the dataset inspector.
- Fastify strand routes now expose create/list/resolve structure request endpoints (`POST /strands/:id/structure/requests`, `GET /strands/:id/structure/requests`, and `POST /strands/structure/requests/:requestId/:action`).
- Landing experience, hero copy, and feature highlights emphasise recursive strands, approvals, and cascaded visibility.
- SDK/API client updated to return single structure requests alongside arrays, avoiding null payload edge cases.
- Interactive examples demonstrate queuing and approving structure requests plus placeholder preferences.

- **Repository topology:** Next.js app (`openstrand-app`), Fastify + Prisma backend (`packages/openstrand-teams-backend`), shared SDK (`packages/openstrand-sdk`), extensive docs (`docs/`), plus ancillary scripts and admin app.
- **Current data model gaps:** `ContentCatalog` still embeds legacy JSON relationships while `WeaveEdge` only covers conceptual links. Services expect normalized tables that are now missing, leading to drift between Prisma models, TypeScript services, and frontend consumers.
- **Frontend assumptions:** Zustand stores and API layer expect REST endpoints such as `/weave`, `/weave/subgraph`, `/strands/upload`, and partial strand payloads that no longer align with Fastify routes. Many components rely on relationship graphs and placeholder behavior that we must rebuild.
- **SDK state:** Types omit `noteType`, hierarchy metadata, visibility caches, and approval flows. Client methods only cover CRUD basics, blocking external integrations.

## Phase 0 – Planning & Safety Nets
1. Capture schema drift and route parity issues (tracked in this doc).
2. Establish feature flags for recursive hierarchy rollout: `recursiveStrands`, `structureApprovals`, `placeholderVisibility`.
3. Define migration guardrails: run `prisma migrate diff` to verify generated SQL, add dry-run scripts, and snapshot counts before/after.
4. Align naming conventions:
   - **Tables:** `Strand`, `StrandLink`, `StrandHierarchy`, `StrandScope`, `StrandVisibilityCache`, `StrandStructureRequest`.
   - **Enums:** `StrandLinkType` (structural vs conceptual), `HierarchyScopeType`, `StructureRequestStatus`.
   - **Queue names:** `structure-cascade`, `visibility-cache`, `scope-dedupe`.

## Phase 1 – Prisma Schema Evolution
1. **Strand core remodel**
   - Replace `ContentCatalog` with new `Strand` model (explicit columns for structural metadata, note info, owners, and JSON payloads for editor content).
   - Introduce JSON column `analysisSnapshot` for future pipeline compatibility and maintain `representedVariants` for visualization reuse.
2. **Unified link model (`StrandLink`)**
   - Fields: `id`, `sourceId`, `targetId`, `linkType`, `scopeId`, `weight`, `provenance`, `justification`, `createdBy`, `createdAt`, `updatedAt`.
   - Constraint: unique `(sourceId, targetId, linkType, scopeId)`; allow self-links only for structural placeholders (`linkType='placeholder'`).
3. **Per-scope DAG (`StrandHierarchy`)**
   - Fields: `strandId`, `scopeId`, `parentId`, `depth`, `position`, `path`, `isPrimary`.
   - Constraint: `parentId` nullable (roots), `isPrimary` ensures single parent per scope.
4. **Scopes & visibility**
   - `StrandScope`: descriptive metadata for collections, datasets, or workspaces.
   - `StrandVisibilityCache`: flattened view capturing resolved visibility (true/false), placeholder flag, inherited source; keyed by `(strandId, scopeId, audience)`.
5. **Approval workflow**
   - `StrandStructureRequest`: records requested structural changes (add child, reparent, merge, unlink) with `requestedBy`, `reviewedBy`, `status`, `payload`.
6. **Legacy auditing tables**
   - `StrandLinkAudit`, `StrandVisibilityAudit` for change logs (optional but recommended).
7. Annotate every model and field with Prisma comments describing purpose and migration guidance.
8. Generate new enums (`StrandClassification`, `PlaceholderBehavior`) and update `schema.prisma` index strategy (composite indexes for frequent queries).

## Phase 2 – Migrations & Backfill
1. Snapshot legacy data:
   - Dump `content_catalog` and `weave_edges`.
   - Extract `relationships` JSON to `legacy_relationships.json`.
2. Create migration scripts:
   - Use `prisma migrate dev` with SQL triggers disabled in dev.
   - Build TypeScript backfill script (`scripts/backfill-strand-links.ts`) to:
     - Map JSON relationships to `StrandLink`.
     - Convert `WeaveEdge` into conceptual links.
     - For each strand, pick primary parent per scope (with heuristics) and populate `StrandHierarchy`.
   - Detect cycles or conflicting parents; log to `migration_reports/structure_conflicts.json`.
3. Populate `StrandVisibilityCache`:
   - For each strand, walk ancestors to compute visibility and placeholders.
   - Flag inconsistent states and emit audit entries.
4. Verify counts before/after:
   - Ensure link totals match.
   - Confirm every strand has at most one primary parent per scope.
5. Make migration idempotent; add `--dry-run` flag for local verification.

## Phase 3 – Background Processing
1. Extend `JobQueueService` with typed helpers (`enqueueStructureCascade`, `enqueueVisibilityRefresh`).
2. Implement workers under `src/services/jobs/structure`:
   - **CascadeWorker:** recomputes descendant visibility, placeholder hydration, and caches.
   - **ScopeIntegrityWorker:** ensures DAG invariants, run hourly.
   - **LinkReindexWorker:** rebuilds search indices / analytics caches.
3. Add metrics via lightweight instrumentation (queue depth, job success/failure, cascade duration) and log structured context for auditability.
4. Provide in-memory fallbacks (respecting current `JobQueueService` design).

## Phase 4 – Backend Routes & Services
1. Refactor `strand.service.ts` to align with new schema:
   - Replace direct `contentCatalog` references with `strand` table.
   - Add methods: `getHierarchy`, `createLink`, `requestStructureChange`, `applyPlaceholderConfig`.
   - Ensure `mapToStrand` aggregates hierarchy, permissions, visibility state.
2. Create `structure.routes.ts` under `/api/v1/structure`:
   - `GET /scopes/:scopeId/tree` – paginated tree fetch.
   - `POST /scopes/:scopeId/requests` – submit structure change requests.
   - `POST /requests/:id/approve|reject` – approve flow.
   - `POST /placeholders/config` – configure placeholder text/icons.
3. Enhance existing `/strands` endpoints:
   - Accept `parentId`, `scopeId`, `position`, `placeholderOptions` on create.
   - Return `links`, `hierarchy`, `visibilityState`.
   - Add `PATCH /:id/visibility` for manual overrides (under feature flag).
4. Update `weave.service.ts` to rely on `StrandLink` for conceptual links.
5. Extend middleware for authorization:
   - Validate structure permissions (link, reparent) before operations.
   - Ensure cascades triggered only when necessary.
6. Add Fastify schemas & `@fastify/swagger` docs for all new routes, with examples.

## Phase 5 – SDK Enhancements
1. Regenerate and expand types:
   - Add `StrandHierarchyNode`, `StrandLink`, `StructureRequest`, `PlaceholderConfig`.
   - Update `Strand` interface to include `hierarchy`, `links`, `visibility`.
2. Client additions:
   - `sdk.strands.links.create/update/delete`.
   - `sdk.structures.tree(scopeId, options)`, `sdk.structures.requestChange`, `sdk.structures.approve`.
   - `sdk.visibility.refresh(strandId, scopeId)`.
3. Ensure exhaustive TSDoc for each method.
4. Add typed error classes for approval rejections and visibility conflicts.
5. Publish updated README examples and migration notes.

## Phase 6 – Frontend Updates
1. API layer (`openstrand.api.ts`):
   - Switch base paths to Fastify equivalents (`/api/v1/strands`, `/api/v1/structure`).
   - Replace `PATCH` calls with `PUT` or new endpoints.
   - Add methods for hierarchy, approvals, placeholders, visibility refresh.
2. Zustand store (`openstrand.store.ts`):
   - Track `hierarchy`, `structureRequests`, `placeholderConfig`, `visibilityState`.
   - Provide actions for tree loading, approval submission, placeholder updates.
3. UI components:
   - Build recursive tree view for strand hierarchy with per-scope toggles.
   - Extend `DatasetInspectorPanel`, `WeaveViewer`, composer flows to display placeholders and approval status.
   - Add access-control modals for approvals and placeholder customization.
4. Accessibility & theming:
   - Ensure tree controls are keyboard navigable and use design-system tokens.
   - Implement placeholder styling guidelines from `DESIGN_SYSTEM.md`.
5. Remove references to defunct endpoints (`/weave/subgraph`) or reimplement using new structure fetch.
6. Add loading/error states for cascades and approvals.

## Phase 7 – Documentation & Tutorials
1. Update `ARCHITECTURE.md`, `spec.md`, `INTELLIGENT_DATA_PIPELINE.md`, `Collaborative-Zettelkasten.md`, `DESIGN_SYSTEM.md`, `ROADMAP.md`, `OPEN_SOURCE_CHECKLIST.md`, and generated docs to reflect the new hierarchy model.
2. Create detailed authoring walkthrough:
   - Manual modeling (pen & paper) with placeholder rules.
   - Digital workflow (composer -> approval -> cascade).
3. Add API reference sections for new endpoints and SDK methods.
4. Provide migration guide for existing deployments (step-by-step with rollback checkpoints).

## Phase 8 – Testing & Verification
1. Backend:
   - Unit tests for services (links, hierarchy, visibility).
   - Integration tests for approval flow and cascades.
2. Frontend:
   - Component tests for tree UI, inspector, approvals.
   - E2E tests covering authoring, approval, and placeholder rendering.
3. SDK:
   - Contract tests to ensure responses align with backend schemas.
4. Migration verification script to compare pre/post invariants (counts, parent uniqueness).

## Phase 9 – Deployment & Monitoring
1. Roll out behind feature flags; enable per tenant/team.
2. Instrument metrics (queue depth, visibility cascade duration, approval turnaround).
3. Set up alerts for DAG violations or cascade job failures.
4. Document escalation paths and rollback plan (disable feature flag, revert migration).

## Deliverables Checklist
- [ ] Prisma schema & migrations aligned with services.
- [ ] Backfill scripts + audit reports.
- [ ] Queue workers & metrics dashboards.
- [ ] Updated Fastify routes, middleware, TSDoc, and tests.
- [ ] Expanded SDK (types, client, docs) with full coverage.
- [ ] Frontend hierarchy UI, approvals, placeholder configuration.
- [ ] Comprehensive documentation updates and tutorials.
- [ ] Automated testing coverage and CI pass.
- [ ] Rollout playbook with monitoring hooks.

