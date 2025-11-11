# OpenStrand API Reference

**Version**: 1.0.0  
**Base URL**: `https://api.openstrand.com` (or `http://localhost:8000` for local)  
**Authentication**: Bearer token in `Authorization` header

## Quick Start with SDK

```typescript
import { OpenStrandClient } from '@framers/openstrand-sdk';

const client = new OpenStrandClient({
  apiUrl: 'https://api.openstrand.com',
  token: 'your-api-token'
});

// Create a strand
const strand = await client.strands.create({
  title: 'My Knowledge',
  content: { markdown: '# Hello World' }
});
```

## Core Endpoints

### Authentication

**POST /auth/login**
```json
{
  "username": "user@example.com",
  "password": "SecurePassword123"
}
```

**POST /auth/register**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "displayName": "John Doe"
}
```

### Strands (Knowledge Units)

**POST /strands** - Create strand
```json
{
  "type": "document",
  "title": "My Research",
  "content": {
    "markdown": "# Research Notes..."
  },
  "tags": ["research", "ai"]
}
```

**GET /strands** - List strands
- Query params: `page`, `pageSize`, `tags`, `search`

**GET /strands/:id** - Get strand by ID

**PUT /strands/:id** - Update strand

**DELETE /strands/:id** - Delete strand

### Import/Export

**POST /import** - Import file
- Multipart form with `file` field
- Supports 20+ formats (PDF, DOCX, MD, etc.)

**POST /export/:id** - Export strand
```json
{
  "format": "pdf" | "docx" | "markdown",
  "includeMetadata": true
}
```

### AI Services

**POST /ai/text** - Generate text
```json
{
  "prompt": "Explain quantum computing",
  "provider": "openai",
  "model": "gpt-4",
  "maxTokens": 2000
}
```

**POST /ai/embedding** - Generate embeddings
```json
{
  "text": "Text to embed",
  "provider": "openai"
}
```

### Search

**GET /search** - Full-text search
- Query params: `q` (query), `type`, `tags`

**POST /search/semantic** - Semantic search
```json
{
  "query": "quantum computing applications",
  "limit": 10
}
```

### Knowledge Graph

**POST /weave** - Create knowledge graph
```json
{
  "name": "My Research Network",
  "strandIds": ["id1", "id2", "id3"]
}
```

**POST /weave/:id/edges** - Add relationship
```json
{
  "sourceId": "strand1",
  "targetId": "strand2", 
  "type": "references"
}
```

## Response Format

All responses follow this structure:

```json
{
  "success": true,
  "data": {
    // Response data here
  }
}
```

Error responses:
```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

## Rate Limits

- Free: 100 requests/hour
- Pro: 5000 requests/hour  
- Team: 10000 requests/hour

Headers indicate remaining quota:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 995
X-RateLimit-Reset: 1635724800
```

## Full Documentation

For complete API documentation with all endpoints, see:
- [Interactive API Docs](http://localhost:8000/docs) (when running locally)
- [Full API Reference](https://github.com/framersai/openstrand-monorepo/blob/main/docs/API_REFERENCE.md)

## SDK Documentation

The TypeScript SDK provides type-safe access to all endpoints:

```bash
npm install @framers/openstrand-sdk
```

See [SDK Guide](https://www.npmjs.com/package/@framers/sdk) for detailed usage.
