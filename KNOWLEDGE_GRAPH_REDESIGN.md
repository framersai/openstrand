# Knowledge Graph Redesign & Editing Surface

## 1. Objectives
- Replace the demo-only `WeaveGraph` canvas with a production-ready 3D knowledge graph that reflects live strand/weave data.
- Support inline authoring: rename nodes, create/delete edges, attach metadata, and launch the full WYSIWYG composer when deeper editing is required.
- Deliver academically minded navigation, analysis lenses, and clustering to improve exploration of complex research graphs.
- Ship as the default experience (no beta flag) while keeping documentation and localization in sync.

## 2. Current Surface Audit

| Area | Notes |
| ---- | ----- |
| Frontend components | `openstrand-app/src/components/pkms/weave/WeaveGraph.tsx` & `WeaveVisualization3D.tsx` render synthetic data via imperative canvas routines. No real API integration or editing hooks. |
| State | There is no dedicated graph store. Visualization relies on demo `nodes` prop or generated nodes. Strand data and metadata live in separate Zustand slices (`openstrand.store`, `visualization-store`). |
| Services | `openstrand.api.ts` exposes `weaveAPI` (list, subgraph, path, recommendations, metrics, export). Endpoints are read-only; no mutation routes for node/edge updates. |
| Backend | Fastify routes handle read operations only. Need verification of schema/table support for inline edits (weave nodes, edges, metadata). |
| UX | Controls limited to play/pause, reset, fullscreen button placeholders. Navigation uses manual rotation increments, no orbit controls or keyboard navigation. No clustering or filtering. |
| Docs | PKMS tutorials mention the knowledge graph but describe legacy behavior. No mention of editing capabilities or new controls. |

## 3. Data & API Roadmap
1. **Schema Review**
   - Confirm backend models for strands, relationships, weave nodes/edges, metadata tags.
   - Determine required fields for inline editing (title, summary, tags, visibility, topic taxonomy).

2. **Write Endpoints** (Fastify + Prisma)
   - `PATCH /weave/nodes/:id` – rename, update metadata.
   - `POST /weave/nodes` – create new strand/concept placeholders.
   - `DELETE /weave/nodes/:id` – optional; respect referential integrity.
   - `POST /weave/edges` & `DELETE /weave/edges/:id` – manage relationships.
   - `POST /weave/nodes/:id/metadata` – attach tags/notes (optional separate route).
   - `POST /weave/subgraph` – fetch deltas after edits to avoid full reload.

3. **Payload Contracts**
   - Introduce version field or `updated_at` timestamp for last-write-wins semantics.
   - Standardize node/edge shape for frontend store: IDs, type, label, metadata summary, edge weight.
   - Include clustering hints (community ID) and graph metrics if available.

4. **Incremental Loading**
   - Define query params for viewport load: bounding box, depth, node limit.
   - Support pagination for large graphs and background hydration of remote clusters.

## 4. Frontend State Architecture
- Create `useKnowledgeGraphStore` (Zustand) keyed by graph ID / weave type.
  - **State:** nodes, edges, clusters, selection, edit mode, viewport state, pending mutations, sync status, analytics overlays.
  - **Actions:** loadInitialGraph, loadSubgraph, applyMutation (optimistic), revertMutation, setClusteringOptions, setFilter, setSelection, setEditMode.
- Store integrates with `useOpenStrandStore` for strand metadata hydration and to mark local/offline statuses.
- Mutation queue handles optimistic updates with rollback; event emitter for UI components to subscribe to changes.

## 5. Rendering & Interaction Layer

1. **Rendering Stack**
   - Adopt `three.js` with `@react-three/fiber` for GPU-accelerated scene graph.
   - Use `@react-three/drei` for orbit controls, camera helpers, gizmos, and pivot controls.
   - Integrate `d3-force-3d` (or `ngraph.force`) for layout computations running in a web worker to avoid blocking UI.

2. **Navigation & Controls**
   - Orbit controls with constrained zoom limits, keyboard WASD/EQ for pan/zoom.
   - “Center on selection” & “Reset view” actions.
   - Minimap/overview lens showing cluster placement, clickable to jump focus.
   - Academic analytics overlays (timeline axis, discipline color coding, node degree heatmap).

