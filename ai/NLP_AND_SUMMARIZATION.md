# NLP & Summarization Strategy

**Last Updated:** November 2025  
**Scope:** Deterministic text processing, entity extraction, and summarization for Community + Teams editions

---

## Overview

OpenStrand uses **deterministic, offline-first NLP** for vocabulary building, entity extraction, and extractive summarization. LLM-based abstractive summarization is optional and runs as a verification/enrichment layer.

This approach ensures:
- **Zero LLM dependency** for core features (Community Edition works 100% offline)
- **Predictable costs** (no surprise API bills)
- **Fast processing** (statistical methods run in <100ms per document)
- **Reproducible results** (same input → same output, always)

---

## Library Strategy

### Current Implementation (Handwritten)

The initial implementation (`packages/openstrand-sdk/src/modules/dataIntelligence.module.ts`) uses handwritten tokenization, TF/IDF scoring, and regex-based NER. This works but lacks:
- Stemming/lemmatization
- Part-of-speech tagging
- Sentence boundary detection
- Advanced entity recognition

### Recommended Libraries (Roadmap)

| Library | Purpose | Size | License | Offline | Notes |
|---------|---------|------|---------|---------|-------|
| **wink-nlp** | Tokenization, stemming, POS, NER | ~400KB | MIT | ✅ | Fast, zero-dependency, browser + Node |
| **compromise** | NER, sentence parsing, tagging | ~200KB | MIT | ✅ | Lightweight, good for casual text |
| **natural** | Stemming, TF/IDF, classifiers | ~1MB | MIT | ✅ | Mature, Node-focused |
| **d3-array** | Statistical helpers (quantile, mean, etc.) | ~50KB | ISC | ✅ | Already used for visualizations |
| **stopword** | Multilingual stop-word lists | ~100KB | MIT | ✅ | Supports 60+ languages |

**Recommendation:** Adopt **wink-nlp** for production. It's actively maintained, has excellent TypeScript support, and runs identically in browser/Electron/Capacitor/Node.

---

## Extractive Summarization (Default)

Extractive summarization selects the most important sentences from the original text without rewriting. This is the **default** for all note/document strands.

### Algorithm: TextRank

1. **Sentence Tokenization**: Split document into sentences (wink-nlp or compromise).
2. **Sentence Embedding**: Represent each sentence as a TF/IDF vector.
3. **Similarity Graph**: Build weighted graph where edges = cosine similarity between sentences.
4. **PageRank**: Run iterative ranking to score sentences by centrality.
5. **Selection**: Pick top N sentences (configurable, default 3–5) and return in original order.

### Implementation Plan

```typescript
// packages/openstrand-sdk/src/modules/summarization.ts

export interface ExtractiveSummaryOptions {
  maxSentences?: number;      // Default: 5
  minSentenceLength?: number;  // Default: 10 words
  diversityPenalty?: number;   // 0.0–1.0, penalize similar sentences
}

export function extractiveSummarize(
  text: string,
  options?: ExtractiveSummaryOptions
): {
  summary: string;
  sentences: Array<{ text: string; score: number; position: number }>;
  method: 'textrank';
} {
  // 1. Tokenize sentences
  // 2. Build TF/IDF vectors
  // 3. Compute similarity matrix
  // 4. Run PageRank
  // 5. Select top sentences
  // 6. Return in original order
}
```

### Backend Integration

- Hook into strand create/update: if `strandType === 'document' || strandType === 'note'`, enqueue extractive summary job.
- Store result in `strand.metadata.extractiveSummary` (plain text + sentence scores).
- Display in strand cards, search results, and knowledge graph node tooltips.

---

## Abstractive Summarization (Optional LLM)

Abstractive summarization uses LLMs to generate new text that captures the essence of the document. This is **optional** and runs as a verification/enrichment step.

### When to Use

- User explicitly requests "AI summary" in composer or settings
- Teams Edition with LLM verification enabled
- Long documents (>5k words) where extractive summaries are too fragmented

### Prompt Engineering Best Practices

