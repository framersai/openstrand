# Journal & Daily Dashboard UI Design

## Overview

This document outlines the UI/UX design for OpenStrand's personal journal and daily dashboard features, including the Daily tab, Records/Analytics section, and doodle integration.

## Timeline & Analytics UI Libraries

### Recommended Libraries

1. **Recharts** (Primary Choice)
   - **Why**: Built on React, composable, responsive, and well-maintained
   - **Use Cases**: Mood trends, accomplishment charts, activity timelines
   - **Pros**: TypeScript support, declarative API, lightweight
   - **Cons**: Less customizable than D3.js
   - **Installation**: `npm install recharts`

2. **@nivo/line, @nivo/calendar** (Secondary)
   - **Why**: Beautiful, animated, responsive charts with great defaults
   - **Use Cases**: Calendar heatmaps for daily activity, line charts for mood tracking
   - **Pros**: Gorgeous out-of-the-box, extensive chart types, React-friendly
   - **Cons**: Larger bundle size
   - **Installation**: `npm install @nivo/core @nivo/line @nivo/calendar`

3. **react-timeline-9000** (Timeline View)
   - **Why**: Horizontal timeline with zoom/pan, perfect for activity feeds
   - **Use Cases**: Daily activity timeline, event tracking
   - **Pros**: Interactive, customizable, handles large datasets
   - **Cons**: Less actively maintained
   - **Alternative**: Build custom with Recharts or use `react-chrono`

4. **react-chrono** (Alternative Timeline)
   - **Why**: Modern, responsive timeline component with multiple modes
   - **Use Cases**: Activity feed, accomplishment timeline
   - **Pros**: Multiple layout modes (vertical, horizontal, tree), animations
   - **Cons**: Opinionated styling
   - **Installation**: `npm install react-chrono`

5. **Tremor** (Dashboard Components)
   - **Why**: Pre-built dashboard components with Tailwind CSS
   - **Use Cases**: KPI cards, metric displays, charts
   - **Pros**: Beautiful defaults, Tailwind-based, minimal setup
   - **Cons**: Less flexible than building from scratch
   - **Installation**: `npm install @tremor/react`

### Recommended Stack

- **Primary**: Recharts for charts, react-chrono for timelines
- **Secondary**: @nivo/calendar for heatmaps
- **Dashboard**: Tremor for KPI cards and metrics
- **Custom**: D3.js (already in stack) for advanced visualizations

---

## Daily Tab UI/UX

### Purpose

The Daily tab is the user's daily check-in hub, featuring:
- Mood check-ins
- Daily accomplishments & reflections
- Inspirational quotes (personalized)
- Recommendations feed
- Notifications (team & personal)
- Quick journal entry
- Doodle pad (Excalidraw integration)

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Daily Dashboard - [Date Picker] [Today] [Tomorrow]         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │  Morning Check-In   │  │  Inspirational Quote        │  │
│  │  ☀️ How are you?    │  │  "..."                      │  │
│  │  [Mood Selector]    │  │  - Author                   │  │
│  └─────────────────────┘  └─────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Quick Journal Entry                                  │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │ [Rich Text Editor / Doodle Toggle]              │ │  │
│  │  │                                                   │ │  │
│  │  │ Write your thoughts, reflections, or doodle...  │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  │  [Save] [Doodle Mode] [Voice Note]                   │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │  Today's Goals      │  │  Accomplishments            │  │
│  │  □ Goal 1           │  │  ✓ Completed task 1         │  │
│  │  □ Goal 2           │  │  ✓ Completed task 2         │  │
│  │  [+ Add Goal]       │  │  [+ Add Accomplishment]     │  │
│  └─────────────────────┘  └─────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Recommendations & Notifications                      │  │
│  │  ┌─────────────────────────────────────────────────┐ │  │
│  │  │ 🔔 Team Notification: New comment on "Project"  │ │  │
│  │  │ 💡 Recommendation: Review "Machine Learning"    │ │  │
│  │  │ 📚 Suggested: "Deep Learning" by Goodfellow     │ │  │
│  │  └─────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Components

#### 1. Mood Check-In Card
- **Visual**: Emoji selector (😊 😐 😢 😤 😴 😎)
- **Interaction**: Click to select mood, auto-saves to daily note
- **Animation**: Subtle bounce on selection
- **State**: Shows current mood if already set

#### 2. Inspirational Quote Card
- **Content**: Fetched based on user interests (from preferences/tags)
- **Source**: Local quote database or API (e.g., Quotable API)
- **Personalization**: Filter by user's top tags/interests
- **Refresh**: Manual refresh button

