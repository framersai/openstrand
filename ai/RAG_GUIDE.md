# RAG (Retrieval-Augmented Generation) Guide

**Last Updated:** November 14, 2025  
**Status:** Production Ready  
**Editions:** Community (local only) | Teams (local + cloud)

---

## Overview

OpenStrand's RAG system lets you ask natural language questions and get answers grounded in your knowledge base. Instead of hallucinating, the LLM only answers based on retrieved context from your strands.

**Key Features:**
- 🔍 Semantic search using vector embeddings
- 💬 Natural language Q&A with citations
- 🏠 Local models (Ollama) or cloud (OpenAI/Anthropic)
- 📚 Works across all strands in a Loom
- 💰 Cost tracking for cloud providers
- ⚡ Async/batched for responsiveness

---

## How It Works

### 1. Embedding (Indexing)
```
Strand Text
    ↓
Chunking (512 tokens, 50 overlap)
    ↓
Generate Embeddings (768-dim vectors)
    ↓
Store in Database (Prisma)
```

### 2. Retrieval (Search)
```
User Question
    ↓
Generate Query Embedding
    ↓
Cosine Similarity Search
    ↓
Top-K Most Relevant Chunks
```

### 3. Generation (Answer)
```
Retrieved Chunks
    ↓
Build Prompt with Context
    ↓
LLM Generates Answer
    ↓
Return with Citations
```

---

## Quick Start

### 1. Enable RAG in Settings
Navigate to `/settings/data-intelligence` and:
1. Toggle "Enable RAG"
2. Choose embedding provider (Local/OpenAI)
3. Choose LLM provider (Local/OpenAI/Anthropic)
4. Optionally enable "Auto-embed new strands"

### 2. Embed Your Strands
```bash
# Via API
curl -X POST http://localhost:8000/api/v1/rag/embed \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"strandId": "strand-123", "loomId": "loom-456"}'

# Via SDK
const sdk = new OpenStrandSDK({ baseUrl: 'http://localhost:8000' });
await sdk.rag.embedStrand({
  strandId: 'strand-123',
  loomId: 'loom-456'
});
```

### 3. Ask Questions
In the PKMS dashboard, use the Ask Bar:
```
🔍 Ask a question...
> What are the main themes in my research notes?
```

Or via API:
```bash
curl -X POST http://localhost:8000/api/v1/rag/query \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "question": "What are the main themes?",
    "loomId": "loom-456"
  }'
```

---

## API Reference

### POST /api/v1/rag/embed
Embed a strand for semantic search

**Request:**
```json
{
  "strandId": "strand-123",
  "loomId": "loom-456",
  "force": false
}
```

**Response:**
```json
{
  "success": true,
  "chunks": 12,
  "message": "Embedded 12 chunks"
}
```

### POST /api/v1/rag/search
Semantic search for relevant chunks

**Request:**
```json
{
  "query": "quantum computing applications",
  "loomId": "loom-456",
  "k": 5,
  "minScore": 0.7
}
```

**Response:**
```json
{
  "results": [
    {
      "id": "chunk-abc",
      "text": "Quantum computing uses qubits...",
      "strandId": "strand-123",
      "score": 0.92,
      "position": 0
    }
  ]
}
```

### POST /api/v1/rag/query
Ask a question (RAG)

**Request:**
```json
{
  "question": "What is quantum computing?",
  "loomId": "loom-456",
  "searchOptions": {
    "k": 5,
    "minScore": 0.7
  },
  "chatOptions": {
    "model": "gpt-4",
    "temperature": 0.7,
    "maxTokens": 2048
  }
}
```

**Response:**
```json
{
  "chatId": "chat-789",
  "answer": "Quantum computing is a type of computation...",
  "citations": [
    {
      "strandId": "strand-123",
      "chunkId": "chunk-abc",
      "score": 0.92,
      "text": "Quantum computing uses qubits..."
    }
  ],
  "durationMs": 2340,
  "cost": 0.0023
}
```

### GET /api/v1/rag/chats
List chat history

**Query Parameters:**
- `loomId`: Filter by loom (optional)
- `limit`: Max results (default: 20)

**Response:**
```json
[
  {
    "id": "chat-789",
    "question": "What is quantum computing?",
    "answer": "Quantum computing is...",
    "status": "completed",
    "createdAt": "2025-11-14T..."
  }
]
```

---

## Configuration

### Per-Loom Settings (RagConfig)
Each Loom can have custom RAG configuration:

```typescript
{
  embeddingProvider: 'local' | 'openai',
  embeddingModel: 'nomic-embed-text' | 'text-embedding-3-small',
  llmProvider: 'local' | 'openai' | 'anthropic',
  llmModel: 'llama2' | 'gpt-4' | 'claude-3-sonnet',
  llmEndpoint: 'http://localhost:11434',  // for local
  chunkSize: 512,
  chunkOverlap: 50,
  topK: 5,
  minScore: 0.5,
  hybridSearch: true,
  autoEmbed: false
}
```

