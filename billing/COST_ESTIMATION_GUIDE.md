# Cost Estimation & Dry-Run Guide

**Last Updated:** November 14, 2025  
**Status:** Production Ready  
**Editions:** Community (local only) | Teams (local + cloud with tracking)

---

## Overview

OpenStrand provides **transparent cost prediction** for all AI operations before execution. See exactly what you'll pay before clicking "Generate" or "Ask."

**Key Features:**
- 💰 Dry-run estimates for all AI operations
- 📊 Real-time cost display in UI
- 🎯 Low/estimate/high ranges for accuracy
- 🏠 Local models: $0 (always)
- ☁️ Cloud models: Transparent pricing
- 📈 Monthly budget tracking (Teams)

---

## How It Works

### 1. Token Estimation
```
Text → Token Count (chars/4 approximation)
     → Model Pricing Lookup
     → Cost Calculation
```

### 2. Cost Calculation
```
Input Cost = (inputTokens / 1000) × inputRate
Output Cost = (outputTokens / 1000) × outputRate
Total = Input + Output + Embedding (if applicable)
```

### 3. Ranges
```
Low:      0.7 × typical completion
Estimate: Typical completion ratio
High:     1.3 × typical (capped at maxTokens)
```

---

## API Reference

### POST /api/v1/cost/estimate/text
Estimate text generation cost

**Request:**
```json
{
  "prompt": "Explain quantum computing",
  "model": "gpt-4",
  "maxTokens": 500,
  "systemMessage": "You are a helpful assistant"
}
```

**Response:**
```json
{
  "inputTokens": 125,
  "outputTokens": {
    "low": 52,
    "estimate": 75,
    "high": 97
  },
  "cost": {
    "input": 0.00375,
    "output": {
      "low": 0.00312,
      "estimate": 0.0045,
      "high": 0.00582
    },
    "total": {
      "low": 0.00687,
      "estimate": 0.00825,
      "high": 0.00957
    }
  },
  "model": "gpt-4",
  "provider": "openai",
  "notes": [
    "Estimate based on typical completion ratio",
    "Max tokens: 500"
  ]
}
```

### POST /api/v1/cost/estimate/embedding
Estimate embedding cost

**Request:**
```json
{
  "texts": ["Hello world", "Another text", "More content"],
  "model": "text-embedding-3-small"
}
```

**Response:**
```json
{
  "embeddingTokens": 15,
  "cost": {
    "embedding": 0.0000003,
    "total": { "estimate": 0.0000003 }
  },
  "model": "text-embedding-3-small",
  "provider": "openai",
  "notes": ["3 texts", "15 total tokens"]
}
```

### POST /api/v1/cost/estimate/rag-query
Estimate RAG query cost

**Request:**
```json
{
  "question": "What is quantum computing?",
  "retrievedChunks": 5,
  "avgChunkTokens": 400,
  "llmModel": "gpt-4o-mini",
  "embeddingModel": "text-embedding-3-small",
  "maxTokens": 2048
}
```

**Response:**
```json
{
  "inputTokens": 2012,
  "outputTokens": { "low": 700, "estimate": 1006, "high": 1307 },
  "embeddingTokens": 12,
  "cost": {
    "input": 0.0003018,
    "output": { "low": 0.00042, "estimate": 0.0006036, "high": 0.0007842 },
    "embedding": 0.00000024,
    "total": { "low": 0.00072204, "estimate": 0.00090564, "high": 0.00108444 }
  },
  "model": "gpt-4o-mini",
  "provider": "openai",
  "notes": [
    "Query embedding: 12 tokens",
    "Context: 5 chunks × 400 tokens",
    "Total prompt: 2012 tokens"
  ]
}
```

### POST /api/v1/cost/estimate/strand-embedding
Estimate strand embedding cost

**Request:**
```json
{
  "textLength": 5000,
  "chunkSize": 512,
  "chunkOverlap": 50,
  "model": "text-embedding-3-small"
}
```

**Response:**
```json
{
  "chunks": 12,
  "embeddingTokens": 6144,
  "cost": {
    "embedding": 0.00012288,
    "total": { "estimate": 0.00012288 }
  },
  "model": "text-embedding-3-small",
  "provider": "openai",
  "notes": [
    "12 chunks",
    "6144 total tokens",
    "Chunk size: 512, overlap: 50"
  ]
}
```

