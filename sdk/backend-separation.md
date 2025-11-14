# SDK vs Backend: Proper Separation of Concerns

## Core Principle

The SDK is a client library - it doesn't implement business logic, it calls the backend API.

## What the Backend Does (Teams Edition)

1) Business Logic Implementation (AI Artisan, billing, schema intelligence, OCR, STT, scraping, moderation)
2) Database Operations (Prisma, queries, transactions)
3) Third-Party Integrations (OpenAI/Anthropic, Stripe/LemonSqueezy, S3/Supabase, Redis, BullMQ)
4) Resource Management (rate limiting, cost limits, quotas, usage tracking)

## What the SDK Should Do

### HTTP Client Wrapper
```typescript
class VisualizationService {
  async create(data: any[], prompt: string): Promise<Visualization> {
    return this.client.post('/visualizations', { data, prompt });
  }
}
```

### Type Safety
```typescript
interface Visualization {
  id: string;
  type: VisualizationType;
  tier: 1 | 2 | 3;
  config: ChartConfig | D3Config | CustomConfig;
  created: Date;
}
```

### Developer Experience
```typescript
const viz = await sdk.visualizations.create(data, "Show sales trends");
```

### Client-Side Features
- Interceptors, retry, token refresh, offline queue, caching, progress tracking

## Corrected SDK Services
- Authentication Client
- Strands Client
- AI Client
- Visualizations Client

## What SDK Should NOT Do
No DB access, no direct AI provider calls, no business logic, no heavy compute, no secrets management.

## Architecture Benefits
- Security, Flexibility, Performance, Business Model alignment.

## Example Flow
User -> SDK -> Backend API -> AI Service -> Provider -> Result

## Implementation Priority
Phase 1: Core clients; Phase 2: Advanced clients; Phase 3: Enhanced features.

## Editions
Community vs Teams vs Enterprise alignment through backend APIs; SDK remains the same.

## Key Takeaway
The SDK is a typed HTTP client. All business logic and integrations happen on the backend.