```typescript
// packages/openstrand-teams-backend/src/services/summarization/abstractive.service.ts

const ABSTRACTIVE_SUMMARY_PROMPT = `You are a research assistant creating concise summaries.

## Instructions
- Read the document carefully
- Identify the main thesis and key supporting points
- Write a 2-3 paragraph summary in your own words
- Preserve technical terms and proper nouns exactly
- Use active voice and clear, academic language
- Do NOT add opinions or external information

## Memory Technique
Before writing, mentally organize the content into:
1. Core thesis/claim
2. Supporting evidence (3-5 key points)
3. Implications or conclusions

## Document
{document_text}

## Summary
`;
```

### Cost Optimization

- Always run extractive summary first; use it as context for the LLM prompt to reduce token count.
- Cache abstractive summaries keyed by `contentHash` (same as vocabulary summaries).
- Offer "Quick" (extractive only) vs "Deep" (abstractive) options in UI.

---

## Entity Extraction (NER)

### Current Implementation

Regex-based patterns in `dataIntelligence.module.ts`:
- Title case sequences → `person` or `organization`
- Acronyms (2+ uppercase letters) → `acronym`
- Heuristic rules for `Inc`, `LLC`, `University`, etc.

### Roadmap: wink-nlp Integration

```typescript
import winkNLP from 'wink-nlp';
import model from 'wink-eng-lite-web-model';

const nlp = winkNLP(model);

export function extractEntities(text: string): EntityCandidate[] {
  const doc = nlp.readDoc(text);
  const entities: EntityCandidate[] = [];

  doc.entities().each(entity => {
    entities.push({
      value: entity.out(),
      type: mapWinkType(entity.type()),
      count: 1,
      confidence: 0.9,
    });
  });

  return entities;
}
```

### Supported Entity Types

- `person`: People (John Doe, Marie Curie)
- `organization`: Companies, institutions (OpenAI, MIT)
- `location`: Cities, countries, addresses
- `acronym`: Abbreviations (NASA, HTTP)
- `keyword`: Domain-specific terms (extracted via TF/IDF)

---

## Stop Words & Multilingual Support

### Default Stop Words

English stop words are hardcoded in `dataIntelligence.module.ts`. For production, use the `stopword` library:

```typescript
import { removeStopwords, eng, spa, jpn } from 'stopword';

export function tokenizeWithStopwords(text: string, language: 'en' | 'es' | 'ja' = 'en'): string[] {
  const tokens = text.toLowerCase().split(/\s+/);
  const stopwordList = language === 'en' ? eng : language === 'es' ? spa : jpn;
  return removeStopwords(tokens, stopwordList);
}
```

### Language Detection

Use `wink-nlp` or `franc-min` (lightweight language detector) to auto-detect document language before applying stop words.

---

## Performance & Caching

### Content Hashing

All analysis results are keyed by SHA-256 hash of the input content:

```typescript
import { createHash } from 'crypto';

function computeContentHash(content: unknown): string {
  return createHash('sha256')
    .update(JSON.stringify(content))
    .digest('hex');
}
```

### Cache Strategy

1. **Vocabulary summaries**: 60-minute TTL (configurable in settings)
2. **Dataset analyses**: 24-hour TTL
3. **Extractive summaries**: Permanent (stored in strand metadata)
4. **Abstractive summaries**: 7-day TTL (LLM results can drift with model updates)

### Incremental Updates

- On strand update, compare `contentHash` with cached value
- If unchanged, skip re-analysis
- If changed, enqueue background job (BullMQ)
- Collection-level summaries merge individual strand summaries (no full recompute)

---

## API Surface

### SDK Client Methods

```typescript
import { OpenStrandSDK } from '@framers/openstrand-sdk';

const sdk = new OpenStrandSDK({ baseUrl: 'http://localhost:8000' });

// Local (deterministic, instant)
const vocab = sdk.dataIntelligence.summarizeLocal(documents, options);

// Remote (with caching + optional LLM)
const vocab = await sdk.dataIntelligence.summarizeRemote({ documents, options });

// Dataset analysis (heuristics + optional AI)
const analysis = await sdk.dataIntelligence.analyzeDataset({ rows, options });
```

