# Analytics → Study Generation Flow

**Status:** Implemented November 2025  
**Components:** Backend services, SDK modules, PKMS UI integration

---

## Overview

The Analytics → Study flow bridges the analytics and learning layers by allowing users to generate flashcards and quizzes directly from strand/loom analytics data. This creates a seamless path from "understanding what's in my knowledge base" to "actively studying it."

---

## Backend

### AnalyticsStudyService

**Location:** `packages/openstrand-teams-backend/src/services/learning/analytics-study.service.ts`

**Features:**
- Reads strand/loom analytics summaries (entities, keywords, topics)
- Identifies high-signal content for study material generation
- Uses AI to generate flashcards and quizzes
- Creates structured study materials linked to source strands

**Methods:**
- `generateFromStrand(strandId, userId, options)` - Generate study materials from a single strand's analytics
- `generateFromLoom(loomId, userId, options)` - Generate study materials from a loom's aggregated analytics

**Options:**
- `type`: 'flashcards' | 'quiz' | 'both'
- `count`: Number of items to generate (1-50 for strands, 1-100 for looms)
- `difficulty`: 'beginner' | 'intermediate' | 'advanced' | 'expert'
- `focusAreas`: ['entities', 'keywords', 'concepts', 'relationships', 'topics']
- `deck`: Optional deck name for flashcards

### API Routes

**Location:** `packages/openstrand-teams-backend/src/routes/learning/analytics-study.routes.ts`

**Endpoints:**
- `POST /api/v1/analytics/study/strand` - Generate from strand analytics
- `POST /api/v1/analytics/study/loom` - Generate from loom analytics

Both endpoints require authentication and return:
```json
{
  "success": true,
  "data": {
    "flashcards": [...],
    "quiz": {...},
    "metadata": {
      "sourceType": "strand" | "loom",
      "sourceId": "...",
      "focusAreas": [...],
      "entitiesUsed": 5,
      "keywordsUsed": 10,
      "generatedAt": "2025-11-15T..."
    }
  },
  "message": "Generated 10 flashcards and 1 quiz"
}
```

---

## SDK

### LearningModule

**Location:** `packages/openstrand-sdk/src/modules/learning.module.ts`

**Usage:**
```typescript
import { OpenStrandSDK } from '@framers/openstrand-sdk';

const sdk = new OpenStrandSDK({ baseUrl: 'http://localhost:8000' });

// Generate flashcards from strand analytics
const result = await sdk.learning.generateFromStrand({
  strandId: 'strand-123',
  type: 'flashcards',
  count: 10,
  difficulty: 'intermediate',
  focusAreas: ['entities', 'keywords']
});

console.log(`Created ${result.flashcards.length} flashcards`);

// Generate from loom analytics
const loomResult = await sdk.learning.generateFromLoom({
  loomId: 'loom-456',
  type: 'both',
  count: 20,
  difficulty: 'advanced',
  focusAreas: ['topics', 'entities', 'relationships']
});
```

---

## Frontend

### StrandAnalyticsPanel

**Location:** `openstrand-app/src/components/pkms/analytics/StrandAnalyticsPanel.tsx`

**Features:**
- "Study this" button in panel header
- Generates 10 flashcards from strand's entities and keywords
- Shows toast notification with count of generated materials
- Button is disabled when no analytics data is available
- Responsive: hides button text on small screens, shows only icon

### LoomAnalyticsPanel

**Location:** `openstrand-app/src/components/pkms/analytics/LoomAnalyticsPanel.tsx`

**Features:**
- "Study this" button in panel header
- Generates 20 flashcards + 1 quiz from loom's topics, entities, and keywords
- Shows toast notification with counts
- Button is disabled when no analytics data is available
- Responsive: hides button text on small screens

---

## User Flow

1. User views analytics for a strand or loom in PKMS dashboard
2. Analytics panel shows entity histogram, keywords, topics, etc.
3. User clicks "Study this" button
4. Backend:
   - Reads analytics summary
   - Extracts high-signal entities/keywords/topics
   - Generates flashcards/quiz using AI
   - Creates records in database
5. Frontend shows success toast with count
6. User can navigate to flashcards/quizzes to start studying

---

## Mobile Responsiveness

All analytics panels are now fully responsive:
- Headers stack vertically on small screens
- Metric grids use `grid-cols-1 sm:grid-cols-3`
- Charts explicitly set `width="100%"` for proper scaling
- "Study this" button hides text label on mobile, shows only icon
- All interactive elements have minimum 44×44px tap targets

---

## Testing

**Unit tests:** `packages/openstrand-teams-backend/tests/unit/analytics-study.service.test.ts`

Tests cover:
- Prompt building with entities, keywords, topics
- Flashcard response parsing (valid JSON, malformed, embedded in text)
- Study content extraction with focus areas and limits
- Quiz response parsing

---

## Future Enhancements

- **Study session UI**: In-app flashcard/quiz player integrated directly in PKMS
- **Adaptive generation**: Use spaced repetition data to focus on weak areas
- **Multi-strand generation**: Generate study materials from multiple selected strands
- **Export options**: Export flashcards to Anki, Quizlet, etc.
- **Progress tracking**: Link study sessions back to analytics for completion metrics

---

For architectural context, see `docs/openstrand/ARCHITECTURE.md` ("Analytics & Visualization Layer"). For implementation details, see TSDoc in the service and route files.

