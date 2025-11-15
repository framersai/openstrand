# OpenStrand Schema Analysis & PKMS Feature Gaps

**Date**: November 14, 2025  
**Status**: Comprehensive Schema Review & Extension Planning

---

## Executive Summary

After reviewing the existing Prisma schema and comparing it against the ARCHITECTURE.md specification, I can confirm that:

✅ **The schemas ARE properly designed and extensible for gamification**  
✅ **Core PKMS features are well-implemented**  
⚠️ **Several critical PKMS features are missing and should be added**

This document provides:
1. Schema validation against architecture spec
2. Gamification extension pathways
3. Missing PKMS features with implementation roadmap
4. Recommended schema additions

---

## Schema Validation

### ✅ What We Have (Well Implemented)

#### 1. **Core Content Model (Strand)**
```prisma
model Strand {
  // ✅ Polymorphic content support
  strandType String @default("document")
  classification String @default("core")
  noteType String? // PKMS-specific
  
  // ✅ Flexible metadata
  content Json
  metadata Json
  learningMetadata Json?
  spiralMetadata Json?
  
  // ✅ Educational features
  difficulty String?
  prerequisites Json
  scaffoldVariants Json
  estimatedTime Int
  
  // ✅ Versioning support
  contentVersions ContentVersion[]
  
  // ✅ Block-level granularity
  blocks ContentBlock[]
}
```

**Status**: ✅ **EXCELLENT** - Fully extensible for gamification via JSONB fields

#### 2. **Knowledge Graph (WeaveEdge + StrandLink)**
```prisma
model WeaveEdge {
  // ✅ Typed relationships
  relationshipType String // prerequisite, related, part-of, references, etc.
  weight Float @default(1.0)
  metadata Json
  
  // ✅ Bidirectional support
  source/target Strand relations
  
  // ✅ Creator tracking
  createdBy String?
}

model BlockRelation {
  // ✅ REVOLUTIONARY: Block-to-block relationships
  sourceBlockId/targetBlockId
  relationType String
  bidirectional Boolean
  confidence Float?
}
```

**Status**: ✅ **EXCELLENT** - Block-level graph is industry-leading

#### 3. **Learning Progress & Spaced Repetition**
```prisma
model LearningProgress {
  // ✅ Spaced repetition (SM2 algorithm)
  lastReview DateTime?
  nextReview DateTime?
  reviewInterval Int
  easeFactor Float @default(2.5)
  
  // ✅ Mastery tracking
  masteryLevel Float @default(0.0)
  completionPercentage Float
  
  // ✅ Time tracking
  timeSpent Int
  
  // ✅ User annotations
  notes String?
  bookmarks Json
  highlights Json
}
```

**Status**: ✅ **GOOD** - Solid foundation, ready for gamification

#### 4. **Hierarchies & Scopes**
```prisma
model StrandHierarchy {
  // ✅ Multiple parent support
  strandId/parentId/scopeId composite key
  depth Int
  position Int
  path String
  isPrimary Boolean
}

model StrandScope {
  // ✅ Multi-tenant isolation
  scopeType HierarchyScopeType
  ownerId/teamId
  autoApprove Boolean
}
```

**Status**: ✅ **EXCELLENT** - Supports complex org structures

#### 5. **Content Versioning**
```prisma
model ContentVersion {
  // ✅ Full version history
  version Int
  content Json
  delta Json? // Efficient storage
  changeType String
  changedBy String
}
```

**Status**: ✅ **GOOD** - Git-like versioning

#### 6. **Block-Level Features (Revolutionary!)**
```prisma
model ContentBlock {
  // ✅ Paragraph-level tags
  tags String[]
  
  // ✅ Block relationships
  sourceRelations/targetRelations BlockRelation[]
  
  // ✅ Geolocation support
  geolocations Geolocation[]
}
```

**Status**: ✅ **INDUSTRY-LEADING** - No other PKMS has this

---

## Gamification Extension Analysis

### How Current Schema Extends for Gamification

#### ✅ Already Extensible via JSON Fields

**User.preferences JSON** can store:
```typescript
{
  gamification: {
    enabled: boolean;
    xp: number;
    level: number;
    currentTitle: string;
    stats: {
      strength: number;
      intelligence: number;
      wisdom: number;
      charisma: number;
      // Custom attributes
    };
    inventory: Item[];
    quests: QuestProgress[];
  };
  
  glanceProfile: {
    learningLevel: 'beginner' | 'intermediate' | 'advanced' | 'expert';
    preferences: {
      visualStyle: string;
      audioSettings: object;
      hapticFeedback: boolean;
    };
    history: GlanceSession[];
  };
}
```

**LearningProgress.metadata JSON** can store:
```typescript
{
  gamification: {
    xpGained: number;
    achievements: string[];
    streakDays: number;
    lastStreakDate: string;
    totalReviews: number;
    perfectReviews: number;
    combo: number;
  };
  
  spiralProgress: {
    encounters: number;
    currentLevel: number;
    nextLevelRequirements: object;
  };
}
```

**Strand.assessmentData JSON** can store:
```typescript
{
  quizzes: Quiz[];
  challenges: Challenge[];
  rewards: {
    xp: number;
    badges: string[];
    unlocks: string[];
  };
  difficultyRating: DifficultyRating;
}
```

### ✅ Recommended Gamification Tables (Optional)

While JSON fields work, dedicated tables provide better performance and querying:

```prisma
// New models for gamification (optional, can use JSON instead)

model Achievement {
  id String @id @default(cuid())
  userId String
  achievementType String // completion, streak, mastery, social
  
  // Achievement details
  name String
  description String
  icon String?
  rarity String // common, uncommon, rare, epic, legendary
  
  // Progress
  currentProgress Int @default(0)
  targetProgress Int
  completed Boolean @default(false)
  completedAt DateTime?
  
  // Rewards
  xpReward Int @default(0)
  badgeUrl String?
  metadata Json @default("{}")
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  
  @@index([userId])
  @@index([achievementType])
  @@index([completed])
  @@map("achievements")
}

model Quest {
  id String @id @default(cuid())
  userId String
  
  // Quest definition
  title String
  description String
  category String // learning, social, creative, challenge
  difficulty Int // 1-10
  
  // Objectives
  objectives Json // QuestObjective[]
  currentStep Int @default(0)
  totalSteps Int
  
  // Status
  status String @default("available") // available, active, completed, failed, expired
  
  // Rewards
  xpReward Int
  badgeReward String?
  itemRewards Json @default("[]")
  
  // Timing
  startedAt DateTime?
  completedAt DateTime?
  expiresAt DateTime?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@index([userId])
  @@index([status])
  @@index([category])
  @@map("quests")
}

model LearningStreak {
  id String @id @default(cuid())
  userId String
  
  // Streak data
  currentStreak Int @default(0)
  longestStreak Int @default(0)
  lastActivityDate DateTime @default(now())
  
  // Freeze/recovery
  freezeCount Int @default(0)
  canFreeze Boolean @default(true)
  
  // Stats
  totalDaysActive Int @default(0)
  streakHistory Json @default("[]") // Historical data
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  updated DateTime @updatedAt
  
  @@unique([userId])
  @@index([currentStreak])
  @@map("learning_streaks")
}

model UserStats {
  id String @id @default(cuid())
  userId String @unique
  
  // XP and leveling
  totalXP Int @default(0)
  level Int @default(1)
  xpToNextLevel Int @default(100)
  
  // Attributes (for GLANCE RPG system)
  strength Int @default(10) // Physical/kinesthetic learning
  intelligence Int @default(10) // Logical-mathematical
  wisdom Int @default(10) // Metacognition/self-awareness
  charisma Int @default(10) // Social learning
  dexterity Int @default(10) // Speed of learning
  constitution Int @default(10) // Persistence/grit
  
  // Learning statistics
  totalLessonsCompleted Int @default(0)
  totalTimeSpent Int @default(0) // seconds
  averageScore Float @default(0.0)
  perfectStreak Int @default(0)
  
  // Social
  helpfulVotes Int @default(0)
  contributions Int @default(0)
  
  // Metadata
  customAttributes Json @default("{}")
  titles Json @default("[]") // Earned titles
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  updated DateTime @updatedAt
  
  @@index([level])
  @@index([totalXP])
  @@map("user_stats")
}
```

