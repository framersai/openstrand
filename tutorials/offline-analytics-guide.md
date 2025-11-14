# Modeling Strands Without LLM Integrations

> Build rich analytical strands when generative AI is restricted or undesirable.

## Why This Matters

Regulated environments (finance, healthcare, on-prem clients) often prohibit outbound AI calls. This guide demonstrates how to enrich strands using deterministic methods: statistical baselines, extractive summarisation, and domain heuristics.

## Personas

- Data or research teams with strict compliance rules.  
- Engineering teams building offline-first or air-gapped deployments.  
- Analysts who prefer interpretable transformations over opaque LLM outputs.

## Learning Outcomes

1. Build strands that describe datasets, experiments, or reports with reproducible metadata.  
2. Generate extractive summaries, topic tags, quality metrics, and duplicates detection using deterministic methods.  
3. Configure OpenStrand services (cache, queues, SDK) for offline pipelines.  
4. Document governance pipelines for later automation.

## Architecture Overview

```
source files ─► ingestion jobs (bullmq OR cron)
            └► offline enrichment:
                 ├─ statistical profiling (mean/max/variance)
                 ├─ keyword extraction (RAKE / TextRank)
                 ├─ extractive summary (SumBasic / BM25 centroid)
                 └─ schema checks (zod, custom validators)
             └► OpenStrand strand creation
```

## Step 1 – Dataset Profiling Strand

1. **Load dataset** (CSV/JSON) into a profiling script. Use libraries such as `pandas-profiling`, `duckdb`, or custom Node modules to compute summary statistics (mean, median, count, null %, etc.).  
2. **Generate a `DatasetSummary`** object containing: dataset ID, schema, descriptive stats, anomalies (null spikes, outliers).  
3. **Create/Update Strands** via `openstrandAPI.strands`: note type `reference`, content includes summary tables, `metadata.profile` carries JSON stats, `metadata.flags` notes anomalies.  
4. **Attach Raw Sample** – store small representative slices using `attachments.service` for offline review.

### Script Sketch (Node + DuckDB)

```ts
import { Database } from 'duckdb';
import { openstrandAPI } from '@/services/openstrand.api';

async function profileDataset(path: string) {
  const db = new Database(':memory:');
  const conn = db.connect();
  conn.run(`CREATE TABLE data AS SELECT * FROM read_csv_auto('${path}')`);
  const profile = conn.run(`SELECT avg(col), min(col), max(col) FROM data`);

  await openstrandAPI.strands.create({
    title: `Profile: ${path}`,
    noteType: 'reference',
    content: { data: profile },
    metadata: { profile },
  });
}
```

## Step 2 – Extractive Summaries

Use deterministic algorithms to summarise without LLMs.

### Recommended Techniques

- **TextRank** (graph-based sentence ranking).  
- **SumBasic** (frequency-based).  
- **BM25 centroid** (retrieve top sentences using similarity to overall document centroid).  
- **Domain heuristics** (if you know key sections: abstract, findings, recommendations).

### Pipeline

1. Tokenise strands with `natural`, `compromise`, or `nltk`.  
2. Remove stop words, compute sentence vectors (TF-IDF).  
3. Rank sentences, pick top `n`.  
4. Save results under `metadata.extractiveSummary` with provenance (algorithm, version).  
5. Update `summary` field if empty, but keep manual summaries as authoritative.

### Example (Python + SumBasic)

```python
from sumbasic import summarize
from openstrand_sdk import OpenStrand

sdk = OpenStrand(token='...')

def enrich_strand(strand_id: str, text: str):
    summary = summarize(text, sentences=4)
    sdk.strands.update(
        strand_id,
        {
            "metadata": {
                "extractiveSummary": {
                    "sentences": summary,
                    "algorithm": "sumbasic",
                }
            }
        }
    )
```

## Step 3 – Keyword & Topic Extraction

1. Implement RAKE or YAKE for multi-word key phrases.  
2. Map keywords to controlled vocab (Zettelkasten note types, domain-specific ontologies).  
3. Store as `metadata.tags` and `metadata.ontology`.  
4. Provide hints to the deduplication service for text similarity (n-gram hashing, MinHash).

## Step 4 – Duplication & Similarity Checks

1. Compute TF-IDF vectors or use locality-sensitive hashing for near duplicates.  
2. Store similarity scores in `metadata.similarStrands` (array of `{ id, score, rationale }`).  
3. When uploading analog notes, run the deduplication service to warn editors before linking duplicates.

## Step 5 – Governance & RBAC

- Enforce `visibility` and `teamId` fields based on compliance rules.  
- Use `FeatureFlag` to hide AI-specific UI toggles when the offline mode flag is enabled.  
- Log enrichment operations in a `NoteActivity` strand or use `activity.service`.  
- Versioning: store `metadata.enrichmentVersion` to track algorithm updates.

## Developer & Ops Notes

- **BullMQ Queues**: configure the `queue.service.ts` to run offline enrichment jobs at off-peak hours.  
- **Caching**: use Redis or in-memory caches to store precomputed profiles and summaries.  
- **Testing**: create Vitest suites covering deterministic outputs for known inputs (see `tests/unit/schema-intelligence.service.test.ts`).  
- **CLI**: build Node CLI commands under `packages/openstrand-teams-backend/scripts` to trigger profiling or summarisation.

## UX Deliverables

- Provide mini charts (sparkline, histogram) in the dashboard `DatasetInspectorPanel`.  
- Display algorithm provenance (badge with “Extracted via TextRank v1.2”).  
- Offer toggles for human review/override before publish.

## Next

- Proceed to [LLM-Augmented Workflows](./llm-augmentations.md) to layer on generative tooling when permitted.  
- Pair with the [Metadata Architecture Playbook](./metadata-architecture.md) for consistent tagging and authorship policies.