#### 3. Quick Journal Entry
- **Editor**: TipTap rich text editor (already in stack)
- **Modes**:
  - **Text Mode**: Rich text with formatting toolbar
  - **Doodle Mode**: Excalidraw canvas (see below)
  - **Voice Mode**: Audio recording with transcription
- **Auto-Save**: Drafts saved to localStorage, final save to DailyNote
- **Linking**: Support for wikilinks (`[[Note Title]]`)

#### 4. Doodle Pad (Excalidraw Integration)
- **Component**: Embedded Excalidraw canvas
- **Storage**: Saved as `EditorState` with `contentType: 'excalidraw'`
- **Linking**: Linked to DailyNote via `strandId`
- **Export**: Can export as PNG/SVG
- **UI**: Toggle button to switch between text and doodle mode

#### 5. Goals & Accomplishments Cards
- **Goals**: Checklist for today's goals
- **Accomplishments**: List of completed tasks/achievements
- **Storage**: Stored in DailyNote metadata
- **Interaction**: Drag to reorder, click to toggle complete

#### 6. Recommendations & Notifications Feed
- **Sections**:
  - **Team Notifications**: Comments, mentions, updates
  - **Personal Recommendations**: Based on learning progress, interests
  - **System Notifications**: Reminders, scheduled reviews
- **Filtering**: Toggle between team/personal/all
- **Privacy**: User can configure visibility (local, team-visible, admin-visible)

### Styling Guidelines

- **Color Palette**: Warm, inviting colors (soft yellows, greens, blues)
- **Typography**: Readable, friendly fonts (Inter, Poppins)
- **Spacing**: Generous padding, clear visual hierarchy
- **Animations**: Subtle, delightful micro-interactions
- **Responsive**: Mobile-first, adapts to tablet/desktop
- **Dark Mode**: Full support with adjusted color palette

### Accessibility

- **Keyboard Navigation**: All interactions accessible via keyboard
- **Screen Reader**: Proper ARIA labels and roles
- **Color Contrast**: WCAG AA compliant
- **Focus Indicators**: Clear, visible focus states

---

## Records/Analytics Section

### Purpose

The Records section provides insights into the user's journaling habits, mood trends, and accomplishments over time.

### Layout Structure

```
┌─────────────────────────────────────────────────────────────┐
│  Records & Analytics                                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  [Date Range Picker: Last 7 Days ▼] [Export]                │
│                                                               │
│  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │  Streak             │  │  Total Entries              │  │
│  │  🔥 14 days         │  │  📝 42 notes                │  │
│  └─────────────────────┘  └─────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Mood Trends (Line Chart)                            │  │
│  │  [Recharts Line Chart showing mood over time]        │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Activity Heatmap (Calendar View)                     │  │
│  │  [Nivo Calendar showing daily activity intensity]     │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │  Accomplishments Timeline (Vertical Timeline)         │  │
│  │  [react-chrono showing key accomplishments]           │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                               │
│  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │  Top Tags           │  │  Mood Distribution          │  │
│  │  #work (24)         │  │  [Pie Chart]                │  │
│  │  #learning (18)     │  │  😊 40% 😐 30% 😢 20%       │  │
│  └─────────────────────┘  └─────────────────────────────┘  │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Components

#### 1. KPI Cards (Tremor)
- **Streak**: Current journaling streak (days)
- **Total Entries**: Total daily notes
- **Avg Mood**: Average mood score
- **Completion Rate**: Goals completed vs. set

#### 2. Mood Trends Chart (Recharts)
- **Type**: Line chart
- **X-Axis**: Date
- **Y-Axis**: Mood score (1-5)
- **Interaction**: Hover to see details, click to jump to daily note

#### 3. Activity Heatmap (@nivo/calendar)
- **Type**: Calendar heatmap
- **Color Scale**: Light to dark based on activity intensity
- **Tooltip**: Shows note count, mood, accomplishments

#### 4. Accomplishments Timeline (react-chrono)
- **Type**: Vertical timeline
- **Items**: Key accomplishments with dates
- **Interaction**: Click to expand details, link to related strands

#### 5. Top Tags & Mood Distribution
- **Top Tags**: Bar chart or list with counts
- **Mood Distribution**: Pie or donut chart

### Privacy & Visibility Settings

Users can configure what data is visible to:
- **Local Only**: Never shared, stored locally
- **Team Visible**: Shared with team members (configurable)
- **Admin Visible**: Visible to team admins

Settings stored in `User.preferences`:
```json
{
  "journal": {
    "shareActivity": "local" | "team" | "admin",
    "shareMood": false,
    "shareAccomplishments": true
  }
}
```

---

## Doodle Integration (Excalidraw)

### Implementation

1. **Component**: `DoodlePad.tsx`
   - Wraps Excalidraw component
   - Handles save/load from EditorState
   - Supports export to PNG/SVG

2. **Storage**:
   - Saved as `EditorState` with `contentType: 'excalidraw'`
   - Linked to DailyNote via `strandId`
   - Versioned via `ContentVersion`

3. **UI**:
   - Toggle button in journal entry toolbar
   - Full-screen mode available
   - Auto-save every 30 seconds

4. **Features**:
   - All Excalidraw tools (pen, shapes, text, etc.)
   - Export to image
   - Embed in daily note
   - Link to other strands

### Example Component Structure

```tsx
<DoodlePad
  dailyNoteId={dailyNote.id}
  onSave={(data) => saveToEditorState(data)}
  autoSave={true}
  fullScreen={false}