### Global Defaults (User Preferences)
Stored in `User.preferences.dataIntelligence.rag`:

```typescript
{
  ragEnabled: boolean,
  ragEmbeddingProvider: 'local' | 'openai',
  ragLlmProvider: 'local' | 'openai' | 'anthropic',
  ragLocalEndpoint: string,
  ragAutoEmbed: boolean,
  ragTopK: number,
  ragMinScore: number
}
```

---

## Local vs Cloud Providers

### Local (Ollama)
**Pros:**
- ✅ Free (no API costs)
- ✅ Private (data never leaves your machine)
- ✅ Fast (no network latency)
- ✅ Works offline

**Cons:**
- ⚠️ Requires Ollama installed
- ⚠️ Lower accuracy than GPT-4
- ⚠️ Slower on CPU-only machines

**Setup:**
```bash
# Install Ollama
curl https://ollama.ai/install.sh | sh

# Pull embedding model
ollama pull nomic-embed-text

# Pull LLM
ollama pull llama2
```

### Cloud (OpenAI/Anthropic)
**Pros:**
- ✅ Higher accuracy
- ✅ No local setup required
- ✅ Faster on low-end hardware

**Cons:**
- ⚠️ Costs per query (~$0.01-0.05)
- ⚠️ Requires internet
- ⚠️ Data sent to third party

**Costs:**
- Embedding: $0.00002 per 1K tokens (OpenAI text-embedding-3-small)
- LLM: $0.01 per 1K tokens (GPT-4o-mini) to $0.06 per 1K tokens (GPT-4)

---

## Edition Differences

| Feature | Community | Teams |
|---------|-----------|-------|
| RAG enabled | ✓ | ✓ |
| Local models (Ollama) | ✓ | ✓ |
| Cloud models (OpenAI/Anthropic) | ✗ | ✓ |
| Auto-embed | ✓ | ✓ |
| Max embedded chunks | 2,000 | Unlimited |
| Concurrent queries | 1 | 5 |
| Cost tracking | ✗ | ✓ |
| Team-wide policies | ✗ | ✓ |

---

## Performance

### Embedding
- **Local (Ollama):** ~200ms per chunk
- **OpenAI:** ~100ms per chunk (batched)
- **Batch Size:** 10 chunks at a time

### Search
- **<1k vectors:** <50ms
- **<10k vectors:** <200ms
- **>10k vectors:** <500ms (in-memory cosine)

### RAG Query (End-to-End)
- **Total:** ~2-5 seconds
  - Query embedding: ~100ms
  - Vector search: ~50ms
  - LLM generation: ~2-4s (streaming)

---

## Best Practices

### Chunking
- **Default:** 512 tokens with 50 token overlap
- **For code:** Use smaller chunks (256 tokens)
- **For narrative:** Use larger chunks (1024 tokens)
- **Overlap:** Preserve context across chunk boundaries

### Search
- **Top-K:** Start with 5, increase for complex questions
- **Min Score:** 0.5 is good default, 0.7 for precision
- **Hybrid Search:** Combine vector + keyword for best results

### Prompting
- **Be specific:** "What are the main themes in Chapter 3?" vs "Tell me about themes"
- **Reference context:** "Based on my research notes, explain..."
- **Iterate:** Ask follow-ups to refine answers

---

## UI Integration

### AskBar Component
```tsx
import { AskBar } from '@/components/rag/AskBar';

<AskBar
  loomId="loom-123"
  onSubmit={(question) => handleAsk(question)}
  recentQuestions={['What is...', 'Explain...']}
/>
```

### ChatPanel Component
```tsx
import { ChatPanel } from '@/components/rag/ChatPanel';

<ChatPanel
  loomId="loom-123"
  onClose={() => setShowChat(false)}
/>
```

### Citation Cards
```tsx
import { CitationCard } from '@/components/rag/CitationCard';

<CitationCard
  citation={{
    strandId: 'strand-123',
    chunkId: 'chunk-456',
    score: 0.92,
    text: 'Quantum computing...'
  }}
  index={0}
  onOpenStrand={(id) => router.push(`/strands/${id}`)}
/>
```

---

## Troubleshooting

### "Failed to generate embeddings"
- **Local:** Ensure Ollama is running (`ollama serve`)
- **Local:** Pull embedding model (`ollama pull nomic-embed-text`)
- **Cloud:** Check API key in settings

### "No results found"
- Ensure strands are embedded (`POST /rag/embed`)
- Lower `minScore` threshold
- Check loom ID is correct

### "Slow responses"
- **Local:** Use GPU-accelerated Ollama
- **Cloud:** Switch to faster model (gpt-4o-mini)
- **Both:** Reduce `topK` to retrieve fewer chunks

### "High costs"
- Use local models for embedding (free)
- Use gpt-4o-mini instead of gpt-4
- Enable cost limits in settings

---

## Security & Privacy

### Data Flow
- **Local Mode:** All data stays on your machine
- **Cloud Mode:** Only query + retrieved chunks sent to API
- **Original strands:** Never sent to cloud, only relevant chunks