### Backend Endpoints

```
POST /api/v1/intelligence/vocabulary
POST /api/v1/intelligence/collections/summary
POST /api/v1/intelligence/datasets/analyze
POST /api/v1/intelligence/jobs
GET  /api/v1/intelligence/jobs/:id
GET  /api/v1/intelligence/jobs/:id/stream (SSE)
```

---

## Electron & Capacitor Integration

All NLP libraries run identically in:
- **Browser** (Next.js client-side)
- **Electron** (desktop app)
- **Capacitor** (iOS/Android)

### Bundle Considerations

- wink-nlp models: ~400KB (acceptable for desktop/mobile)
- stopword lists: ~100KB (lazy-load per language)
- Total NLP bundle overhead: ~500KB (gzipped)

### Offline Guarantees

- No network calls required for deterministic analysis
- LLM verification gracefully degrades (shows "Verification unavailable offline" badge)
- All processing happens in main thread or Web Workers (no external services)

---

## Testing Strategy

### Unit Tests

- Tokenization edge cases (unicode, punctuation, mixed case)
- TF/IDF scoring accuracy (compare against known datasets)
- Entity extraction precision/recall (test against labeled corpus)
- Extractive summarization coherence (manual review of top 10 sentences)

### Integration Tests

- End-to-end: Upload document → verify extractive summary in metadata
- Cache invalidation: Update strand → verify new summary generated
- Job queue: Enqueue analysis → poll status → verify result persisted

### Performance Benchmarks

| Operation | Input Size | Target Time | Current |
|-----------|------------|-------------|---------|
| Vocabulary (10 docs) | 50KB total | <100ms | ✅ 45ms |
| Entity extraction | 10KB doc | <50ms | ✅ 23ms |
| Extractive summary | 5KB doc | <200ms | 🚧 TBD |
| Dataset analysis | 1000 rows × 10 cols | <500ms | ✅ 312ms |

---

## Documentation & User Education

### In-App Help

- Add tooltip on "Extractive" vs "Abstractive" toggle in composer
- Settings page explains deterministic vs LLM approaches
- Knowledge graph shows "Verified by LLM" badges when applicable

### Public Docs

- Tutorial: "Building Vocabularies Without LLMs" (`docs/tutorials/offline-analytics-guide.md`)
- API reference updated with new endpoints (`docs/API_REFERENCE.md` ✅)
- Architecture doc explains NLP stack (`docs/INTELLIGENT_DATA_PIPELINE.md` ✅)

---

## Roadmap

### Phase 1 (Current)
- ✅ Handwritten TF/IDF + regex NER
- ✅ Vocabulary summaries (terms, bigrams, entities)
- ✅ Dataset schema analysis (heuristics + optional AI)
- ✅ Background job queue + caching

### Phase 2 (Next)
- 🚧 Integrate wink-nlp for production NER
- 🚧 Implement TextRank extractive summarization
- 🚧 Add multilingual stop-word support
- 🚧 Expose extractive summaries in strand cards

### Phase 3 (Future)
- Keyphrase extraction (RAKE, YAKE)
- Topic modeling (LDA, NMF) for collection clustering
- Sentiment analysis (VADER, lexicon-based)
- Citation/reference extraction for academic papers

---

## Security & Privacy

- All NLP processing happens locally (Community) or in user's backend instance (Teams)
- No text data sent to third-party services unless user explicitly enables LLM verification
- Content hashes are one-way (cannot reverse-engineer original text)
- Cached summaries respect strand visibility (private summaries never leak to other users)

---

## References

- wink-nlp: https://github.com/winkjs/wink-nlp
- compromise: https://github.com/spencermountain/compromise
- TextRank paper: https://web.eecs.umich.edu/~mihalcea/papers/mihalcea.emnlp04.pdf
- TF/IDF: https://en.wikipedia.org/wiki/Tf%E2%80%93idf
- stopword library: https://github.com/fergiemcdowall/stopword

---

_This document will evolve as we integrate production NLP libraries. Update it when switching from handwritten helpers to third-party packages._