**Assessment**: ✅ **Schema is fully extensible** - Can add gamification via JSON NOW or dedicated tables LATER

---

## Missing PKMS Features

### ❌ Critical Missing Features

#### 1. **Daily Notes / Journal System**

**Status**: ❌ **MISSING** - Major PKMS feature gap

**What's Needed**:
```prisma
model DailyNote {
  id String @id @default(cuid())
  userId String
  date DateTime @db.Date
  
  // Content
  strandId String? // Link to actual note strand
  summary String?
  mood String? // Optional mood tracking
  weather String? // Context tracking
  
  // Quick capture
  quickNotes Json @default("[]") // Array of quick captures
  tasks Json @default("[]") // Daily tasks
  habits Json @default("[]") // Habit tracking
  
  // Stats
  wordCount Int @default(0)
  timeSpent Int @default(0)
  
  // Templates
  templateId String?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  strand Strand? @relation(fields: [strandId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@unique([userId, date])
  @@index([userId, date])
  @@map("daily_notes")
}
```

**UI Requirements**:
- Quick keyboard shortcut (Ctrl+D) to open today's note
- Calendar view showing which days have entries
- Template system for daily notes
- Automatic backlink creation to referenced strands

---

#### 2. **Backlinks / Bidirectional Links Display**

**Status**: ⚠️ **PARTIAL** - We have the data model but missing UI features

**What We Have**:
- `WeaveEdge` tracks relationships
- `StrandLink` with bidirectional support
- Block-level relationships via `BlockRelation`

**What's Missing**:
- **Backlinks Panel**: UI component showing "Linked References" and "Unlinked Mentions"
- **Auto-linking**: Detect `[[wikilink]]` syntax and create relationships
- **Mention tracking**: Track which strands mention others without explicit links
- **Backlink suggestions**: AI-powered suggestion of related content

**Implementation Needed**:
```typescript
// Backend API endpoint
GET /api/v1/strands/:id/backlinks
Response: {
  linkedReferences: Strand[]; // Explicit links pointing here
  unlinkedMentions: Array<{    // Text mentions without links
    strandId: string;
    context: string;
    confidence: number;
  }>;
  suggestedLinks: Strand[];   // AI-suggested related content
}

// Frontend component
<BacklinksPanel strandId={id}>
  <Section title="Linked References" count={linked.length}>
    {linked.map(ref => <BacklinkCard strand={ref} />)}
  </Section>
  
  <Section title="Unlinked Mentions" count={unlinked.length}>
    {unlinked.map(mention => (
      <MentionCard 
        mention={mention}
        onConvertToLink={() => createLink(mention)}
      />
    ))}
  </Section>
</BacklinksPanel>
```

---

#### 3. **Graph View / Local Graph**

**Status**: ⚠️ **PARTIAL** - Global graph exists, missing local graph

**What We Have**:
- Global WeaveViewer (D3 force-directed)
- 3D KnowledgeGraphScene
- Basic node/edge visualization

**What's Missing**:
- **Local Graph**: Shows only immediate neighbors (1-2 hops) of current strand
- **Filters**: Filter by relationship type, date range, tag
- **Depth Control**: Adjustable hop distance (1, 2, 3 hops)
- **Focus Mode**: Highlight specific paths through graph
- **Timeline View**: Temporal graph showing how concepts evolved

**Implementation Needed**:
```typescript
// Component structure
<LocalGraphView strandId={currentId}>
  <GraphControls>
    <DepthSlider value={depth} onChange={setDepth} max={3} />
    <FilterPanel>
      <RelationshipTypeFilter />
      <DateRangeFilter />
      <TagFilter />
    </FilterPanel>
  </GraphControls>
  
  <Graph3D 
    nodes={localNodes}
    edges={localEdges}
    centerNode={currentId}
    depth={depth}
    onNodeClick={navigateToStrand}
  />
</LocalGraphView>
```

---

#### 4. **Transclusion / Block Embeds**

**Status**: ❌ **MISSING** - Major PKMS feature

**Description**: Ability to embed content from one note into another, with live updates.

**What's Needed**:
```prisma
model BlockEmbed {
  id String @id @default(cuid())
  
  // Embedding location
  hostStrandId String
  hostBlockId String?
  
  // Embedded content
  sourceStrandId String
  sourceBlockId String? // null = embed entire strand
  
  // Display options
  displayMode String @default("full") // full, excerpt, reference
  showMetadata Boolean @default(true)
  
  // Sync
  lastSynced DateTime @default(now())
  syncEnabled Boolean @default(true)
  
  // Relations
  hostStrand Strand @relation("HostStrand", fields: [hostStrandId], references: [id])
  sourceStrand Strand @relation("EmbeddedStrand", fields: [sourceStrandId], references: [id])
  
  created DateTime @default(now())
  
  @@index([hostStrandId])
  @@index([sourceStrandId])
  @@map("block_embeds")
}
```

**Editor Syntax**:
```markdown
![[strand-id]]                 // Embed entire strand
![[strand-id#block-id]]        // Embed specific block
![[strand-id#^paragraph-3]]    // Embed paragraph 3
```

---

#### 5. **Templates & Snippets System**

**Status**: ⚠️ **PARTIAL** - Basic templates exist, missing robust system

**What We Have**:
- `Template` model with categories
- Template selector UI
- Basic variable substitution

**What's Missing**:
- **Custom User Templates**: Users can't easily create/share templates
- **Snippet Library**: Reusable content blocks
- **Dynamic Templates**: Templates that query/generate content
- **Template Variables**: Rich variable system with defaults/validation
- **Template Marketplace**: Community-contributed templates

