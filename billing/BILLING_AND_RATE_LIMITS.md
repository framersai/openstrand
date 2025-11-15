# Billing, Plans, and Rate Limits

This document defines how plans, BYOK, and credit limits are enforced across local/offline and cloud deployments.

## Plans & Pricing

| Plan | Price | Notes |
| ---- | ----- | ----- |
| Community | Free | MIT OSS. Single global workspace (one Loom). Optional sign-in for sync/backups. Same feature set as Teams except multi-tenant/project/user. |
| Teams (Launch) | $1,000 lifetime | Unlimited Looms/projects, RBAC, SSO, governance, deterministic+LLM data intelligence, premium support. Lifetime updates for early buyers. |
| Teams (Power user) | $500 lifetime | Identical to Teams, but for non-commercial/personal creators who still want multi-project tooling. |
| Teams (Student / EDU) | $250 lifetime | Automatic discount for verified `.edu` and non-profit research programs. |
| Teams (Post-launch) | $1,000 license + $100/year support | After launch, new customers get one year of updates; continued updates/support billed annually. |

- Community support channel: GitHub Discussions / Issues (`https://github.com/framersai/openstrand`).
- Teams support: dedicated helpdesk + optional success retainers.
- Upgrade CTAs appear whenever Community users attempt to create a second Loom/project.

## Plan Codes

- `free` – local/offline; optional Supabase auth purely for sync.
- `team`, `enterprise` – self-hosted or managed cloud, multi-user, BYOK, governance.
- `team_power` – discounted SKU for non-commercial makers.
- `team_edu` – EDU/non-profit SKU.

## BYOK vs Global Keys

- BYOK (Bring Your Own Keys):
  - Users/teams configure their own provider API keys server-side (stored encrypted).
  - Usage under BYOK is not counted against global credits. Plan gating still applies to UI and some features.
- Global Keys (hosted/cloud only):
  - Calls route through OpenStrand-managed provider keys.
  - Rate-limited by plan (monthly/rolling credits).

## Enforcement Model

- All AI calls are routed through backend service providers (speech, OCR, LLM, media summary).
- Middleware attaches `usageContext` = { userId, teamId, plan, byok: boolean }.
- Provider services record usage units (e.g., seconds transcribed, tokens, image pages) per context.
- With BYOK = true: usage is recorded for analytics but not decremented from global credits.
- With BYOK = false: decremented from plan credits; 429 returned when exhausted.

## Data Model (DB)

- subscription: plan, status, provider, periodStart/End
- usage_ledger: id, userId/teamId, unit, amount, provider, ts
- credit_balance: scope (user/team), unit, remaining, periodStart/End
- byok_credentials: scope, provider, encryptedKey, lastVerified

## Env & Config

- BILLING_PROVIDER=stripe|lemonsqueezy
- BILLING_PLAN_MAP=… (price IDs per plan)
- ENFORCE_RATE_LIMITS=true
- ALLOW_BYOK=true

## Request Flow

1) Auth → load plan + BYOK state
2) Feature gate (UI + API)
3) Provider call → usage meter → credit decrement if global keys
4) Return result; if exhausted → 429 with upgrade link

## UI/UX

- Plans and limits shown in Dashboard status/Account panel (credits remaining)
- BYOK section to add/manage provider keys (team/user scope)
- Error surfaces link to billing/manage subscription

## Counted Usage Types (examples)

- llm_tokens_in / llm_tokens_out
- embeddings_calls
- image_generation_calls
- speech_transcription_minutes
- ocr_pages
- media_summary_calls

All usage records include metadata: operation, model/provider, isBYOK flag, requestId (when available).

## Assistant Credit Pools (OS / "Weaver")

| Plan / Scope | Included Assistant Credits (global keys) | Notes |
| --- | --- | --- |
| Community (hosted) | 0 assistant_tokens by default | Assistant still runs daily check-ins using heuristics + local/NLP logic. Users/teams can add BYOK keys; optional marketing pool of 2M tokens/month/user can be enabled manually. |
| Teams / Enterprise (managed cloud) | 25M assistant_tokens per month per paying seat | Covers hourly evaluations + daily check-ins on the mini model tier (~$0.24/1M tokens cost). Additional packs (50M tokens = $25) or BYOK for heavy teams. |
| BYOK (any plan) | Unlimited (metered for analytics only) | Usage recorded but not decremented from global credits. Still subject to rate limits to avoid spam. |

- `assistant_tokens` assume mini/instruct model pricing (~$0.15/1M in, $0.60/1M out); internal cost ≈ $0.24/1M tokens, giving 4–6× markup at SaaS seat prices.
- Premium/GPT‑4o tier requests are tracked separately as `assistant_premium_messages` and bill from top-up packs or BYOK only.
- Daily check-ins are enabled for every user (local heuristics if no credits). Hourly evaluations run for teams with available credits; evaluations only emit feed messages when priority thresholds trigger.
- Admins can configure per-team frequency (down to the minute), quiet hours, and whether premium models are allowed.