3. **Performance**
   - Use InstancedMesh for nodes and edges where possible.
   - Render clustering hulls/billboards for aggregated groups.
   - Debounce store updates to 60 fps max.

## 6. Editing Experience

1. **Selection Popover**
   - On node click: open anchored panel with quick fields (title, type, tags), metadata badges, and a CTA button “Open full composer.”
   - Inline renaming with validation (slug uniqueness, reserved names).

2. **Edge Creation Workflow**
   - “Add relation” button -> choose relation type -> select target node via search or drag-from-node-handle.
   - Visual highlight of potential target nodes; confirm prompt.

3. **Metadata Attachments**
   - Support taxonomy dropdowns, numeric weights, notes.
   - Save uses optimistic update to store + API; highlight unsynced changes.

4. **Conflict Handling**
   - Last write wins: API returns authoritative payload; store merges result.
   - Display subtle toast when remote updates override local state.

5. **Inline Editor**
   - Compact markdown/RTF fields for short summaries; link to full WYSIWYG composer page for long-form editing.
   - Keep editing context (selected node ID) when navigating to composer.

## 7. Incremental Loading & Clustering
- Auto-fetch nodes on camera move using debounced viewport queries.
- Clustering modes:
  - **Auto** (default): backend-supplied community IDs; render as collapsible groups.
  - **Manual**: user pin/unpin nodes, adjust grouping.
  - Settings panel (in graph toolbar) to toggle clustering, adjust detail slider, and configure maximum visible nodes.
- Preload neighbor shells (N hops) for focused editing without full graph load.

## 8. Access, Permissions, and Collaboration
- Editing allowed for any authenticated user; maintain bottom ribbon showing edit/view status, last synced timestamp, and conflict notifications.
- Offline/local mode: queue edits, mark nodes as “pending sync,” and merge when connection restored.
- Provide admin flag to temporarily lock graph (future enhancement if needed).

## 9. Testing & Quality
- **Unit Tests:** graph store reducers, API hooks, editing popover logic, clustering config.
- **Integration Tests:** load graph, perform rename, add/remove edge, ensure store + backend sync.
- **Visual Regression:** screenshot tests for graph controls, popovers, clustering states (use Chromatic or Playwright).
- **Performance Bench:** synthetic dataset (1k, 5k nodes) to validate frame rate, incremental loading latency.
- **Accessibility:** ensure keyboard navigation, focus management in popovers, adequate contrast for academic palette.

## 10. Documentation & Localization
- Update:
  - `docs/PKMS` (existing tutorials) with new graph editing workflow.
  - Onboarding wizards (`LocalOnboarding`, team onboarding) to mention inline graph editing.
  - Derive new tutorial pages for clustering and analytic overlays (`/tutorials` namespace with i18n strings under `tutorials.graph`).
- Create change log entry and design doc (this file) to track decisions.
- Ensure all new strings live in `i18n/messages.ts` namespaces and provide Spanish/Japanese translations for smoke testing.

## 11. Implementation Phases
1. **Backend Support** – add mutation routes + tests, document API contracts.
2. **State & Services** – implement graph store, API hooks, mutation handlers.
3. **Rendering Core** – build R3F scene, orbit controls, node/edge components, incremental loading.
4. **Editing UX** – popover menus, inline forms, edge creation, conflict handling.
5. **Analytics Enhancements** – clustering UI, overlays, filtering, status ribbon.
6. **Docs & Localization** – update tutorials, onboarding, generated docs; run translations.
7. **QA & Rollout** – performance tuning, accessibility, regression tests, monitoring hooks.

## 12. Dependencies & Tooling
- Add packages: `three`, `@react-three/fiber`, `@react-three/drei`, `d3-force-3d`, `zustand/middleware/immer` (optional for store ergonomics).
- Consider `leva` for developer debug controls during implementation (remove or hide for production).
- Ensure build pipeline handles WebGL/three tree-shaking (configure Next.js transpile as needed).

## 13. Open Considerations
- Evaluate whether to reuse existing command palette for graph actions.
- Decide on theming for academic overlays (contrast, colorblind-friendly palettes).
- Plan migration path for existing users’ saved view preferences (if any).