**Enhancement Needed**:
```prisma
model Snippet {
  id String @id @default(cuid())
  userId String
  
  // Snippet details
  title String
  description String?
  trigger String // Shortcut to insert (e.g., "/meet")
  
  // Content
  content Json
  contentType String // tiptap, markdown, code
  
  // Variables
  variables Json @default("[]") // { name, type, default, required }
  
  // Usage
  usageCount Int @default(0)
  isPublic Boolean @default(false)
  
  // Categories
  category String? // meeting, template, code, etc.
  tags String[]
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@index([userId])
  @@index([trigger])
  @@index([category])
  @@map("snippets")
}

// Enhance existing Template model
model Template {
  // ADD:
  isUserCreated Boolean @default(false)
  isShared Boolean @default(false)
  sharedWithTeams String[] @default([])
  
  // Dynamic content generation
  dynamicQueries Json @default("[]") // Queries to populate template
  conditionalSections Json @default("[]") // Show/hide based on variables
  
  // Usage analytics
  lastUsed DateTime?
  usageByUser Json @default("{}")
}
```

---

#### 6. **Smart Search / Semantic Search**

**Status**: ⚠️ **PARTIAL** - Text search exists, semantic search needs enhancement

**What We Have**:
- Elasticsearch integration
- Vector embeddings in `VectorEmbedding` model
- RAG system with embeddings

**What's Missing**:
- **Saved Searches**: Save and name complex queries
- **Search Operators**: Advanced syntax (AND, OR, NOT, tags:, type:)
- **Search History**: Recent searches with quick access
- **Search Suggestions**: Auto-complete as you type
- **Fuzzy Search**: Typo-tolerant searching

**Enhancement Needed**:
```prisma
model SavedSearch {
  id String @id @default(cuid())
  userId String
  
  // Search details
  name String
  description String?
  query String
  filters Json // { tags, types, dates, etc. }
  
  // Display
  icon String?
  color String?
  pinned Boolean @default(false)
  
  // Usage
  lastExecuted DateTime?
  executionCount Int @default(0)
  resultCount Int?
  
  // Notifications
  notifyOnNewResults Boolean @default(false)
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@index([userId])
  @@index([pinned])
  @@map("saved_searches")
}

model SearchHistory {
  id String @id @default(cuid())
  userId String
  
  query String
  filters Json?
  resultCount Int
  clickedResults String[] // IDs of results user clicked
  
  timestamp DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id])
  
  @@index([userId, timestamp])
  @@map("search_history")
}
```

---

#### 7. **Aliases / Alternate Names**

**Status**: ❌ **MISSING** - Important for flexible note-taking

**Description**: Multiple names for the same concept (e.g., "ML" = "Machine Learning" = "машинное обучение")

**Implementation Needed**:
```prisma
model StrandAlias {
  id String @id @default(cuid())
  strandId String
  
  alias String
  language String? // ISO 639-1
  isPrimary Boolean @default(false)
  
  // Relations
  strand Strand @relation(fields: [strandId], references: [id])
  
  created DateTime @default(now())
  
  @@unique([strandId, alias])
  @@index([alias]) // Fast lookup by alias
  @@index([strandId])
  @@map("strand_aliases")
}
```

**Editor Integration**:
```markdown
[[ML]] or [[Machine Learning]] → Both resolve to same strand
[[ML|Machine Learning]] → Display "Machine Learning" but link to "ML"
```

---

#### 8. **Page Properties / Frontmatter**

**Status**: ⚠️ **PARTIAL** - Metadata exists but not user-extensible

**What's Missing**:
- **Custom Properties**: User-defined properties per strand type
- **Property Types**: String, Number, Date, Select, Multi-select, Person, URL
- **Property Templates**: Reusable property sets
- **Property-based Queries**: Filter/sort by custom properties

**Enhancement Needed**:
```prisma
model PropertyDefinition {
  id String @id @default(cuid())
  userId String?
  teamId String?
  
  // Property details
  name String
  key String // Machine-readable key
  type String // text, number, date, select, multiselect, person, url, email
  
  // Validation
  required Boolean @default(false)
  validation Json? // { min, max, pattern, options }
  defaultValue Json?
  
  // Display
  icon String?
  description String?
  group String? // Group properties together
  
  // Scope
  appliesTo String[] // strandTypes this applies to
  isGlobal Boolean @default(false)
  
  created DateTime @default(now())
  
  @@unique([userId, key])
  @@index([userId])
  @@index([teamId])
  @@map("property_definitions")
}

// Enhance Strand model
model Strand {
  // ADD:
  properties Json @default("{}") // User-defined properties
  computedProperties Json @default("{}") // Auto-calculated properties
}
```

---

#### 9. **Outline / Document Outline**

**Status**: ❌ **MISSING** - Essential for navigation

**Description**: Collapsible outline showing document structure (headings)

**Implementation**: Can be computed from `ContentBlock` where `blockType = 'heading'`

**UI Component Needed**:
```typescript
<DocumentOutline strandId={id}>
  <OutlineTree nodes={outlineNodes}>
    {(node) => (
      <OutlineItem
        level={node.level}
        title={node.title}
        onClick={() => scrollToBlock(node.blockId)}
      />
    )}
  </OutlineTree>
</DocumentOutline>
```

---

#### 10. **Orphan Detection**

**Status**: ❌ **MISSING** - Important for knowledge graph hygiene

**Description**: Find strands with no incoming/outgoing links

**Implementation Needed**:
```typescript
// Backend service method
class StrandService {
  async findOrphans(userId: string, scopeId?: string): Promise<Strand[]> {
    return prisma.strand.findMany({
      where: {
        ownerId: userId,
        primaryScopeId: scopeId,
        AND: [
          { outgoingLinks: { none: {} } },
          { incomingLinks: { none: {} } }
        ]
      }
    });
  }
  
  async findDangling(userId: string): Promise<Strand[]> {
    // Strands with only outgoing links (no incoming)
    return prisma.strand.findMany({
      where: {
        ownerId: userId,
        incomingLinks: { none: {} },
        outgoingLinks: { some: {} }
      }
    });
  }
  
  async findHubs(userId: string, minConnections: number = 5): Promise<Strand[]> {
    // Strands with many connections (hub notes)
    const result = await prisma.$queryRaw`
      SELECT s.*, 
        (SELECT COUNT(*) FROM strand_links WHERE source_id = s.id) +
        (SELECT COUNT(*) FROM strand_links WHERE target_id = s.id) as connection_count
      FROM strands s
      WHERE s.owner_id = ${userId}
      HAVING connection_count >= ${minConnections}
      ORDER BY connection_count DESC
      LIMIT 50
    `;
    return result as Strand[];
  }
}
```

---

#### 11. **Workspaces / Multiple Vaults**

**Status**: ⚠️ **PARTIAL** - StrandScope exists but not exposed as "vaults"

