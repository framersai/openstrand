# OpenStrand Packaging & Distribution

_Last updated: November 2025_

OpenStrand is distributed as a single npm workspace. Every deployable artifact is built from the same repository to guarantee version alignment and reproducibility.

---

## Monorepo Packages

| Package / App                     | npm name / output                 | License | Notes                                                   |
|----------------------------------|-----------------------------------|---------|--------------------------------------------------------|
| `openstrand-app/`                | — (Next.js app)                   | MIT     | User-facing PKMS experience. Deployed as static build or Vercel app. |
| `packages/openstrand-teams-backend/` | `@openstrand/teams-backend`     | MIT     | Fastify API + Prisma. Publishes compiled Node bundle and type defs. |
| `packages/openstrand-sdk/`       | `@openstrand/sdk`                 | MIT     | TypeScript SDK + shared schemas (CJS + ESM + `.d.ts`). |
| `openstrand-admin/`              | — (optional)                      | UNLICENSED | Private admin UI kept out of OSS distribution.        |

All packages inherit tooling from the root workspace (`tsconfig.base.json`, ESLint, Prettier).

---

## Build Outputs

### Backend (`@openstrand/teams-backend`)
- Targets Node 18 LTS.
- Emits `dist/` with compiled JS + type declarations.
- Includes Prisma schema and migrations (`prisma/`).
- Production start command: `node dist/server.js`.

### SDK (`@openstrand/sdk`)
- Bundled with `tsup`.
- Dual package export map:
  ```
  import { OpenStrandClient } from '@openstrand/sdk'          // ESM
  const { OpenStrandClient } = require('@openstrand/sdk')     // CommonJS
  ```
- Generated types stay in sync with backend contracts.

### Frontend (`openstrand-app`)
- Uses Next.js `app` router.
- Build command: `npm run build --workspace openstrand-app`.
- Deployed either as a Next server (SSR) or exported static bundle behind a CDN.

---

## Publishing Workflow

1. Bump versions uniformly across packages (use Changesets or manual `npm version`).
2. `npm run build` to compile every workspace.
3. Publish artifacts:
   ```bash
   npm publish --workspace @openstrand/sdk
   npm publish --workspace @openstrand/teams-backend
   ```
4. Tag the release (`git tag vX.Y.Z`) and push.
5. Deploy frontend via CI (Vercel, Netlify, or static hosting).

> Tip: keep the SDK release ahead of the backend when introducing new client features. Consumers can pin exact versions via npm semver.

---

## Docker Images (Optional)

Example Dockerfile for the backend:

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
COPY packages/openstrand-teams-backend ./packages/openstrand-teams-backend
RUN npm install --omit=dev
RUN npm run build --workspace @openstrand/teams-backend

FROM node:18-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/packages/openstrand-teams-backend/dist ./dist
COPY packages/openstrand-teams-backend/prisma ./prisma
ENV NODE_ENV=production
CMD ["node", "dist/server.js"]
```

---

## Versioning Strategy

- **Semantic Versioning** across all publishable packages.
- Sync major versions between backend and SDK to reflect API compatibility.
- Frontend sticks to calendar-based tags (e.g. `2025.11.0`) but depends on the same git tag.

---

## Supported Deployment Targets

- **Local / OSS:** `./start-local.sh` (PGlite, no external services required).
- **Self-hosted Cloud:** Docker, Fly.io, Railway, Render, or Kubernetes.
- **Static Frontend Hosting:** Vercel, Netlify, Cloudflare Pages, or static S3 bucket.

Set `OPENSTRAND_ENVIRONMENT` + `OPENSTRAND_IS_CLOUD` per environment so the app toggles billing and telemetry appropriately.

---

## Artefact Checklist

| Artefact       | Description                          | Source command                                    |
|----------------|--------------------------------------|---------------------------------------------------|
| SDK package    | npm tarball (`@openstrand/sdk`)      | `npm run build --workspace @openstrand/sdk`       |
| Backend bundle | Node bundle + Prisma schema          | `npm run build --workspace @openstrand/teams-backend` |
| Frontend app   | Next.js build output (`.next/`)      | `npm run build --workspace openstrand-app`        |
| Docker image   | Optional runtime image               | `docker build -f Dockerfile.backend .`            |

Keep everything driven by the monorepo to avoid drift between clients, API, and documentation.