### Access Control
- RAG queries respect Loom permissions
- Users can only search Looms they have access to
- Citations link to strands user can read

### Cost Protection
- Per-user monthly limits (Community: N/A, Teams: configurable)
- Cost tracking per query
- Warnings when approaching limits

---

## Roadmap

### Phase 1 (Current)
- ✅ Basic RAG with cosine similarity
- ✅ Local + cloud providers
- ✅ UI integration (AskBar, ChatPanel)
- ✅ Settings and configuration

### Phase 2 (Next)
- [ ] Hybrid search (vector + BM25 keyword)
- [ ] Streaming responses (SSE)
- [ ] Re-ranking for better relevance
- [ ] Multi-turn conversations

### Phase 3 (Future)
- [ ] pgvector for Postgres (faster search)
- [ ] HNSW index for >10k vectors
- [ ] Fine-tuned embeddings
- [ ] Graph-aware retrieval (follow strand relationships)

---

## Examples

### Research Assistant
```typescript
// Embed all research papers
for (const strand of researchPapers) {
  await sdk.rag.embedStrand({
    strandId: strand.id,
    loomId: 'research-loom'
  });
}

// Ask questions
const answer = await sdk.rag.query({
  question: 'What are the key findings about neural networks?',
  loomId: 'research-loom'
});

console.log(answer.answer);
answer.citations.forEach((c, i) => {
  console.log(`[${i+1}] ${c.text.slice(0, 100)}...`);
});
```

### Story Consistency Checker
```typescript
// Ask about plot points
const answer = await sdk.rag.query({
  question: 'What color are Alice\'s eyes in Chapter 3?',
  loomId: 'novel-loom',
  searchOptions: {
    strandIds: ['chapter-3-strand']
  }
});
```

### World-Building Reference
```typescript
// Query lore database
const answer = await sdk.rag.query({
  question: 'Describe the magic system in the northern kingdoms',
  loomId: 'worldbuilding-loom'
});
```

---

## SDK Usage

```typescript
import { OpenStrandSDK } from '@framers/openstrand-sdk';

const sdk = new OpenStrandSDK({
  baseUrl: 'http://localhost:8000',
  token: 'your-token'
});

// Embed
await sdk.rag.embedStrand({
  strandId: 'strand-123',
  loomId: 'loom-456'
});

// Search
const results = await sdk.rag.search({
  query: 'quantum computing',
  loomId: 'loom-456',
  k: 5
});

// Query
const answer = await sdk.rag.query({
  question: 'What is quantum computing?',
  loomId: 'loom-456'
});

// List history
const chats = await sdk.rag.listChats({ loomId: 'loom-456' });
```

---

## Admin Monitoring

Admins can monitor RAG usage at `/dashboard/data-intelligence`:

- **Total Queries:** Number of RAG queries across all users
- **Embedded Chunks:** Total vectors stored
- **Avg Response Time:** Average query duration
- **Total RAG Cost:** Cumulative cost for cloud providers

---

## Cost Optimization

### Use Local Models
```
Embedding: Ollama (nomic-embed-text) - FREE
LLM: Ollama (llama2) - FREE
Total: $0.00 per query
```

### Hybrid Approach
```
Embedding: Ollama - FREE
LLM: OpenAI (gpt-4o-mini) - $0.01 per query
Total: ~$0.01 per query
```

### Full Cloud
```
Embedding: OpenAI (text-embedding-3-small) - $0.00002 per query
LLM: OpenAI (gpt-4) - $0.06 per query
Total: ~$0.06 per query
```

**Recommendation:** Use local embeddings (free, fast) + cloud LLM for best balance.

---

## Technical Details

### Vector Storage
- **Format:** Float32 arrays stored as `Bytes` in Prisma
- **Normalization:** L2 normalized for cosine similarity
- **Dimensions:** 768 (Ollama) or 1536 (OpenAI)

### Similarity Metric
- **Cosine Similarity:** `dot(A, B) / (||A|| * ||B||)`
- **Range:** 0 (unrelated) to 1 (identical)
- **Threshold:** 0.5 is good default

### Chunking Strategy
- **Max Tokens:** 512 (configurable)
- **Overlap:** 50 tokens (preserves context)
- **Method:** Sentence-aware (splits on `.!?`)

---

## Limitations

### Current
- In-memory cosine search (not optimized for >10k vectors)
- No hybrid search (vector + keyword)
- No re-ranking
- Single-turn conversations only

### Future Improvements
- pgvector extension for Postgres
- HNSW index for approximate nearest neighbors
- BM25 keyword search integration
- Multi-turn conversation context

---

## References

- [RAG Paper (Lewis et al., 2020)](https://arxiv.org/abs/2005.11401)
- [Ollama Documentation](https://ollama.ai/docs)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [Cosine Similarity Explained](https://en.wikipedia.org/wiki/Cosine_similarity)

---

_For implementation details, see `packages/openstrand-teams-backend/src/services/rag/`_