**What We Have**:
- `StrandScope` with types (COLLECTION, DATASET, PROJECT, TEAM, GLOBAL)
- Multi-tenant isolation

**What's Missing**:
- **Vault UI**: Switch between completely separate knowledge bases
- **Vault-level Settings**: Each vault has its own theme, templates, preferences
- **Cross-vault Linking**: Link between vaults (with permission)
- **Vault Sync**: Different sync strategies per vault

**Enhancement Needed**:
```prisma
model Vault {
  id String @id @default(cuid())
  userId String
  
  // Vault details
  name String
  slug String
  description String?
  icon String?
  color String?
  
  // Configuration
  theme String @default("default")
  settings Json @default("{}")
  
  // Stats
  strandCount Int @default(0)
  totalSize BigInt @default(0)
  
  // Sync
  syncEnabled Boolean @default(true)
  lastSynced DateTime?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  scopes StrandScope[]
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@unique([userId, slug])
  @@index([userId])
  @@map("vaults")
}
```

---

#### 12. **Markdown-style Wikilinks**

**Status**: ❌ **MISSING** - Critical PKMS UX feature

**What's Needed**:
- Parse `[[Note Title]]` syntax
- Auto-create strands on-the-fly if they don't exist
- Show suggestions while typing
- Support aliases: `[[Note Title|Display Text]]`
- Support headers: `[[Note Title#Heading]]`
- Support blocks: `[[Note Title#^block-id]]`

**Editor Extension Needed**:
```typescript
// TipTap extension for wikilinks
import { Node, mergeAttributes } from '@tiptap/core';

export const WikiLink = Node.create({
  name: 'wikilink',
  
  group: 'inline',
  inline: true,
  content: 'text*',
  
  addAttributes() {
    return {
      strandId: { default: null },
      title: { default: null },
      alias: { default: null },
      anchor: { default: null }, // #heading or #^block
      exists: { default: true },
    };
  },
  
  parseHTML() {
    return [{ tag: 'a[data-wikilink]' }];
  },
  
  renderHTML({ HTMLAttributes }) {
    return ['a', mergeAttributes(HTMLAttributes, {
      'data-wikilink': '',
      'href': `#`,
      'class': HTMLAttributes.exists ? 'wikilink' : 'wikilink-broken'
    }), 0];
  },
  
  addInputRules() {
    return [
      {
        find: /\[\[([^\]]+)\]\]/,
        handler: ({ range, match }) => {
          const title = match[1];
          // Create wikilink node
          // Trigger autocomplete suggestions
        }
      }
    ];
  }
});
```

---

#### 13. **Tags Hierarchy / Nested Tags**

**Status**: ❌ **MISSING** - Tags are flat arrays currently

**What's Needed**:
```prisma
model Tag {
  id String @id @default(cuid())
  userId String
  
  // Tag details
  name String
  slug String
  color String?
  icon String?
  
  // Hierarchy
  parentTagId String?
  path String // Full path (e.g., "coding/python/django")
  level Int @default(0)
  
  // Usage stats
  strandCount Int @default(0)
  lastUsed DateTime?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  parent Tag? @relation("TagHierarchy", fields: [parentTagId], references: [id])
  children Tag[] @relation("TagHierarchy")
  
  created DateTime @default(now())
  
  @@unique([userId, slug])
  @@index([userId])
  @@index([parentTagId])
  @@index([path])
  @@map("tags")
}

model StrandTag {
  strandId String
  tagId String
  
  // Tagging metadata
  addedBy String?
  addedAt DateTime @default(now())
  source String @default("user") // user, ai, import
  
  strand Strand @relation(fields: [strandId], references: [id])
  tag Tag @relation(fields: [tagId], references: [id])
  
  @@id([strandId, tagId])
  @@index([strandId])
  @@index([tagId])
  @@map("strand_tags")
}
```

**UI Features**:
- Tag tree view
- Drag-and-drop tag reorganization
- Tag merging and splitting
- Unused tag cleanup
- Tag suggestions based on content

---

#### 14. **Quick Capture / Inbox**

**Status**: ❌ **MISSING** - Essential for rapid note-taking

**Description**: Capture ideas quickly without interrupting flow

**What's Needed**:
```prisma
model InboxItem {
  id String @id @default(cuid())
  userId String
  
  // Content
  content String @db.Text
  contentType String @default("text") // text, url, image, voice
  
  // Quick metadata
  tags String[]
  quickNote String?
  
  // Processing
  processed Boolean @default(false)
  processedAt DateTime?
  convertedToStrandId String?
  
  // Source tracking
  source String @default("manual") // manual, mobile, email, browser-extension
  sourceUrl String?
  sourceDevice String?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  
  @@index([userId, processed])
  @@index([created])
  @@map("inbox_items")
}
```

**UI Integration**:
- Global keyboard shortcut (Ctrl+Shift+C)
- Mobile widget
- Browser extension
- Email-to-inbox
- Voice memo capture

---

#### 15. **Timeline View / Activity Feed**

**Status**: ❌ **MISSING** - Useful for reviewing recent work

**What's Needed**:
```prisma
model ActivityEvent {
  id String @id @default(cuid())
  userId String
  
  // Event details
  eventType String // created, updated, linked, tagged, reviewed
  entityType String // strand, link, comment, etc.
  entityId String
  
  // Changes
  changes Json? // What changed
  previousValue Json?
  newValue Json?
  
  // Context
  sessionId String?
  deviceInfo String?
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  timestamp DateTime @default(now())
  
  @@index([userId, timestamp])
  @@index([eventType])
  @@map("activity_events")
}
```

---

#### 16. **Canvas / Whiteboard Mode**

**Status**: ⚠️ **PARTIAL** - EditorState supports Excalidraw but needs integration

**What We Have**:
```prisma
model EditorState {
  contentType String @default("tiptap") // Can be 'excalidraw'
  content Json
}
```

**What's Missing**:
- **Canvas View**: Infinite canvas for spatial organization
- **Card Positioning**: Free-form positioning of strand cards
- **Drawing Tools**: Arrows, shapes, annotations
- **Canvas Templates**: Pre-designed layouts
- **Canvas Export**: Export as image/PDF

---

#### 17. **Breadcrumbs Navigation**

**Status**: ⚠️ **PARTIAL** - StrandHierarchy supports it but UI needs enhancement

**Implementation Needed**:
```typescript
// Service method
class StrandService {
  async getBreadcrumbs(strandId: string, scopeId: string): Promise<Breadcrumb[]> {
    const hierarchy = await prisma.strandHierarchy.findUnique({
      where: { strandId_scopeId: { strandId, scopeId } }
    });
    
    if (!hierarchy) return [];
    
    // Parse path to get all ancestors
    const ancestorIds = hierarchy.path.split('/').filter(Boolean);
    
    const ancestors = await prisma.strand.findMany({
      where: { id: { in: ancestorIds } },
      select: { id: true, title: true, slug: true }
    });
    
    // Build breadcrumb trail
    return ancestorIds.map(id => 
      ancestors.find(a => a.id === id)
    ).filter(Boolean) as Breadcrumb[];
  }
}
```

---

#### 18. **Starred / Pinned Items**

**Status**: ⚠️ **PARTIAL** - Favorites exist but no pinning UI

**What We Have**:
```prisma
model Favorite {
  userId String
  strandId String
}
```

**What's Missing**:
- **Pin to Sidebar**: Quick access sidebar with pinned items
- **Pin Order**: User-controlled ordering
- **Pin to Scope**: Pin within specific scope/vault
- **Smart Pins**: Auto-pin frequently accessed items

**Enhancement Needed**:
```prisma
model Favorite {
  // ADD:
  isPinned Boolean @default(false)
  pinnedOrder Int?
  pinnedScope String? // Where to show pin
  autoPin Boolean @default(false) // Auto-pinned by system
  
  @@index([userId, isPinned, pinnedOrder])
}
```

---

#### 19. **Reading List / Queue**

**Status**: ❌ **MISSING** - Useful for managing learning pipeline

**Implementation Needed**:
```prisma
model ReadingList {
  id String @id @default(cuid())
  userId String
  
  name String @default("Reading List")
  description String?
  
  // Items
  items ReadingListItem[]
  
  // Settings
  autoAddSuggestions Boolean @default(false)
  sortOrder String @default("manual") // manual, priority, date
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@index([userId])
  @@map("reading_lists")
}