### GET /api/v1/cost/pricing
Get model pricing information

**Query Parameters:**
- `provider`: Filter by provider (optional)

**Response:**
```json
[
  {
    "model": "gpt-4",
    "provider": "openai",
    "inputCostPer1k": 0.03,
    "outputCostPer1k": 0.06,
    "typicalCompletionRatio": 0.6
  },
  {
    "model": "gpt-4o-mini",
    "provider": "openai",
    "inputCostPer1k": 0.00015,
    "outputCostPer1k": 0.0006,
    "typicalCompletionRatio": 0.5
  },
  {
    "model": "text-embedding-3-small",
    "provider": "openai",
    "embeddingCostPer1k": 0.00002
  },
  {
    "model": "llama2",
    "provider": "ollama",
    "inputCostPer1k": 0,
    "outputCostPer1k": 0
  }
]
```

---

## Dry-Run Support

All AI operations support `dryRun` parameter:

### RAG Embed (Dry Run)
```bash
curl -X POST http://localhost:8000/api/v1/rag/embed \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"strandId": "...", "loomId": "...", "dryRun": true}'
```

### RAG Query (Dry Run)
```bash
curl -X POST http://localhost:8000/api/v1/rag/query \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"question": "...", "loomId": "...", "dryRun": true}'
```

---

## SDK Usage

```typescript
import { OpenStrandSDK } from '@framers/openstrand-sdk';

const sdk = new OpenStrandSDK({ baseUrl: 'http://localhost:8000' });

// Estimate text generation
const textEst = await sdk.cost.estimateText({
  prompt: 'Explain quantum computing',
  model: 'gpt-4',
  maxTokens: 500
});

console.log(`Est. cost: $${textEst.cost.total.estimate.toFixed(4)}`);

// Estimate embedding
const embeddingEst = await sdk.cost.estimateEmbedding({
  texts: ['Hello', 'World'],
  model: 'text-embedding-3-small'
});

// Estimate RAG query
const ragEst = await sdk.cost.estimateRagQuery({
  question: 'What is quantum computing?',
  retrievedChunks: 5,
  avgChunkTokens: 400,
  llmModel: 'gpt-4o-mini',
  embeddingModel: 'text-embedding-3-small'
});

// Estimate strand embedding
const strandEst = await sdk.cost.estimateStrandEmbedding({
  textLength: 5000,
  model: 'text-embedding-3-small'
});

console.log(`${strandEst.chunks} chunks, $${strandEst.cost.total.estimate.toFixed(4)}`);

// Get pricing info
const pricing = await sdk.cost.getPricing('openai');
pricing.forEach(p => {
  console.log(`${p.model}: $${p.inputCostPer1k}/1k input`);
});
```

---

## UI Integration

### AskBar (Real-Time Estimates)
The AskBar shows live cost estimates as you type:

```
┌─────────────────────────────────────────┐
│ 🔍 What is quantum computing?           │
│                           [$0.018] [Ask]│
└─────────────────────────────────────────┘
```

- Updates 500ms after typing stops (debounced)
- Shows `<$0.01` for very cheap queries
- Only visible for cloud providers (local = $0)

### ChatPanel (Actual Costs)
After receiving an answer, shows actual cost:

```
Assistant: Quantum computing is...
[1] strand-123 (score: 0.92)
[2] strand-456 (score: 0.87)

💵 Cost: $0.0182
```

### Upload Wizard
When enabling auto-embed:

```
☑ Auto-embed for semantic search
   Est. 12 chunks, $0.00024 (OpenAI)
   or $0.00 (Local/Ollama)
```

### Settings Page
Shows provider rates and example costs:

```
RAG Settings
  Embedding Provider: OpenAI
    Cost: ~$0.00002 per 1K tokens
    
  LLM Provider: GPT-4o-mini
    Cost: ~$0.00015 input, ~$0.0006 output per 1K tokens
    
  Example: Typical query costs ~$0.01-0.02
```

---

## Model Pricing

