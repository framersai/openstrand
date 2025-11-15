# Journal Reflection Flow

**Status:** Implemented November 2025  
**Components:** Backend services, PKMS UI integration

---

## Overview

The Journal Reflection flow allows users to synthesize their daily notes, mood data, and quick captures into cohesive reflection strands. This creates a bridge between ephemeral journaling and permanent knowledge artifacts.

---

## Backend

### JournalReflectionService

**Location:** `packages/openstrand-teams-backend/src/services/journal/journal-reflection.service.ts`

**Features:**
- Aggregates daily notes, moods, and quick captures over a time window
- Uses AI to synthesize a cohesive reflection
- Creates a new strand linked to the user's journal weave
- Preserves metadata for traceability and future analytics

**Methods:**
- `createReflection(userId, options)` - Create a reflection strand from journal data

**Options:**
- `startDate` / `endDate`: Time window for aggregation
- `includeMood`: Whether to include mood data in reflection (default: true)
- `includeQuickCaptures`: Whether to include quick captures (default: true)
- `title`: Optional custom title (defaults to date range)
- `tags`: Optional tags to add to the reflection strand
- `loomId`: Optional loom/scope to link the reflection to
- `style`: 'summary' | 'narrative' | 'bullet_points'

**Mood Analysis:**
- Calculates mood distribution across the time window
- Identifies most common mood
- Detects trend: 'improving' | 'stable' | 'declining' | 'unknown'
- Compares first half vs second half of period

### API Routes

**Location:** `packages/openstrand-teams-backend/src/routes/journal.routes.ts`

**Endpoint:**
- `POST /api/v1/journal/reflections` - Create reflection strand

**Request:**
```json
{
  "startDate": "2025-01-01T00:00:00Z",
  "endDate": "2025-01-07T23:59:59Z",
  "includeMood": true,
  "includeQuickCaptures": true,
  "style": "narrative",
  "tags": ["weekly-review", "2025-w1"]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "strand": {
      "id": "strand-xyz",
      "title": "Weekly Reflection: 2025-01-01",
      "summary": "...",
      "content": {...}
    },
    "metadata": {
      "dailyNotesCount": 7,
      "quickCapturesCount": 12,
      "moodEntries": 6,
      "dateRange": {
        "start": "2025-01-01T00:00:00Z",
        "end": "2025-01-07T23:59:59Z"
      },
      "generatedAt": "2025-01-08T10:30:00Z"
    }
  },
  "message": "Reflection strand created"
}
```

---

## Frontend

### JournalReflectionCard

**Location:** `openstrand-app/src/components/journal/JournalReflectionCard.tsx`

**Features:**
- Date range presets: Last 7 days, Last 30 days, Custom
- Style selector: Summary, Narrative, Bullet points
- Generate button with loading state
- Success feedback with "View Reflection" link
- Toast notifications for success/failure
- Fully responsive design

**Integration:**
- Rendered in `/daily` page sidebar above recommendations
- Automatically creates reflection strand on generation
- Links to created strand for immediate viewing

---

## User Flow

1. User opens daily journal page (`/daily`)
2. Right sidebar shows "Create Reflection" card
3. User selects:
   - Time window (7 days, 30 days, or custom)
   - Reflection style (summary, narrative, bullet points)
4. User clicks "Generate Reflection"
5. Backend:
   - Queries daily notes, moods, quick captures in date range
   - Analyzes mood trends and patterns
   - Uses AI to synthesize cohesive reflection
   - Creates new strand with reflection content
6. Frontend shows success toast with metadata
7. User clicks "View Reflection" to see the created strand
8. Reflection strand appears in PKMS dashboard and analytics

---

## Reflection Styles

### Summary (Default)
- Balanced overview of the period
- Highlights key themes and insights
- 1-2 paragraphs
- Best for: Quick reviews, regular check-ins

### Narrative
- Cohesive story connecting themes
- First person perspective ("I noticed...", "I learned...")
- 2-3 paragraphs
- Best for: Deep reflection, personal growth tracking

### Bullet Points
- Organized by clear categories
- Highlights key themes, learnings, patterns
- Concise and actionable
- Best for: Action planning, structured reviews

---

## Metadata Tracking

Each reflection strand stores:
- `reflectionType`: 'journal'
- `dateRange`: { start, end }
- `sources`: { dailyNotes: [...], quickCaptures: [...] }
- `moodSummary`: { distribution, mostCommon, trend }
- `generatedAt`: timestamp

This enables:
- Traceability back to source journal entries
- Analytics on reflection patterns over time
- Future enhancements (e.g., "Compare this month to last month")

---

## Mobile Responsiveness

- Card layout adapts to small screens
- Select dropdowns are full-width on mobile
- Generate button is prominent and easy to tap
- Toast notifications are readable on all screen sizes
- "View Reflection" link is clearly visible after creation

---

## Testing

**Unit tests:** `packages/openstrand-teams-backend/tests/unit/journal-reflection.service.test.ts`

Tests cover:
- Default title generation (single day, weekly, monthly, custom)
- Slug generation with special character handling
- Mood aggregation (distribution, most common, trend detection)
- Reflection prompt building for different styles
- Edge cases (empty mood data, no quick captures)

---

## Future Enhancements

- **Scheduled reflections**: Automatic weekly/monthly reflection generation
- **Comparison view**: Compare current reflection to previous periods
- **Mood insights**: Deeper mood pattern analysis with recommendations
- **Quick capture processing**: AI-powered categorization before reflection
- **Collaborative reflections**: Team reflection strands for shared projects
- **Export options**: Export reflections as PDF, markdown, or email

---

For architectural context, see `docs/openstrand/ARCHITECTURE.md` ("Personal Knowledge Workflows"). For implementation details, see TSDoc in the service and route files.