model ReadingListItem {
  id String @id @default(cuid())
  listId String
  strandId String
  
  // Status
  status String @default("unread") // unread, reading, completed, archived
  priority Int @default(0)
  position Int
  
  // Progress
  progress Float @default(0.0) // 0-1
  estimatedTime Int? // minutes
  timeSpent Int @default(0) // seconds
  
  // Notes
  note String?
  rating Int? // 1-5
  
  // Relations
  list ReadingList @relation(fields: [listId], references: [id])
  strand Strand @relation(fields: [strandId], references: [id])
  
  addedAt DateTime @default(now())
  startedAt DateTime?
  completedAt DateTime?
  
  @@unique([listId, strandId])
  @@index([listId, status])
  @@map("reading_list_items")
}
```

---

#### 20. **Collaborative Editing (Real-Time)**

**Status**: ⚠️ **PARTIAL** - PresenceSession exists but no CRDT

**What We Have**:
```prisma
model PresenceSession {
  userId String
  strandId String
  status String // viewing, editing, idle
  lastActivity DateTime
}
```

**What's Missing**:
- **Y.js Integration**: CRDT for conflict-free collaborative editing
- **Cursor Positions**: Show where other users are typing
- **Selection Sharing**: Share highlighted text
- **Awareness**: See who's viewing/editing in real-time

**Implementation Path**:
- Integrate Y.js with TipTap
- WebSocket server for awareness protocol
- Store Y.js documents in separate table or Redis

---

#### 21. **Flashcards / Anki Integration**

**Status**: ❌ **MISSING** - Important for active recall

**What's Needed**:
```prisma
model Flashcard {
  id String @id @default(cuid())
  strandId String
  userId String
  
  // Card content
  front String @db.Text
  back String @db.Text
  reversible Boolean @default(false)
  
  // Context
  blockId String? // Source block if generated from block
  tags String[]
  
  // Spaced repetition (Anki-compatible)
  deckName String @default("Default")
  ease Float @default(2.5)
  interval Int @default(1)
  lapses Int @default(0)
  due DateTime
  
  // Stats
  reviews Int @default(0)
  againCount Int @default(0)
  hardCount Int @default(0)
  goodCount Int @default(0)
  easyCount Int @default(0)
  
  // Status
  suspended Boolean @default(false)
  buried Boolean @default(false)
  
  // Relations
  strand Strand @relation(fields: [strandId], references: [id])
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  modified DateTime @updatedAt
  
  @@index([userId, due])
  @@index([strandId])
  @@index([deckName])
  @@map("flashcards")
}
```

---

#### 22. **Semantic Suggestions / "Similar To"**

**Status**: ⚠️ **PARTIAL** - `getRelated` API exists but basic

**Enhancement Needed**:
- **Why Similar**: Explain why items are related
- **Similarity Score**: Show confidence levels
- **Similarity Types**: By content, by structure, by usage patterns
- **Temporal Similarity**: "People who read this also read..."

**Service Enhancement**:
```typescript
class StrandService {
  async getSimilarStrands(
    strandId: string,
    options: {
      method: 'content' | 'structural' | 'behavioral' | 'hybrid';
      limit: number;
      minScore: number;
    }
  ): Promise<Array<{
    strand: Strand;
    similarity: number;
    reasons: SimilarityReason[];
  }>> {
    // Use vector embeddings for content similarity
    // Use graph structure for structural similarity
    // Use user behavior for behavioral similarity
    // Combine all three for hybrid
  }
}
```

---

#### 23. **Export Formats**

**Status**: ⚠️ **PARTIAL** - Basic exports exist, missing advanced formats

**What We Have**:
- JSON export
- Markdown export (basic)

**What's Missing**:
- **Obsidian Format**: Full Obsidian vault export with wikilinks
- **Roam Format**: Roam Research JSON format
- **Notion Format**: Notion-compatible export
- **Org-mode**: Emacs org-mode export
- **PDF with Links**: Interactive PDF preserving internal links
- **OPML**: Outline format
- **CSV of Metadata**: Metadata table export

---

#### 24. **Import Formats**

**Status**: ⚠️ **PARTIAL** - Basic imports exist, missing popular formats

**What We Have**:
- Zip upload
- Individual file upload

**What's Missing**:
- **Obsidian Vault Import**: Parse entire vault with wikilinks
- **Notion Export Import**: Parse Notion's export format
- **Roam JSON Import**: Import Roam graphs
- **Evernote ENEX**: Import Evernote notebooks
- **Bear Notes**: Import Bear's markdown
- **Apple Notes**: Import via export
- **OneNote**: Parse OneNote exports

---

#### 25. **Conflict Resolution UI**

**Status**: ❌ **MISSING** - Important for sync scenarios

**What's Needed**:
- Visual diff view
- Three-way merge UI
- Automatic conflict detection
- Conflict history

---

#### 26. **Obsidian-style Callouts**

**Status**: ❌ **MISSING** - Popular markdown extension

**Syntax**:
```markdown
> [!note] This is a note
> Content goes here

> [!warning] Watch out!
> Be careful with this

