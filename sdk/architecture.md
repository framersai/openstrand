# OpenStrand SDK Architecture

Last Updated: November 13, 2025

Purpose: Clarify what the SDK is, what it does, and how it relates to backends

---

## What is the SDK?

The OpenStrand SDK (`@framers/openstrand-sdk`) is a thin TypeScript client library that:

1. Wraps REST API calls to any OpenStrand backend (Teams or Community)
2. Shares types between frontend and backend (TypeScript interfaces)
3. Provides validation schemas (Zod) for request/response
4. Offers helper methods like `sdk.templates.list()`, `sdk.factCheck.start()`

### What SDK is NOT:
- NOT a place for business logic
- NOT a place for LLM provider implementations (OpenAI, Anthropic, etc.)
- NOT a backend itself

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend (openstrand-app)                │
│                                                             │
│  import { OpenStrandSDK } from '@framers/openstrand-sdk';   │
│                                                             │
│  const sdk = new OpenStrandSDK({                            │
│    baseUrl: 'http://localhost:8000',                        │
│    token: authToken                                         │
│  });                                                        │
│                                                             │
│  const templates = await sdk.templates.list();              │
│  const job = await sdk.factCheck.start('Is sky blue?');     │
└───────────────────────────┬─────────────────────────────────┘
                            │
                            │ HTTP REST calls
                            │ (fetch)
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Backend (openstrand-teams-backend)             │
│                                                             │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                  Fastify Routes                         │ │
│  │  /api/templates, /api/fact-check, /api/enrichment      │ │
│  └──────────────────────┬─────────────────────────────────┘ │
│                         │                                   │
│                         ▼                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │                    Services                             │ │
│  │  TemplateService, FactCheckService, LLMService          │ │
│  └──────────────────────┬─────────────────────────────────┘ │
│                         │                                   │
│                         ▼                                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │              LLM Provider Implementations               │ │
│  │  OpenAIProvider, AnthropicProvider, GroqProvider        │ │
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

---

## SDK Structure

```
packages/openstrand-sdk/
├── src/
│   ├── client.ts                     ← Main SDK class
│   ├── modules/
│   │   ├── templates.module.ts       ← sdk.templates.*
│   │   ├── factCheck.module.ts       ← sdk.factCheck.*
│   │   ├── enrichment.module.ts      ← sdk.enrichment.*
│   │   ├── plugins.module.ts         ← sdk.plugins.*
│   │   └── wizard.module.ts          ← sdk.wizard.*
│   ├── types/
│   │   ├── template.types.ts         ← Shared types
│   │   ├── factCheck.types.ts
│   │   └── llm.types.ts              ← LLM types (NO implementations)
│   └── schemas/
│       └── validation.schemas.ts     ← Zod schemas
└── package.json
```

### Key Files

#### `client.ts` – Main SDK class
```typescript
export class OpenStrandSDK {
  private baseUrl: string;
  private token?: string;

  public templates: TemplatesModule;
  public factCheck: FactCheckModule;
  public enrichment: EnrichmentModule;
  // ...

  async request<T>(method: string, path: string, options?: RequestOptions): Promise<T> {
    // Makes fetch() call to backend
  }
}
```

#### `modules/factCheck.module.ts` – Fact-check API wrapper
```typescript
export class FactCheckModule {
  constructor(private sdk: OpenStrandSDK) {}

  async start(options: FactCheckStartOptions): Promise<FactCheckResult> {
    // Calls POST /api/fact-check
    return this.sdk.request('POST', '/api/fact-check', { body: options });
  }

  async getStatus(jobId: string): Promise<FactCheckResult> {
    // Calls GET /api/fact-check/:id
    return this.sdk.request('GET', `/api/fact-check/${jobId}`);
  }
}
```

---

## Data Flow Example: Fact-Check

1) Frontend calls SDK; 2) Backend receives request; 3) Backend service uses LLMService; 4) Providers called.

---

## Pros & Cons

Pros: Separation of concerns; works with any backend; type safety; easy testing; cost control.

Cons: Extra network hop; backend dependency; no offline LLM.

---

## Community vs Teams

Both editions use the same SDK; only the backend differs.

---

## Usage Examples

Initialize SDK, Templates, Fact-Check, Enrichment usage examples.

---

## Backend Implementation

LLMService, providers, and routes live in backend.

---

## Summary

The SDK never calls LLM providers directly; it always goes through the backend to keep keys secure, cost tracking centralized, and rate limiting enforceable.