### OpenAI
| Model | Input (per 1K) | Output (per 1K) | Use Case |
|-------|----------------|-----------------|----------|
| gpt-4 | $0.03 | $0.06 | Highest quality |
| gpt-4-turbo | $0.01 | $0.03 | Fast + quality |
| gpt-4o | $0.005 | $0.015 | Balanced |
| gpt-4o-mini | $0.00015 | $0.0006 | **Recommended** |
| gpt-3.5-turbo | $0.0005 | $0.0015 | Budget |
| text-embedding-3-small | - | - | $0.00002 (embedding) |
| text-embedding-3-large | - | - | $0.00013 (embedding) |

### Anthropic
| Model | Input (per 1K) | Output (per 1K) | Use Case |
|-------|----------------|-----------------|----------|
| claude-3-opus | $0.015 | $0.075 | Highest quality |
| claude-3-sonnet | $0.003 | $0.015 | **Recommended** |
| claude-3-haiku | $0.00025 | $0.00125 | Fast + cheap |

### Ollama (Local)
| Model | Cost | Use Case |
|-------|------|----------|
| llama2 | **$0** | General chat |
| mistral | **$0** | Fast responses |
| nomic-embed-text | **$0** | **Recommended** embeddings |
| all-minilm | **$0** | Lightweight embeddings |

---

## Cost Optimization

### Use Local Models (Free)
```typescript
// Settings
{
  ragEmbeddingProvider: 'local',
  ragLlmProvider: 'local',
  ragLocalEndpoint: 'http://localhost:11434'
}

// Result: $0.00 per query
```

### Hybrid Approach (Recommended)
```typescript
// Settings
{
  ragEmbeddingProvider: 'local',      // Free
  ragLlmProvider: 'openai',           // Paid
  chatOptions: { model: 'gpt-4o-mini' }
}

// Result: ~$0.01 per query
```

### Reduce Context
```typescript
// Settings
{
  ragTopK: 3,          // Fewer chunks (default: 5)
  ragMinScore: 0.7     // Higher threshold (default: 0.5)
}

// Result: Smaller prompts = lower costs
```

### Set Budgets (Teams)
```typescript
// Team settings
{
  monthlyBudget: 100.00,
  hardCap: true,
  alertThreshold: 0.8
}

// Result: Automatic enforcement + alerts
```

---

## UI Examples

### AskBar with Cost
```tsx
import { AskBar } from '@/components/rag/AskBar';

<AskBar
  loomId="loom-123"
  onSubmit={(question) => handleAsk(question)}
  // Cost estimate shown automatically as user types
/>
```

**What Users See:**
- Type question
- Wait 500ms
- See `[$0.018]` badge appear
- Click "Ask" with confidence

### ChatPanel with Actual Cost
```tsx
import { ChatPanel } from '@/components/rag/ChatPanel';

<ChatPanel
  loomId="loom-123"
  onClose={() => setShowChat(false)}
  // Shows actual cost after each response
/>
```

**What Users See:**
- Ask question
- Get answer with citations
- See `💵 Cost: $0.0182` at bottom

---

## Edition Differences

| Feature | Community | Teams |
|---------|-----------|-------|
| Cost estimates | ✓ (local only) | ✓ (all providers) |
| Real-time estimates | ✓ | ✓ |
| Actual cost tracking | ✗ | ✓ |
| Monthly budgets | ✗ | ✓ |
| Budget alerts | ✗ | ✓ |
| Hard caps | ✗ | ✓ |
| Cost reports | ✗ | ✓ |
| Per-user limits | ✗ | ✓ |

---

## Typical Costs

### RAG Query
```
Local (Ollama):
  Embedding: $0.00
  LLM: $0.00
  Total: $0.00 ✅

Hybrid (Ollama + GPT-4o-mini):
  Embedding: $0.00
  LLM: ~$0.01
  Total: ~$0.01 ✅

Cloud (OpenAI):
  Embedding: ~$0.0001
  LLM (GPT-4o-mini): ~$0.01
  Total: ~$0.01 ✅

Cloud (OpenAI GPT-4):
  Embedding: ~$0.0001
  LLM: ~$0.06
  Total: ~$0.06 ⚠️
```