> [!tip] Pro tip
> This will save you time
```

**Implementation**: TipTap extension + CSS styling

---

#### 27. **Metadata Panel / Properties Panel**

**Status**: ⚠️ **PARTIAL** - Metadata exists but not prominent in UI

**What's Needed**:
- Sidebar panel showing all properties
- Editable properties
- Property suggestions
- Computed properties (word count, links, etc.)

---

#### 28. **Random Note / Serendipity**

**Status**: ❌ **MISSING** - Fun discovery feature

**Implementation**:
```typescript
// Service method
async getRandomStrand(
  userId: string,
  filters?: {
    unread?: boolean;
    tags?: string[];
    minAge?: number; // days since last viewed
  }
): Promise<Strand> {
  return prisma.$queryRaw`
    SELECT * FROM strands
    WHERE owner_id = ${userId}
      ${filters?.unread ? 'AND id NOT IN (SELECT strand_id FROM learning_progress WHERE user_id = ${userId})' : ''}
      ${filters?.minAge ? `AND last_accessed < NOW() - INTERVAL '${filters.minAge} days'` : ''}
    ORDER BY RANDOM()
    LIMIT 1
  `;
}
```

---

#### 29. **Note Refactoring Tools**

**Status**: ❌ **MISSING** - Important for maintaining knowledge base

**Features Needed**:
- **Extract Block**: Move block to new strand
- **Merge Strands**: Combine multiple strands
- **Split Strand**: Break large strand into multiple
- **Rename & Update Links**: Rename strand and update all references
- **Batch Operations**: Apply operations to multiple strands

---

#### 30. **Mobile-Specific Features**

**Status**: ⚠️ **PARTIAL** - Mobile app exists but missing features

**What's Missing**:
- **Offline Mode**: Full offline editing with sync
- **Voice Notes**: Quick voice capture
- **Photo Notes**: Camera integration
- **Location-based Notes**: Auto-tag with location
- **Quick Share**: Share to OpenStrand from other apps
- **Widget**: Home screen widget for quick capture

---

## Gamification Schema Extensions

### Approach 1: Lightweight (Use Existing JSON Fields)

**Advantages**:
- No schema changes needed
- Deploy immediately
- Flexible and fast to iterate

**Use**:
- `User.preferences.gamification`
- `LearningProgress.metadata.gamification`
- `Strand.assessmentData`

### Approach 2: Dedicated Tables (Recommended for Scale)

**Advantages**:
- Better query performance
- Proper indexing
- Type safety
- Analytics-friendly

**Recommended New Models**:

```prisma
// ============================================================================
// GAMIFICATION & GLANCE RPG SYSTEM
// ============================================================================

/// User gamification profile
model GamificationProfile {
  id String @id @default(cuid())
  userId String @unique
  
  // XP and leveling
  totalXP BigInt @default(0)
  level Int @default(1)
  xpToNextLevel Int @default(100)
  
  // Character attributes (GLANCE RPG)
  attributes Json @default("{}") // { strength, intelligence, wisdom, etc. }
  
  // Progression
  title String? // Current title/rank
  availableTitles String[] @default([])
  
  // Stats
  lessonsCompleted Int @default(0)
  perfectReviews Int @default(0)
  helpfulContributions Int @default(0)
  
  // Streaks
  currentStreak Int @default(0)
  longestStreak Int @default(0)
  lastActiveDate DateTime @default(now())
  freezeTokens Int @default(0)
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  achievements Achievement[]
  quests Quest[]
  
  updated DateTime @updatedAt
  
  @@index([level])
  @@index([totalXP])
  @@map("gamification_profiles")
}

model Achievement {
  id String @id @default(cuid())
  profileId String
  
  // Achievement definition
  achievementId String // Reference to achievement definition
  name String
  description String
  category String // learning, social, mastery, creative
  
  // Progress
  progress Int @default(0)
  target Int
  completed Boolean @default(false)
  
  // Reward
  xpReward Int
  badgeUrl String?
  itemRewards Json @default("[]")
  
  // Dates
  unlockedAt DateTime?
  completedAt DateTime?
  
  profile GamificationProfile @relation(fields: [profileId], references: [id])
  
  @@unique([profileId, achievementId])
  @@index([profileId, completed])
  @@map("achievements")
}

