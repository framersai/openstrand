# OpenStrand Architecture

**Last Updated:** November 2025  
**Stack:** TypeScript end-to-end (Node.js 18+, Next.js 14, Fastify 4, Prisma 5)

---

## 1. High-Level View

```
browser / electron shell
        │
        ▼
openstrand-app (Next.js, React 18)
        │          ▲
REST / WebSockets  │ SWR (React Query)
        ▼          │
@framers/openstrand-teams-backend (Fastify + Prisma)
        │
        ▼
PostgreSQL ─── or ─── PGlite (embedded)  ← fallback for local/offline
```

Shared types, validation schemas, and utility functions live inside `packages/openstrand-sdk` to keep the API surface consistent across the monorepo.

### Collaborative Slip-Box & Recursive Strands

- `Strand`, `StrandLink`, `StrandHierarchy`, and `StrandVisibilityCache` replace the legacy `ContentCatalog`/`WeaveEdge` layout.
- Structural and conceptual links are unified through `StrandLinkType` (`STRUCTURAL`, `CONCEPTUAL`, `PLACEHOLDER`, `REFERENCE`, `DERIVED`).
- Per-scope DAG enforcement lives in `StrandHierarchy`; every scope guarantees a single primary parent while allowing cross-scope reuse.
- Authorship metadata is captured on strands (`createdBy`, `updatedBy`, `coAuthorIds`) and on links (`createdBy`, `provenance`, `justification`).
- Visibility cascades are cached in `StrandVisibilityCache` to keep descendant permissions responsive and auditable.
- Structure changes flow through `StrandStructureRequest` with approval queues, Slack/email hooks (roadmap), and background cascades.
- See [`docs/Collaborative-Zettelkasten.md`](./Collaborative-Zettelkasten.md) for the updated PKMS workflows.

---

## 2. Monorepo Layout

| Directory                         | Purpose                                                                |
|----------------------------------|------------------------------------------------------------------------|
| `openstrand-app/`                | Next.js App Router application (public UX, offline-first)              |
| `openstrand-admin/`              | Optional admin interface (kept private; not part of OSS surface)       |
| `packages/openstrand-teams-backend/` | Fastify API, background jobs, billing integrations, Prisma schema   |
| `packages/openstrand-sdk/`       | TypeScript client/SDK (shared models, validation, API wrappers)        |
| `scripts/`                       | Release scripts, database helpers, automation                          |
| `docs/`                          | Product & engineering documentation                                    |

All workspaces are managed with npm workspaces and a single `package-lock.json`.

---

## 3. Application Layers

### Frontend – `openstrand-app`
- Next.js 14 App Router with Server Components where appropriate.
- Tailwind + Radix UI for theming and accessible primitives.
- Zustand for UI state, React Query for API caching, and localStorage/IndexedDB for offline drafts.
- Dynamic routing covers: `/strands`, `/threads`, `/pkms`, `/settings`.
- Feature gating reads `OPENSTRAND_ENVIRONMENT` to hide cloud-only flows on local installs.

### Backend – `@framers/openstrand-teams-backend`
- Fastify 4 with typed routes (Zod schemas).
- Prisma ORM to target both PostgreSQL (cloud) and PGlite (embedded).
- Modular services: billing, search, graph analytics, AI integrations, ingestion pipelines.
- Background jobs via BullMQ + Redis (optional; falls back to in-process mode during local dev).
- Authentication modes: local credentials, JWT session tokens, optional Supabase integration.
- Rate limiting and structured logging baked in (pino + pino-pretty for dev).
- Collaborative notes: recursive strand hierarchies, placeholder policies, link attribution, and approval flows exposed via `/strands`, `/strands/:id/relationships`, and `/strands/:id/structure/requests`.

### Shared SDK – `@framers/openstrand-sdk`
- Generated types from Prisma schema and OpenAPI routes.
- Node + Browser friendly bundle (`CJS`, `ESM`, and `.d.ts` outputs).
- Fetch wrapper with retry/backoff, SSE helpers for long-running tasks.

---

## 4. Data & Storage

