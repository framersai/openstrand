# Intelligent Data Pipeline Plan

## Overview
OpenStrand needs to (1) enforce dataset upload limits by plan tier and (2) extract rich metadata summaries so large files can be reasoned about without dumping the entire payload into an LLM context window. This document captures the architectural plan for both requirements so we can implement them incrementally with confidence.

## Goals
- **Plan-aware uploads**: Hard cap uploads per tier (Free <=5 MB, Basic <=50 MB, Pro/Org unlimited aside from backend safety rails).
- **Predictable costs**: Avoid streaming entire CSVs into LLMs; summarize columns/statistics locally and send only structured metadata/prompt context.
- **Deterministic parsing**: Store schema, detected types, and sampling data so visualizations can be generated programmatically (LLM recommends config, not raw data).
  - **Extensible pipeline**: Support future heuristics (e.g., semantic column tagging, anomaly detection) without re-ingesting data.
  - **Structure-aware insights**: Feed detected hierarchies and placeholders into strand structure requests and approvals.

## Recursive Strands Integration

- Schema intelligence now emits suggested structural placements (`ADD_CHILD`, `REATTACH`, `REORDER`) that surface as draft `StrandStructureRequest` payloads.
- Placeholder recommendations map to scope-level placeholder defaults so unauthorized viewers see meaningful context.
- Auto-insights attach provenance metadata to conceptual `StrandLink` records (`provenance='SYSTEM'`) while structural changes stay in approval queues.
- Dataset inspector panels show cascaded visibility states and outstanding structure requests so analysts can triage conflicts quickly.

## Current State (Baseline)
1. Frontend uploads CSV -> `backend/app/api/routes/core.py:/upload`.
2. Backend enforces `config.MAX_UPLOAD_SIZE_MB` (currently 50 MB global).
3. CSV is read entirely into pandas, metadata stored in-memory (app_state).
4. Visualization prompts send raw context (columns + preview rows) to heuristics/LLM.

Limitations:
- No tier-specific gating (everyone shares 50 MB cap).
- No persisted summaries beyond first five rows.
- LLM context grows with dataset complexity.

## Proposed Architecture

### 1. Plan Awareness & Enforcement
| Plan | Upload Cap | Notes |
| ---- | ---------- | ----- |
| Free | 5 MB | Hard block >5 MB. Encourage upgrade modal. |
| Basic | 50 MB | Matches current global limit. |
| Pro / Org | Unlimited (soft limit 200 MB safety) | Enforce via streaming parser & summarizer, not request body. |

Implementation outline:
- Extend Supabase `profiles` (or billing-subscription table) with `plan_tier`.
- Backend middleware/dependency resolves user (optional auth) -> plan.
- Upload endpoint reads `Content-Length` or streaming chunk size:
  ```text
  plan_limits = {'free': 5, 'basic': 50, 'pro': None}
  ```
- If file size exceeds tier limit -> `HTTP 413` with actionable message + upgrade link.
- Pro tier still needs guard: stream to temp file and abort >200 MB to prevent resource exhaustion.

### 2. Intelligent Metadata & Summaries

#### Pipeline Stages
1. **Upload staging**
   - Stream CSV to temp file; capture file stats (size, row count estimate).
   - For pro-tier large files, chunk-read to avoid memory blowup.

2. **Schema detection**
   - Use pandas/pyarrow to infer column types; convert to normalized taxonomy (string, categorical, datetime, numeric).
   - Compute per-column stats (min/max/mean, cardinality, null counts).

3. **Semantic tagging**
   - Lightweight heuristics + optional LLM classification on column names/sample values (e.g., detect geos, currency, dates).

4. **Sample extraction**
   - Store head/tail rows + random sample of up to N rows for debugging.

5. **Summary objects**
   - `DatasetSummary`: high-level description (row count, time span, key measures).
   - `ColumnSummary[]`: type, units, value distribution buckets.
   - `InsightCandidates`: heuristics-driven suggestions (correlations, outliers) computed locally where possible.

6. **LLM prompt context**
   - Instead of raw data, send:
     ```json
     {
       "dataset_summary": "...",
       "columns": [{ "name": "...", "type": "...", "stats": {...}, "semantic_tags": [] }],
       "insight_candidates": [...]
     }
     ```
   - Enforce token budget by truncating text sections deterministically.

7. **Storage**
   - Persist summaries to Supabase (Postgres) or on-disk cache keyed by dataset_id.
   - Keep original CSV path (S3/local) for future streaming if needed.

#### Dataset Notes & Knowledge Strands
- The dashboard inspector now captures notes directly against the active dataset.
- Each save issues:
  1. `POST /strands` with `type: "note"` and TipTap JSON stored in `content.data`.
  2. `POST /strands/:id/relationships` linking the note back to the dataset with `type: "references"`.
- The client stores `content.metadata.plainText` so the note can be previewed inline without a TipTap render pass. When older strands are opened, the UI backfills this metadata and updates the strand in-place.
- Recent notes are fetched from `POST /weave/subgraph` (depth = 1) and displayed in the inspector with quick actions to reopen in the full composer or navigate to the PKMS strands view.

