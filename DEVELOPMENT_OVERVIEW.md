# OpenStrand Monorepo – Development Overview

This project is a single repository that houses both the open-source platform (MIT) and the proprietary Teams Edition. The goals of this document are to help engineers navigate the codebase, understand package boundaries, and know how updates propagate across SDKs, apps, and services.

---

## Repository Layout (Top-Level)

| Path | Purpose | License |
|------|---------|---------|
| `openstrand-app/` | Next.js web app (Community UI, landing pages, composer demos) | MIT |
| `openstrand-admin/` | Internal/enterprise admin UI (Next.js) | Proprietary |
| `packages/openstrand-teams-backend/` | Fastify + Prisma backend powering Teams/Enterprise | Proprietary |
| `packages/openstrand-sdk/` | Public TypeScript client library (community) | MIT |
| `packages/openstrand-sdk/python/` (if present) | Python bindings | MIT |
| `packages/legacy-python/` | Archived Python backend reference | MIT (legacy) |
| `docs/` | Shared documentation (specs, plans, audits) | Mixed |

### Key Backend Subdirectories

- `src/server.ts` – Fastify bootstrap (registers routes & middleware)
- `src/routes/` – Route modules grouped by domain (`strands`, `attachments`, `billing`, etc.)
- `src/services/` – Business logic services (AI, speech, attachments, billing…)
- `prisma/` – Prisma schema and seed scripts
- `tests/` – Vitest suites for unit/integration coverage

### SDK Directories

- `packages/openstrand-sdk/src/` – Browser/Node client for public consumption (`OpenStrandSDK`)
- `packages/openstrand-teams-backend/src/sdk/` – Internal exports (Teams-only services)
  - This is re-exported via `packages/openstrand-teams-backend/src/index.ts` for private dependencies.

---

## Dual SDK Strategy

| Package | Audience | Scope |
|---------|----------|-------|
| `openstrand-sdk` | Community developers, self-hosters | Auth, strands, search, visualization, etc. (open API only) |
| `openstrand-teams-backend/src/sdk` | Teams app, admin tools | Wraps everything in `openstrand-sdk` **plus** proprietary endpoints (billing, advanced AI, attachments, etc.) |

When updating APIs or types, consider both layers:

1. **Backend Route/Service change** → update the Teams SDK first, then decide if the public `openstrand-sdk` needs a mirrored change.
2. **Public API changes** → version bump `openstrand-sdk`, update docs, and ensure compatibility with existing Community installations.
3. **Teams-only features** → expose via internal SDK. Do **not** leak proprietary endpoints into the MIT client.

---

## Development Workflow

1. **Install dependencies**
   ```bash
   npm install              # root – installs workspaces
   npm run dev --workspace openstrand-app
   npm run dev --workspace packages/openstrand-teams-backend
   ```

2. **Run backend locally**
   - Copy `.env.example` → `.env`
   - Ensure `DATABASE_URL` points to Postgres (Docker or local)
   - `npm run dev` inside `packages/openstrand-teams-backend`

3. **Run web app**
   - `npm run dev --workspace openstrand-app`
   - For Teams/admin flows, run `openstrand-admin` similarly.

4. **Testing & linting**
   - `npm run lint` / `npm run test` in relevant workspace
   - `npm run format` (or `npm run format:check`) leverages the shared `prettier.config.cjs`
     to keep TypeScript, JSON, Markdown, and stylesheets aligned across packages
   - Use `read_lints` (IDE integration) after edits
   - Brand assets: `npm run brand:export` regenerates logos, favicons, and OG images

5. **Prisma updates**
   - Modify `schema.prisma`
   - `npx prisma migrate dev` (for real migrations)
   - `npx prisma generate` whenever schema changes
   - Regenerate clients before committing backend + SDK changes

---

## Package Interactions

```
openstrand-app ─┐
                │ uses REST API exposed by → openstrand-teams-backend
openstrand-admin┘                                     │
                                                      ├─ exports SDK (Teams-only)
packages/openstrand-sdk (MIT) ────────────────────────┘
```

- Community UI and external developers consume the public API via `openstrand-sdk` or directly via HTTP.
- Internal products (Teams app, admin tools) often import `@framers/openstrand-teams-backend` and call the proprietary SDK helpers.

---

## Adding New Features – Checklist