---

**Next Actions**
1. Draft API design changes (request/response examples) and align with backend team.
2. Spike R3F scene rendering with mock data to validate performance and control schemes.
3. Prepare UX wireframes for popovers, minimap, and analytics toolbar for stakeholder review.
# Knowledge Graph Redesign – Implementation Blueprint

## Overview

We are overhauling the PKMS knowledge graph experience to provide real-time editing, richer analytics, and a research-friendly navigation model. The new stack replaces the demo canvas renderer with a production-ready 3D scene, synchronizes edits with strand/weave data, and introduces inline metadata authoring while preserving a link to the full composer.

This document outlines the required changes across backend APIs, frontend state, rendering, UX, and documentation. It serves as the implementation contract for the feature work tracked in the `knowledge-graph` epic.

## Goals

- Inline management of graph nodes and edges (rename, retag, connect/disconnect, metadata updates)
- Seamless invocation of the existing strand composer for deep content edits
- Academic, analytics-oriented 3D navigation with keyboard, clustering, and focus tools
- Incremental loading and clustering for large graphs, with user-configurable thresholds
- Optimistic editing with conflict-safe overwrites (last write wins + gentle notifications)
- Shipping as the default experience (no long-term feature flag)

## Non-Goals (for this iteration)

- Real-time multi-user presence indicators or CRDT-based collaboration
- Advanced diff/merge workflows beyond last-write-wins with toast notifications
- Mobile-first fine-tuning (desktop experience is primary, responsive baseline still required)

## Architecture Changes

### Backend / API Surface

1. **Graph Mutations**
   - `PATCH /api/v1/weave/nodes/:id` → rename node, update type/tag metadata
   - `POST /api/v1/weave/nodes` → create new node bound to strand/weave
   - `DELETE /api/v1/weave/nodes/:id`
   - `POST /api/v1/weave/edges` → create relationships (source, target, weight, annotation)
   - `DELETE /api/v1/weave/edges/:id`
   - `PATCH /api/v1/weave/edges/:id` → update metadata/weight

2. **Metadata Attachments**
   - Extend strand metadata service to accept partial updates from graph popover
   - Fire change events that update derived caches (search index, analytics summaries)

3. **Graph Fetching**
   - `GET /api/v1/weave/graph` accepts query params:
     - `bounds=<x,y,z,radius>` for incremental loading
     - `depth`, `types[]`, `cluster=true|false` toggles
   - Response includes `clusters` array (precomputed on server when requested)

4. **Clustering & Analytics**
   - Add service module wrapping community detection (`node modularity`, `betweenness`)
   - Persist user toggle preferences in profile or accept per-request settings

### Frontend State

- Create `useKnowledgeGraphStore` (Zustand) with slices:
  - `nodes`, `edges`, `clusters`, `selection`, `settings`, `loadingState`
  - Actions: `loadViewport`, `setSelection`, `applyNodeUpdate`, `applyEdgeUpdate`, `toggleCluster`, `setNavigationMode`
  - Maintain optimistic queue with rollback on failure (store snapshot + error toast)
- Add React Query hooks for graph loading/mutations with cache hydration to the store
- Integrate with existing strand store to reflect metadata changes in lists/panels

### Rendering Stack

- Adopt **Three.js + @react-three-fiber** for the scene graph
- Use `d3-force-3d` for initial layout + cluster separation (run in worker to avoid blocking UI)
- Implement camera controller with:
  - Orbit controls + keyboard pan/zoom (WASD, QE for vertical)
  - Focus-on-selection (auto animate to node centroid)
  - Minimap overlay for orientation
- Render nodes as spheres with shader-based halos; edges as tubes with curvature hints
- Support layering overlays (heat maps, type-specific glyphs, time ribbons)

### Interaction & Editing

- **Selection pipeline**
  - Raycast hits populate selection store (support multi-select via Shift)
  - Popover toolbar anchored to selection with actions: Rename, Retag, Add Connection, Edit Metadata, Open Composer

- **Inline Editor**
  - Lightweight inline form for properties (title, summary, tags)
  - Quick metadata chips toggles (confidence, status)
  - “Open in Composer” button linking to full strand page