### 3. Visualization Flow Adjustments
1. Prompt arrives -> fetch cached summaries.
2. Heuristic/LLM uses metadata to choose chart config.
3. Visualization factory queries actual data slices (from pandas or pre-aggregated caches) only when needed for rendering.
4. For Pro/Org, add optional "Deep dive" button to re-run summarization or fetch additional sample slices.

## Implementation Plan

### Phase 1 - Tier Enforcement (Backend + Frontend Messaging)
1. **Backend**
   - Add `PlanTier` enum + `get_plan_limits(plan)` helper.
   - Update `/upload` to read file size before pandas load.
   - Return descriptive errors (`detail: {"limit_mb": 5, "plan": "free"}`).
2. **Frontend**
   - Surface toast + modal when limit exceeded, linking to `/billing`.
   - Show current plan + remaining quota in Upload panel (requires `/auth/me` returning plan).
3. **Docs**
   - Update README + docs/PRICING_ANALYSIS.md with new caps.

### Phase 2 - Metadata & Summaries
1. Create `backend/services/datasets/summary_service.py`.
2. Implement streaming ingestion:
   - Use `csv.Sniffer` or pandas chunked reader.
   - Build column stats incrementally.
3. Design summary schema (Pydantic models) and persist (Supabase table `dataset_summaries`).
4. Update `DatasetMetadata` to include summary refs and store in `app_state`.
5. Adjust `api.generateVisualization` to send summary to heuristics/LLM.

### Phase 2.5 – Data Intelligence Service (TypeScript Monorepo)
- Introduce `DataIntelligenceService` inside `packages/openstrand-teams-backend/src/services/data-intelligence/`.
  - Wraps the existing `SchemaIntelligenceService` for dataset summaries.
  - Reuses the new SDK helper `modules/dataIntelligence.module.ts` for deterministic vocabulary/NER stats that run without any LLM.
- Responsibilities:
  1. Tokenise uploaded strands/datasets, compute TF/IDF, co-occurrence, bigrams, and collection-level term drift.
  2. Run heuristic entity extraction (capitalisation + pattern matching) to produce `metadata.entities`.
  3. Persist summaries per collection/team so vocabulary studio, Looms, and visualization services can query them rapidly.
  4. Offer optional “LLM verification” hooks that run only when a Pro/Enterprise workspace enables them.
- Tests: live alongside the service to guarantee statistical helpers remain deterministic.

### Library Strategy (Offline by default)
- **Tokenization / NER / POS** → Use [`wink-nlp`](https://winkjs.org/) or [`compromise`](https://github.com/spencermountain/compromise) in both the backend and the electron/capacitor runtimes so we are not hand-rolling NLP.
- **Stop word lists** → Reuse wink’s curated lists, extend via configuration (no bespoke arrays scattered across the repo).
- **Extractive summarisation** → Default to `wink-nlp`’s sentence scoring (text-rank style) for notes/datasets; abstractive/LLM summaries become an opt-in verification step that piggybacks on the deterministic summary payload.
- **Statistical helpers** → Prefer `scan-stats`, `ml-distance`, or d3-array utilities for histograms, PMI, correlations instead of custom math.
- **Documentation** → Every NLP helper exported through the SDK must have TSDoc describing which third-party library is powering it, expected language coverage, and how to swap in a different model if vendors require it.

### Phase 3 - Semantic Tagging & Heuristic Insights
1. Add heuristics for currency/date recognition.
2. Optional LLM call (cheap model) for ambiguous columns (bounded tokens).
3. Generate `InsightCandidate` list (e.g., correlations > threshold).
4. Expose summary + insights in UI (collapsible panel in Upload tab).

### Phase 4 - Scalability & Future Work
- Stream processing for >500 MB (chunked aggregator).
- Background jobs to re-summarize when dataset updates.
- Column-level security if multi-tenant.
- Schema versioning to support data lineage.

## Testing Strategy
- Unit tests for `plan_limits` helper.
- Integration tests:
  - Upload 4 MB as Free -> success.
  - Upload 6 MB as Free -> 413 + upgrade hint.
  - Upload 60 MB as Basic -> 413; as Pro -> success.
- Snapshot tests for summary JSON.
- Contract tests ensuring `VisualizationRequest` uses summary structure.

## Open Questions / Decisions Needed
1. **Plan source of truth** - Supabase profile vs. billing webhook table?
2. **Storage target** - Keep pandas DataFrame in memory vs. persist to DuckDB/Parquet?
3. **LLM provider for semantic tagging** - Use existing provider with cheap model or add new inference endpoint?
4. **UI exposure** - Where to surface dataset summaries (Upload tab, separate sidebar, etc.)?
5. **Security** - Need encryption-at-rest for uploaded CSVs before storing beyond temp directory?

Once we agree on this plan we can implement Phase 1 (tier enforcement) first, then proceed phase-by-phase.



