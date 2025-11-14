# Environment Variables

Create a `.env` for the backend and configure the following. For frontend, set `NEXT_PUBLIC_API_URL` at the app level.

## Frontend (Next.js)
- NEXT_PUBLIC_API_URL: Default `http://localhost:8000/api/v1`

## Backend (Fastify)
- PORT, HOST, ENABLE_SWAGGER
- CORS_ORIGINS
- DATABASE_URL
- JWT_SECRET

## Object Storage (S3-compatible)
- STORAGE_BACKEND=s3
- S3_BUCKET, S3_REGION
- S3_ACCESS_KEY_ID, S3_SECRET_ACCESS_KEY
- S3_ENDPOINT (Linode example: `https://us-east-1.linodeobjects.com`)
- S3_FORCE_PATH_STYLE=true (Linode/MinIO)

## AI Providers (Global keys)
- OPENAI_API_KEY, OPENAI_ORGANIZATION
- ANTHROPIC_API_KEY
- OLLAMA_BASE_URL

## Speech / OCR
- SPEECH_PROVIDER=openai
- OPENAI_SPEECH_MODEL=whisper-1
- SPEECH_COST_PER_MINUTE_USD=0.006
- ATTACHMENT_TRANSCRIBE_CONCURRENCY=2
- ENABLE_OCR=true
- OCR_LANGUAGE=eng
- OCR_PREPROCESSING=true
- OCR_COST_PER_PAGE_USD=0.002

## Billing
- BILLING_PROVIDER=stripe|lemonsqueezy
- STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
- STRIPE_PRICE_FREE, STRIPE_PRICE_CLOUD, STRIPE_PRICE_PRO, STRIPE_PRICE_TEAM, STRIPE_PRICE_ENTERPRISE
- LEMONSQUEEZY_API_KEY, LEMONSQUEEZY_WEBHOOK_SECRET
- ENFORCE_RATE_LIMITS=true

## BYOK Encryption
- ENCRYPTION_KEY (64 hex chars; generate with `openssl rand -hex 32`)

## Admin bootstrap (optional)
- ADMIN_DEFAULT_USERNAME
- ADMIN_DEFAULT_PASSWORD
- ADMIN_DEFAULT_EMAIL (optional)

## Frontend URL
- FRONTEND_URL (returnUrl for portals)

See also:
- Billing, plans, and rate-limits: `docs/BILLING_AND_RATE_LIMITS.md`
- Storage and backups: `docs/BACKUPS_AND_STORAGE.md`
- Adapter standardization: `docs/DEVELOPMENT_OVERVIEW.md`