model Quest {
  id String @id @default(cuid())
  profileId String
  
  // Quest details
  questId String // Reference to quest definition
  title String
  description String
  narrative String? // Story context
  
  // Progress
  objectives Json // QuestObjective[]
  currentObjective Int @default(0)
  status String // available, active, completed, failed, expired
  
  // Rewards
  xpReward Int
  rewards Json
  
  // Timing
  startedAt DateTime?
  completedAt DateTime?
  expiresAt DateTime?
  
  profile GamificationProfile @relation(fields: [profileId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@index([profileId, status])
  @@map("quests")
}

model Badge {
  id String @id @default(cuid())
  userId String
  
  // Badge details
  badgeType String
  name String
  description String
  icon String
  rarity String // common, rare, epic, legendary
  
  // Conditions
  earnedFor String // What triggered the badge
  strandId String? // Related strand if applicable
  
  // Display
  isVisible Boolean @default(true)
  isPinned Boolean @default(false)
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  earnedAt DateTime @default(now())
  
  @@index([userId])
  @@index([badgeType])
  @@map("badges")
}

model LeaderboardEntry {
  id String @id @default(cuid())
  userId String
  
  // Leaderboard type
  boardType String // global, team, topic, weekly, monthly
  metric String // xp, lessons, contributions, streaks
  
  // Position
  rank Int
  score BigInt
  
  // Period
  periodStart DateTime
  periodEnd DateTime
  
  // Relations
  user User @relation(fields: [userId], references: [id])
  
  calculated DateTime @default(now())
  
  @@unique([userId, boardType, periodStart])
  @@index([boardType, rank])
  @@map("leaderboard_entries")
}
```

---

## PKMS Feature Priority Matrix

### Tier 1: Critical (Implement First)

| Feature | Complexity | Impact | Current Status |
|---------|-----------|--------|----------------|
| Backlinks Panel UI | Medium | High | ⚠️ Partial (data exists) |
| Wikilinks `[[]]` | Medium | High | ❌ Missing |
| Local Graph View | Medium | High | ⚠️ Partial (global only) |
| Daily Notes | Low | High | ❌ Missing |
| Quick Capture/Inbox | Low | High | ❌ Missing |
| Orphan Detection | Low | Medium | ❌ Missing |

### Tier 2: Important (Implement Soon)

| Feature | Complexity | Impact | Current Status |
|---------|-----------|--------|----------------|
| Transclusion/Embeds | High | Medium | ❌ Missing |
| Tag Hierarchy | Medium | Medium | ❌ Missing (flat only) |
| Templates Enhancement | Medium | Medium | ⚠️ Partial |
| Saved Searches | Low | Medium | ❌ Missing |
| Reading List | Low | Medium | ❌ Missing |
| Timeline View | Low | Medium | ❌ Missing |

### Tier 3: Nice-to-Have (Future)

| Feature | Complexity | Impact | Current Status |
|---------|-----------|--------|----------------|
| Canvas/Whiteboard | High | Medium | ⚠️ Partial (Excalidraw support) |
| Real-time Collab (CRDT) | Very High | Medium | ⚠️ Partial (presence only) |
| Flashcards | Medium | Low | ❌ Missing |
| Advanced Export Formats | Medium | Low | ⚠️ Partial |
| Smart Suggestions | High | Medium | ⚠️ Partial |
| Conflict Resolution UI | High | Low | ❌ Missing |

---

## Recommended Schema Additions

### Priority 1: Add These Models Now

```prisma
// Add to schema.prisma

// 1. Daily Notes
model DailyNote {
  id String @id @default(cuid())
  userId String
  date DateTime @db.Date
  strandId String?
  summary String?
  mood String?
  quickNotes Json @default("[]")
  tasks Json @default("[]")
  habits Json @default("[]")
  
  user User @relation(fields: [userId], references: [id])
  strand Strand? @relation(fields: [strandId], references: [id])
  
  created DateTime @default(now())
  updated DateTime @updatedAt
  
  @@unique([userId, date])
  @@index([userId, date])
  @@map("daily_notes")
}

// 2. Inbox for quick capture
model InboxItem {
  id String @id @default(cuid())
  userId String
  content String @db.Text
  contentType String @default("text")
  tags String[]
  processed Boolean @default(false)
  processedAt DateTime?
  convertedToStrandId String?
  source String @default("manual")
  
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  
  @@index([userId, processed])
  @@map("inbox_items")
}

// 3. Strand aliases
model StrandAlias {
  id String @id @default(cuid())
  strandId String
  alias String
  language String?
  isPrimary Boolean @default(false)
  
  strand Strand @relation(fields: [strandId], references: [id])
  
  created DateTime @default(now())
  
  @@unique([strandId, alias])
  @@index([alias])
  @@map("strand_aliases")
}

// 4. Tag system
model Tag {
  id String @id @default(cuid())
  userId String
  name String
  slug String
  color String?
  icon String?
  parentTagId String?
  path String
  level Int @default(0)
  strandCount Int @default(0)
  
  user User @relation(fields: [userId], references: [id])
  parent Tag? @relation("TagHierarchy", fields: [parentTagId], references: [id])
  children Tag[] @relation("TagHierarchy")
  
  created DateTime @default(now())
  
  @@unique([userId, slug])
  @@index([userId])
  @@index([path])
  @@map("tags")
}

// 5. Saved searches
model SavedSearch {
  id String @id @default(cuid())
  userId String
  name String
  query String
  filters Json
  pinned Boolean @default(false)
  notifyOnNewResults Boolean @default(false)
  
  user User @relation(fields: [userId], references: [id])
  
  created DateTime @default(now())
  
  @@index([userId])
  @@map("saved_searches")
}
```

### Priority 2: Gamification Tables (Optional - Can Use JSON)

```prisma
// Only add if you want dedicated gamification tables
// Otherwise, use User.preferences and LearningProgress.metadata

model GamificationProfile {
  id String @id @default(cuid())
  userId String @unique
  totalXP BigInt @default(0)
  level Int @default(1)
  attributes Json // GLANCE attributes
  currentStreak Int @default(0)
  longestStreak Int @default(0)
  
  user User @relation(fields: [userId], references: [id])
  achievements Achievement[]
  
  @@map("gamification_profiles")
}

model Achievement {
  id String @id @default(cuid())
  profileId String
  achievementId String
  name String
  progress Int
  target Int
  completed Boolean @default(false)
  
  profile GamificationProfile @relation(fields: [profileId], references: [id])
  
  @@unique([profileId, achievementId])
  @@map("achievements")
}
```

---

## Migration Strategy

### Phase 1: High-Impact, Low-Complexity (Week 1-2)

1. **Daily Notes System**
   - Add `DailyNote` model
   - Create API endpoints
   - Build UI component
   - Add keyboard shortcut

2. **Inbox/Quick Capture**
   - Add `InboxItem` model
   - Global quick capture modal
   - Mobile integration
   - Process inbox workflow

3. **Backlinks Panel UI**
   - Build frontend component
   - Enhance `/strands/:id/backlinks` endpoint
   - Add unlinked mentions detection
   - Show in strand view sidebar

4. **Local Graph View**
   - Add depth parameter to graph API
   - Build local graph component
   - Add filters and controls
   - Integrate into strand view

### Phase 2: Enhanced Experience (Week 3-4)

5. **Wikilink Support**
   - TipTap extension for `[[]]` syntax
   - Auto-complete dropdown
   - Broken link detection
   - Create-on-type for missing strands

6. **Strand Aliases**
   - Add `StrandAlias` model
   - Update search to include aliases
   - UI for managing aliases
   - Wikilink resolution with aliases

7. **Tag Hierarchy**
   - Add `Tag` model
   - Migrate flat tags to hierarchical
   - Tag tree UI component
   - Tag-based filtering and search

8. **Saved Searches**
   - Add `SavedSearch` model
   - Advanced search syntax
   - Save/pin/notify features
   - Search history tracking

### Phase 3: Advanced Features (Month 2)

9. **Transclusion/Embeds**
   - Add `BlockEmbed` model
   - Embed syntax and parser
   - Live update mechanism
   - Embed UI components

10. **Flashcard System**
    - Add `Flashcard` model
    - Anki-compatible SRS
    - Flashcard generation from content
    - Study mode UI

11. **Canvas Mode**
    - Enhance `EditorState` for canvas
    - Infinite canvas component
    - Spatial positioning
    - Export to image

12. **Advanced Exports**
    - Obsidian vault format
    - Notion export format
    - Interactive PDF
    - OPML outline

### Phase 4: Gamification (Month 3)

13. **Gamification Core**
    - Decide: JSON fields vs. dedicated tables
    - Implement XP/level system
    - Create achievement definitions
    - Build progression UI

14. **Quest System**
    - Quest generation from learning paths
    - Quest tracking UI
    - Narrative integration
    - Reward distribution

15. **Leaderboards**
    - Leaderboard calculation
    - Multiple board types
    - Privacy controls
    - Competitive/collaborative modes

---

## Schema Extension Examples

### Example 1: Adding XP to Existing System

**Without Schema Changes** (Use JSON):
```typescript
// When user completes a learning activity
await prisma.learningProgress.update({
  where: { userId_strandId: { userId, strandId } },
  data: {
    metadata: {
      ...existingMetadata,
      gamification: {
        xpGained: 50,
        achievements: ['first-completion'],
        streakDays: 7
      }
    }
  }
});

// Update user profile
await prisma.user.update({
  where: { id: userId },
  data: {
    preferences: {
      ...existingPrefs,
      gamification: {
        totalXP: (existingPrefs.gamification?.totalXP || 0) + 50,
        level: calculateLevel((existingPrefs.gamification?.totalXP || 0) + 50)
      }
    }
  }
});
```

### Example 2: Quest Tracking in Spiral Metadata

**Without Schema Changes**:
```typescript
await prisma.strand.update({
  where: { id: strandId },
  data: {
    spiralMetadata: {
      ...existing,
      quests: [
        {
          id: 'recursion-quest',
          title: 'Master Recursion',
          objectives: [
            { id: 'understand-base-case', completed: true },
            { id: 'write-fibonacci', completed: false }
          ],
          progress: 0.5,
          rewards: { xp: 200, badge: 'recursion-master' }
        }
      ]
    }
  }
});
```

---

## Implementation Recommendations

### Immediate Actions (This Week)

1. ✅ **Confirm Schema Design**: Schema is well-designed and extensible ✓
2. 🔨 **Add Backlinks UI**: Build the missing UI component
3. 🔨 **Add Daily Notes**: Implement daily notes system
4. 🔨 **Add Quick Capture**: Implement inbox system
5. 🔨 **Enhance Graph**: Add local graph view

### Short-term (Next Month)

6. 🔨 **Wikilinks**: Implement `[[]]` syntax
7. 🔨 **Aliases**: Add alias system
8. 🔨 **Tag Hierarchy**: Migrate to hierarchical tags
9. 🔨 **Transclusion**: Implement block embeds

### Mid-term (Next Quarter)

10. 🎮 **Gamification**: Implement core gamification (decide on approach)
11. 📱 **Mobile Polish**: Enhance mobile-specific features
12. 🎨 **Canvas Mode**: Full canvas/whiteboard implementation
13. 🔄 **Advanced Exports**: Support more export formats

---

## Technical Considerations

### JSON vs. Dedicated Tables

**Use JSON When**:
- Rapid prototyping
- Flexible/evolving schema
- Low query frequency
- Data primarily read as a whole

**Use Dedicated Tables When**:
- Need to query/filter efficiently
- Need aggregations (COUNT, SUM, etc.)
- Need proper indexing
- Data grows large

**Recommendation**: 
- Start with JSON for gamification (fast to iterate)
- Migrate to dedicated tables once proven and stable
- Use dedicated tables for core PKMS features (daily notes, aliases, tags)

### Performance Considerations

**Current Schema Performance**: ✅ Good
- Proper indexes on all foreign keys
- Composite indexes for common queries
- JSONB for flexible metadata (PostgreSQL optimized)

**Optimization Opportunities**:
1. Add GIN indexes for JSONB columns that are queried frequently
2. Materialized views for complex aggregations (leaderboards, stats)
3. Partitioning for time-series data (ActivityEvent, AnalyticsEvent)

```sql
-- Example: Add GIN indexes for better JSON querying
CREATE INDEX idx_strand_metadata_tags ON strands USING GIN ((metadata->'tags'));
CREATE INDEX idx_user_preferences_gamification ON users USING GIN ((preferences->'gamification'));
CREATE INDEX idx_learning_progress_metadata ON learning_progress USING GIN (metadata);
```

---

## Comparison with Other PKMS Systems

### vs. Obsidian

| Feature | Obsidian | OpenStrand | Status |
|---------|----------|-----------|--------|
| Wikilinks | ✅ | ❌ | **Missing** |
| Backlinks | ✅ | ⚠️ | **Data exists, UI partial** |
| Graph View | ✅ | ✅ | **Equivalent** |
| Daily Notes | ✅ | ❌ | **Missing** |
| Templates | ✅ | ⚠️ | **Basic only** |
| Plugins | ✅ | ✅ | **Equivalent** |
| Block References | ✅ | ❌ | **Missing (but have embeds)** |
| Tags | ✅ (Nested) | ⚠️ | **Flat only** |
| Canvas | ✅ | ⚠️ | **Excalidraw partial** |
| Sync | ✅ | ✅ | **Equivalent** |

### vs. Roam Research

| Feature | Roam | OpenStrand | Status |
|---------|------|-----------|--------|
| Block References | ✅ | ❌ | **Missing** |
| Bidirectional Links | ✅ | ✅ | **Equivalent** |
| Daily Notes | ✅ | ❌ | **Missing** |
| Block Embeds | ✅ | ❌ | **Missing** |
| Graph View | ✅ | ✅ | **Better (3D)** |
| Queries | ✅ | ⚠️ | **Basic only** |
| Page Properties | ✅ | ⚠️ | **Metadata exists** |
| Multiplayer | ✅ | ✅ | **Equivalent** |

### vs. Notion

| Feature | Notion | OpenStrand | Status |
|---------|--------|-----------|--------|
| Databases | ✅ | ✅ | **Better (flexible datasets)** |
| Relations | ✅ | ✅ | **Equivalent** |
| Templates | ✅ | ⚠️ | **Basic only** |
| Collaboration | ✅ | ✅ | **Equivalent** |
| Page Properties | ✅ | ⚠️ | **Not user-extensible** |
| Embeds | ✅ | ❌ | **Missing** |
| Synced Blocks | ✅ | ❌ | **Missing** |

### 🎯 OpenStrand Unique Features (Industry-Leading)

1. ✅ **Block-level tagging and relationships** (No one else has this!)
2. ✅ **Educational metadata with spiral curriculum**
3. ✅ **Multi-format publishing (PDF, web, AR/VR)**
4. ✅ **RAG/AI integration with local-first support**
5. ✅ **Geolocation support at block level**
6. ✅ **Advanced collaboration with presence**

---

## Conclusion

### ✅ Schema Assessment: **EXCELLENT**

Your current Prisma schema is:
- ✅ Well-designed and normalized
- ✅ Flexible with JSON fields
- ✅ Properly indexed
- ✅ Extensible for gamification
- ✅ Already has unique features (block-level graph)

### ⚠️ Feature Gaps: **ADDRESSABLE**

Missing features are:
- ⚠️ Standard PKMS expectations (wikilinks, daily notes, backlinks UI)
- ⚠️ Not architecture problems, just UI/service implementation
- ⚠️ Can be added incrementally without breaking changes

### 🚀 Recommended Path Forward

1. **Week 1-2**: Implement Tier 1 critical features (wikilinks, daily notes, backlinks UI, local graph)
2. **Week 3-4**: Add Tier 2 important features (tag hierarchy, transclusion, saved searches)
3. **Month 2**: Gamification using JSON fields first, migrate to dedicated tables if needed
4. **Month 3**: Advanced features and polish

### 🎮 Gamification Extension Path

**Recommended Approach**:
1. **Start with JSON** for flexibility:
   - `User.preferences.gamification`
   - `LearningProgress.metadata.gamification`
   - `Strand.assessmentData`

2. **Migrate to dedicated tables** once stable:
   - Better performance
   - Easier analytics
   - Type safety

3. **Integration points**:
   - Hook into `LearningProgress` updates
   - Track `ScheduleItem` completions
   - Monitor streaks via `lastActiveDate`
   - Award XP/achievements on milestones

The schema is **ready for gamification TODAY** using JSON fields, and can be enhanced with dedicated tables when needed.

---

*This analysis confirms OpenStrand's schema is production-ready and properly designed for both current PKMS features and future gamification extensions.*