| Use Case                     | Default (Local)              | Cloud / Team deployment                   |
|------------------------------|------------------------------|-------------------------------------------|
| Primary database             | PGlite (embedded PostgreSQL) | PostgreSQL 14+ (Supabase, RDS, Timescale) |
| File storage                 | Local filesystem             | S3-compatible bucket (optional)           |
| Background jobs              | In-memory queue              | Redis + BullMQ                            |
| Search / vector store        | Prisma + JSON fields         | (Roadmap) External vector DB integrations |
| Collaborative metadata       | Prisma columns (`noteType`, `createdBy`) | Same schema; use migrations |

Prisma schema is the single source of truth (`packages/openstrand-teams-backend/prisma/schema.prisma`). Migration commands (`npm run prisma:migrate`) generate SQL appropriate for the target provider.

---

## 5. Runtime Detection

`start-local.sh` exports:

```
OPENSTRAND_ENVIRONMENT=local
OPENSTRAND_IS_CLOUD=false
```

These flags drive feature toggles for:
- Billing flows (Stripe / LemonSqueezy disabled locally)
- Supabase integration (skipped when `OPENSTRAND_ENVIRONMENT=local`)
- Telemetry endpoints (disabled unless explicitly enabled)

In hosted environments set `OPENSTRAND_ENVIRONMENT=cloud` (or `self_hosted`) to unlock cloud-only capabilities.

---

## 6. Cross-Cutting Concerns

- **Authentication**  
  - Local mode: email/password + Argon2  
  - JWT access tokens stored as HttpOnly cookies  
  - OIDC / Supabase support behind feature flags

- **Authorization**  
  - Role-based (Viewer, Contributor, Admin, Owner)  
  - Fine-grained permissions on strands, teams, AI usage

- **Validation**  
  - Zod schemas shared between backend and SDK  
  - `@fastify/ajv-compiler` handles request/response contracts

- **Observability**  
  - Pino structured logging  
  - Health checks at `/health` and `/ready`  
  - Integration points for Sentry / OpenTelemetry (optional)

---

## 7. External Integrations

| Area            | Default       | Optional / Cloud                                                   |
|-----------------|---------------|--------------------------------------------------------------------|
| LLM providers   | BYOK via REST | OpenRouter, OpenAI, Anthropic, Azure OpenAI                        |
| Storage         | Local FS      | AWS S3, Cloudflare R2 (via `STORAGE_PROVIDER`)                     |
| Billing         | Disabled      | Stripe or LemonSqueezy (choose via `BILLING_PROVIDER`)             |
| Authentication  | Local login   | Supabase Auth (OIDC, magic links)                                 |
| Realtime        | Disabled      | WebSocket bridge powered by BullMQ + Redis                        |

All integrations are optional; the OSS distribution runs without network access.

---

## 8. Deployment Scenarios

1. **Local / Offline**  
   - `./start-local.sh` (PGlite, no external dependencies)  
   - Desktop builds via Electron and Capacitor (in progress)

2. **Self-hosted**  
   - Deploy API to Render, Fly.io, Railway, or Kubernetes  
   - Host Next.js app on Vercel/Netlify or as static export  
   - Provide PostgreSQL + Redis (Docker Compose template included)

3. **Managed Cloud (Roadmap)**  
   - Multi-tenant backend  
   - Billing, analytics, SSO  
   - Terraform modules and observability stack

---

## 9. Roadmap Snapshot

- Finish Prisma-backed billing service refactor & tests
- Pluggable vector search providers (Qdrant, Pinecone)
- Graph sync + CRDT collaboration
- Cross-device presence and comments
- Stable Electron build pipeline

See [`docs/ROADMAP.md`](ROADMAP.md) for detailed milestones.

---

## 10. Developer Notes

- Use `npm run typecheck` before commits; workspaces share `tsconfig.base.json`.
- When touching Prisma schema run `npm run prisma:generate --workspace @framers/openstrand-teams-backend`.
- Keep features behind `featureFlags` (frontend) or `FeatureGate` (backend) for gradual rollout.
- Prefer `@framers/openstrand-sdk` helpers when consuming APIs to avoid drifting types.

The architecture is intentionally modular—swap providers, plug in additional queues, or override services by extending the relevant Fastify plugin. Contributions that keep this separation clean are very welcome.