- **Edge creation**
  - Drag from node handle or keyboard command → show candidate nodes, confirm connection
  - Provide quick weight presets (weak/strong) and notes

- **Incremental loading**
  - Debounced viewport changes call `loadViewport` → merges new nodes, updates clusters
  - Show skeleton nodes (ghost spheres) while loading

- **Clustering controls**
  - Toolbar with presets: Auto, Manual, Discipline, Chronology
  - Expand/collapse clusters; maintain expanded state in store

### UI Enhancements

- Academic tool palette with filters (node type, activity status, tags)
- Analytics panel toggles: degree distribution, recency heatmap, trust scores
- Status ribbon (bottom): mode indicator (View/Edit), last sync timestamp, conflict alerts
- Improved keyboard shortcuts cheat sheet (displayable with `?`)

## Documentation & Tutorials

- Update existing PKMS docs sections:
  - `docs/pkms/knowledge-graph.md` (create if missing)
  - Tutorials referencing graph navigation and editing
  - Onboarding wizard copy for offline/local mode (call out inline editing availability)
- Add “Knowledge Graph Editing” tutorial to `/tutorials` with i18n hooks and sample translations (EN + ES + JA stub)
- Document new API endpoints in backend README / API reference

## Testing & QA

- Unit tests: store reducers, mutation helpers, clustering utilities
- Integration tests: editing flows (rename, link/unlink, metadata updates), optimistic rollback
- Visual regression snapshots for renderer (Chromatic/Playwright optional)
- Performance tests with synthetic large graphs (1k, 5k nodes) ensuring smooth orbit
- Accessibility: keyboard navigation, ARIA labels for toolbar/popovers

## Milestones

1. **Backend foundations** – routes + services for graph editing, clustering, incremental loading
2. **State + hooks** – frontend store, React Query wiring, optimistic logic
3. **Renderer scaffold** – three.js scene with static data, new navigation controls
4. **Editing UI** – selection popovers, inline metadata form, edge creation
5. **Incremental + clustering** – viewport loading, cluster toggles, settings integration
6. **Analytics overlays & UX polish**
7. **Docs & i18n updates**
8. **QA + rollout**

Each milestone should include documentation updates and corresponding unit/integration tests.

## Open Questions / Follow-ups

- Finalize analytical overlays (which metrics matter most to initial users?)
- Confirm storage location for user graph preferences (local vs profile)
- Decide on fallback behavior for unsupported browsers (WebGL warnings)

---

This document will evolve as implementation proceeds. Update it when endpoint contracts or UX decisions change, and reference it in PR descriptions for traceability.

## 9. Implementation Progress

- **React Three-Fiber Scene**: The `KnowledgeGraphScene` component now renders live workspace data using `@react-three/fiber`, complete with Orbit controls, ambient/directional lighting, and adaptive edge/node styling keyed by type.
- **Store Wiring**: `KnowledgeGraphCanvas` delegates directly to the scene, passing dimension props from `KnowledgeGraphPanel` so the Zustand store remains the single source of truth for selection, clustering, and CRUD flows.
- **Segment Loading Hook**: Initial incremental loading is triggered on mount via `loadGraphSegment({ limit: 600, cluster: true })`, laying groundwork for viewport-aware fetching.
- **Viewport Fetching**: Orbit interaction now streams additional graph segments by sampling the camera target and radius, keeping the 3D view hydrated as users explore.
- **Focus Controls**: Toolbar exposes “Focus selection” backed by store-level focus requests; the scene animates OrbitControls to the computed centroid and distance.
- **Viewport Telemetry**: Store records viewport radius and freshness timestamps so the UI can surface streaming status and scale hints.
- **Inline Composer**: Inspector now launches a Strand composer dialog for strand-backed nodes, keeping the graph focused while authors edit content inline.
- **Inspector Tooling**: Node panel now edits strand links and note types directly, validates strand identifiers inline, and exposes tags/difficulty fields; the inline composer can mint new strands when none are linked, and shared tag suggestions speed up metadata entry.
- **Type Updates**: `WeaveNode` gained an optional `clusterId` for consistent cluster-aware colorization and selection.

