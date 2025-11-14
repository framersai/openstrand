# Storage, Backups, and Migrations

## Overview

- Metadata: Postgres (Prisma) in Teams backend
- Attachments: S3‑compatible object storage (Linode Object Storage recommended for current deploy)
- Local/offline: SQL adapter (sql.js in browser; native on desktop/mobile) for drafts/media cache

## Providers

- Linode Object Storage (S3‑compatible)
  - Pros: cost‑effective, S3 API, lifecycle rules
  - Cons: egress fees; regional availability
- AWS S3
  - Pros: mature, wide features, cross‑region replication
  - Cons: cost; egress

## Configuration (Teams backend)

- STORAGE_BACKEND=s3|local|supabase
- S3_BUCKET=…
- S3_REGION=…
- S3_ACCESS_KEY_ID=…
- S3_SECRET_ACCESS_KEY=…
- (Optional) S3_ENDPOINT=https://<linode-region>.linodeobjects.com
- (Optional) S3_FORCE_PATH_STYLE=true (Linode/MinIO)

## What we store in cloud

- Attachments uploaded via Composer/Wizard (images/video/audio)
- Transcripts and media summaries (JSON) attached to strand attachments
- We do not store full user SQL caches; these remain local

## Backups

- Objects: rely on provider lifecycle + replication policies
  - Versioning: enable bucket versioning
  - Replication: mirror to a second region/bucket
- Database: pg_dump scheduled backup (daily)
- Exports: on demand export endpoints for strands/weave/attachments manifest

## Migrations

- Prisma migrations for DB schema
- Object storage is schema‑less; we keep a manifest per strand attachment (ID, path, mime, size)

## Import/Export (CMS‑style)

- Export:
  - Strands JSON (metadata + content)
  - Attachments manifest (S3 keys + signed URLs)
  - Weave graph JSON
- Import:
  - Upload attachments → re‑hydrate strand metadata → rebuild relationships

## Cost guidance

- Linode Object Storage is cost‑effective at low‑to‑mid scale and fine for MVP
- Keep content deduplication enabled server‑side to reduce storage
- Use lifecycle rules to transition infrequently accessed objects to cheaper tiers if available