### Strand Embedding
```
Local (Ollama):
  12 chunks: $0.00 ✅

OpenAI (text-embedding-3-small):
  12 chunks: ~$0.00012 ✅

OpenAI (text-embedding-3-large):
  12 chunks: ~$0.00078 ⚠️
```

### Vocabulary Verification (Optional)
```
Deterministic (TF/IDF, NER):
  Cost: $0.00 ✅

LLM Verification (GPT-4o-mini):
  100 terms: ~$0.005 ✅

LLM Verification (GPT-4):
  100 terms: ~$0.03 ⚠️
```

---

## Best Practices

### 1. Start Local, Upgrade Selectively
- Use Ollama for everything initially ($0)
- Switch to cloud LLM for critical queries
- Keep embeddings local (minimal quality difference)

### 2. Monitor and Optimize
- Check monthly costs in admin panel
- Identify expensive queries
- Optimize prompts and context size

### 3. Set Budgets Early
- Teams: Set monthly budget per user/team
- Enable alerts at 80% threshold
- Use hard caps to prevent overruns

### 4. Educate Users
- Show cost estimates prominently
- Explain local vs cloud trade-offs
- Provide cost-saving tips

---

## Implementation Details

### Token Estimation
```typescript
// Rough approximation (1 token ≈ 4 characters)
function estimateTokens(text: string): number {
  return Math.ceil(text.length / 4);
}

// For production, consider:
// - tiktoken for OpenAI models
// - Anthropic tokenizer for Claude
// - Model-specific tokenizers
```

### Completion Ratio
```typescript
// Typical output/input ratios by model
const COMPLETION_RATIOS = {
  'gpt-4': 0.6,           // Verbose
  'gpt-4o-mini': 0.5,     // Concise
  'claude-3-sonnet': 0.7, // Very verbose
};

// Estimate output tokens
const outputEst = inputTokens * ratio;
const outputLow = outputEst * 0.7;
const outputHigh = Math.min(outputEst * 1.3, maxTokens);
```

### Cost Calculation
```typescript
const inputCost = (inputTokens / 1000) * inputRate;
const outputCost = (outputTokens / 1000) * outputRate;
const embeddingCost = (embeddingTokens / 1000) * embeddingRate;
const total = inputCost + outputCost + embeddingCost;
```

---

## Testing

### Unit Tests
```typescript
describe('CostEstimationService', () => {
  it('estimates GPT-4 text generation cost', () => {
    const estimate = service.estimateText({
      prompt: 'Explain quantum computing',
      model: 'gpt-4',
      maxTokens: 500
    });
    
    expect(estimate.cost.total.estimate).toBeGreaterThan(0);
  });
  
  it('returns zero cost for local models', () => {
    const estimate = service.estimateEmbedding({
      texts: ['Test'],
      model: 'nomic-embed-text'
    });
    
    expect(estimate.cost.total.estimate).toBe(0);
  });
});
```

---

## Troubleshooting

### "Unknown model" error
- Check model name spelling
- Verify model is in pricing registry
- Add custom model pricing if needed

### Estimates seem off
- Token estimation is approximate (chars/4)
- Completion ratio varies by prompt
- Use actual costs to calibrate

### Local models showing cost
- Verify provider is 'ollama'
- Check pricing registry has 0 costs
- Ensure local endpoint is configured

---

## Future Enhancements

### Phase 2
- [ ] Integrate tiktoken for accurate OpenAI token counts
- [ ] Per-user cost history and trends
- [ ] Budget alerts via email/webhook
- [ ] Cost optimization suggestions

### Phase 3
- [ ] Fine-tuned completion ratios per user
- [ ] A/B test cost vs quality trade-offs
- [ ] Automatic provider switching based on budget
- [ ] Cost-aware caching strategies

---

## References

- [OpenAI Pricing](https://openai.com/pricing)
- [Anthropic Pricing](https://www.anthropic.com/pricing)
- [Ollama Models](https://ollama.ai/library)
- [tiktoken (OpenAI tokenizer)](https://github.com/openai/tiktoken)

---

_For implementation details, see `packages/openstrand-teams-backend/src/services/cost/`_