1. **Design**
   - Decide if feature is Community, Teams, or both.
   - Catalogue route + service requirements.
2. **Backend**
   - Build/update services (e.g., `speech.service.ts`, `attachments.service.ts`).
   - Add routes under `src/routes`. Register them in `src/server.ts`.
   - Expand Prisma schema + regenerate client.
   - Add unit/integration tests if possible.
3. **SDKs**
   - If public, update `packages/openstrand-sdk` (types + client methods).
   - For Teams-only surfaces, add wrappers in `packages/openstrand-teams-backend/src/sdk` and export.
4. **Front-end / Docs**
   - Integrate into `openstrand-app` or `openstrand-admin` as needed.
   - Update docs (API reference, CHANGELOG, roadmap files).

---

## Environment & Feature Flags

- `.env.example` in Teams backend lists all tunables (AI providers, storage, Redis, queue limits, feature toggles).
- Plan-tier gating lives in the backend (e.g., voice note duration) and is mirrored in front-end messaging/tooltips.
- For Community builds, ensure proprietary routes remain unavailable; only MIT SDK/embed should ship.
- `ADMIN_SERVICE_TOKEN` authenticates the `openstrand-admin` console against the Teams backend. Add it to `packages/openstrand-teams-backend/.env` and mirror the same value in `openstrand-admin` (server-side env only).
- Front-end env variables (set in `openstrand-app/.env.local`):
  - `NEXT_PUBLIC_API_URL` – base API URL (include `/api/v1`) so the landing header and developer console can link to Swagger.
  - `NEXT_PUBLIC_API_DOCS_URL` – optional override for the Swagger host (omit `/docs`, defaults to the API host without `/api/v1`).
  - `NEXT_PUBLIC_SDK_DOCS_URL` – location of generated SDK typings (defaults to `docs/generated`).
  - `NEXT_PUBLIC_FORMSPREE_FORM_ID` – Formspree form identifier powering contact/about CTA forms.
- Front-end build targets:
  - `NEXT_PUBLIC_APP_VARIANT=personal` (default) hides Teams-only surfaces.
  - `NEXT_PUBLIC_APP_VARIANT=team` unlocks collaborative UI, RBAC prompts, and the team onboarding wizard.
  - `NEXT_PUBLIC_OFFLINE_MODE=true` forces the app into local/offline mode (used by `start-local.sh`).
  - `start-local.sh` now exports the above defaults so local runs feel “offline first” while still enabling an optional cloud handoff.
- AI Artisan is unlocked in both Community (offline/local) and Teams editions when operators supply their own provider API keys. Only the shared OpenStrand global keys impose usage quotas.
- Onboarding overlays:
  - `LocalOnboarding` appears when capabilities report `environment.mode === 'offline'`. It walks through SQLite storage, setting a local password, and linking to the cloud-onboarding tutorials if the user wants to sync later.
  - `TeamOnboarding` appears when the app runs in Teams/cloud mode and the store has not been marked complete. The wizard covers infrastructure checks, inviting members, configuring RBAC, and enabling AI/publisher automations with deep links into the tutorial hub.

### Billing telemetry & admin console

- `/api/v1/billing/admin/summary` aggregates Stripe and LemonSqueezy revenue, plan counts, and renewal heatmaps for dashboards.
- `/api/v1/billing/admin/subscriptions` returns a paginated ledger for finance/export workflows; both routes accept the `x-service-token` header.
- The `openstrand-admin` dashboard now renders live data from these endpoints, so deployments must configure Stripe/LemonSqueezy credentials plus `ADMIN_SERVICE_TOKEN` for full parity.
- The admin console ships with a light/dark aware schema explorer (`/dashboard/schema`) that describes Prisma models, relationships, and retention notes for audit reviews.

---

## Useful Docs

- `docs/FINAL_PRODUCTION_READINESS_AUDIT.md` – status matrix.
- `docs/LANDING_PAGE_AND_PROMPT_CHAINING_PLAN.md` – current roadmap for marketing + advanced features.
- `packages/openstrand-teams-backend/CHANGELOG.md` – Teams backend release notes.
- `docs/tutorials/README.md` – tutorial suite index covering analog, deterministic, AI, metadata, and DX/UX guides.

Need a deeper dive? Ping the docs team to expand sections, or create feature-specific design docs under `docs/`.
