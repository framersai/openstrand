# Landing Page Refresh & Prompt Chaining Roadmap

## Landing Page Refresh (Q4 Escort)

- **Narrative Flow**
  - Hero (value proposition, “ingest → analyse → publish” animation, primary CTA).
  - Product story (PKMS foundations, AI layers, collaboration, enterprise automation).
  - Proof (customer badges / trust deck, security & governance callouts).
  - Pricing strip (Community vs Teams/Enterprise) + secondary CTA.
  - Footer with resource index & compliance badges.

- **Experience & Motion**
  - Animated hero canvas highlighting live knowledge graph overlays.
  - Scroll-triggered demo showing dataset import → auto insights → publishing → custom domains.
  - Inline voice note demo surfaced via the new composer UI.
  - Responsive breakpoints: mobile one-column, tablet hero animation simplified, desktop parallax layers.

- **Content System**
  - All copy & iconography drawn from `landing/config.ts` so marketing can localize quickly.
  - Shared CTA component wired to Supabase auth (Community) and Stripe checkout (Teams/Enterprise).
  - SEO schema bits (FAQ, HowTo) auto-generated from config for structured data.
  - Feature tile for “Deterministic Data Intelligence” that explains vocab/NER analytics (offline) plus the optional “LLM verification” upgrade badge for Teams/Enterprise.
  - Pricing ribbons:
    - “Community Edition – Free • One global Loom • Offline forever.”
    - “Teams Edition – $1,000 lifetime (launch) / $500 power-user / $250 student. Multiple Looms, RBAC, governance.” Include countdown to when pricing switches to annual $100 upgrades.

- **Metrics & Experimentation**
  - Pixel for CTA click tracking (mixpanel segment).
  - Heatmap-friendly layout for hero, features, testimonials (Hotjar instrumentation).
  - Variant-ready hero copy blocks for future A/B testing.

## Prompt Chaining Feature (Teams Backbone)

- **Architecture**
  - `PromptChain` (head) → `PromptStep` (node) → `PromptRun`/`PromptRunLog` (telemetry).
  - Steps support: LLM call, tool invocation, dataset transformer, branch/merge, conditionals.
  - Execution orchestrated by queue workers with resumable state (BullMQ + Redis already in place).

- **Authoring & UX**
  - Visual builder (React Flow) for Teams; simplified linear template editor for Community.
  - Variable injection (plan metadata, dataset context, prior voice transcripts).
  - Versioning + draft/published states; chain library with tagging.

- **Observability & Guardrails**
  - Per-step logging (prompt, cost, latency) persisted to `PromptRunLog`.
  - Hard budget + token ceilings per chain; automatic abort + notification on breach.
  - Manual replay / resume for failed steps.

- **Licensing Model**
  - **Teams / Proprietary Backend**: full chaining (branching, BYOK providers, scheduling, analytics dashboards, shared libraries). Covered by the lifetime launch license; after launch, upgrades follow the annual support plan (~$100).
  - **Community / MIT Stack**: linear chains up to 3 steps, no parallel branches, no shared libraries, capped monthly executions (provides taste without enterprise automations). Restricted to the single global Loom.
  - Gating enforced via plan tier + feature flags; landing page messaging reflects tier capabilities.

- **Delivery Phasing**
  1. Backend primitives + API surface (Q1).
  2. Teams builder UI + integration hooks into composer/voice attachments (Q1).
  3. Community linear templates + “upgrade” path (Q2).
  4. Analytics dashboards + alerting (Q2).

- **Dependencies**
  - Reuse new attachment transcripts for chain variables.
  - Continue leveraging Redis queue + CacheService for run acceleration.
  - Align cost tracking with existing `CostRecord` model.

---
**Next steps**: content design mock-ups for hero animation (Figma), finalize PromptChain Prisma schema, stand-up builder scaffold, update landing copy deck for localization.