/>
```

---

## Technical Implementation Notes

### API Endpoints

- `GET /api/v1/journal/weave` - Get journal weave
- `POST /api/v1/journal/daily-notes` - Create daily note
- `GET /api/v1/journal/daily-notes/:date` - Get daily note
- `PUT /api/v1/journal/daily-notes/:date` - Update daily note
- `GET /api/v1/journal/analytics/mood` - Get mood analytics
- `GET /api/v1/journal/analytics/accomplishments` - Get accomplishments

### State Management (Zustand)

```ts
interface JournalStore {
  currentDailyNote: DailyNote | null;
  mood: string | null;
  goals: Goal[];
  accomplishments: Accomplishment[];
  setMood: (mood: string) => void;
  addGoal: (goal: Goal) => void;
  toggleGoal: (goalId: string) => void;
  addAccomplishment: (accomplishment: Accomplishment) => void;
}
```

### TanStack Query Hooks

```ts
useQuery(['dailyNote', date], () => fetchDailyNote(date));
useMutation(createDailyNote);
useMutation(updateDailyNote);
useQuery(['moodAnalytics', dateRange], () => fetchMoodAnalytics(dateRange));
```

---

## Inspirational Quotes System

### Quote Database

- **Local JSON**: Pre-loaded quotes categorized by topic
- **API Integration**: Quotable API or similar
- **Personalization**: Filter by user's top tags/interests

### Example Quote Structure

```json
{
  "quote": "The only way to do great work is to love what you do.",
  "author": "Steve Jobs",
  "tags": ["work", "passion", "motivation"]
}
```

### Matching Algorithm

1. Get user's top 5 tags from recent notes
2. Filter quotes by matching tags
3. Randomly select from filtered set
4. Cache for the day (one quote per day)

---

## Recommendations Feed

### Sources

1. **Learning Progress**: Strands due for review (spaced repetition)
2. **Related Content**: Strands similar to recently viewed
3. **Team Activity**: Recent team comments, updates
4. **System Reminders**: Scheduled tasks, deadlines

### Personalization

- **Interests**: Based on user's tags, note types
- **Activity**: Time of day, usage patterns
- **Preferences**: User-configured recommendation types

### Privacy

- **Local Recommendations**: Only user's own data
- **Team Recommendations**: Configurable by user & admin
- **Admin Recommendations**: System-wide announcements

---

## Mobile Considerations

- **Responsive Design**: Mobile-first approach
- **Touch Gestures**: Swipe to navigate, pinch to zoom (doodle)
- **Offline Support**: Service worker for offline journaling
- **Push Notifications**: Daily reminders (opt-in)

---

## Accessibility Checklist

- [ ] Keyboard navigation for all interactions
- [ ] Screen reader support (ARIA labels)
- [ ] Color contrast (WCAG AA)
- [ ] Focus indicators
- [ ] Alt text for images/doodles
- [ ] Captions for audio notes
- [ ] Resizable text
- [ ] High contrast mode

---

## Future Enhancements

- **Voice Journaling**: Full voice-to-text with emotion detection
- **AI Insights**: Sentiment analysis, topic extraction
- **Collaborative Journaling**: Shared team journals
- **Habit Tracking**: Integration with habit tracking systems
- **Export Options**: PDF, EPUB, print-ready formats
- **Integrations**: Calendar, task managers, fitness apps

---

## Summary

The Daily tab and Records section provide a comprehensive, elegant, and minimal UI for personal knowledge workflows. By leveraging modern React libraries (Recharts, react-chrono, Tremor) and integrating Excalidraw for doodling, we create a delightful, accessible, and powerful journaling experience.

