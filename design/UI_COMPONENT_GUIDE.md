# OpenStrand UI Component Guide

## Overview

This guide documents our UI/UX patterns, component library, and best practices for building consistent, accessible, and delightful user experiences in OpenStrand.

## Design Principles

### 1. Accessibility First
- WCAG 2.1 AA compliance minimum
- Semantic HTML
- ARIA labels and roles
- Keyboard navigation
- Screen reader support

### 2. Progressive Disclosure
- Show essential information first
- Reveal complexity on demand
- Use collapsible sections
- Implement step-by-step workflows

### 3. Consistent Feedback
- Clear loading states (skeletons > spinners)
- Meaningful empty states
- Helpful error messages
- Success confirmations

### 4. Dark Mode Compatible
- All components support light/dark themes
- Use semantic color tokens
- Test in both modes
- Respect system preferences

### 5. Responsive Design
- Mobile-first approach
- Touch-friendly targets (44x44px minimum)
- Adaptive layouts
- Progressive enhancement

---

## Component Library

### 1. Command Palette

**Purpose:** Global keyboard-driven command system for navigation, search, and actions.

**Location:** `src/components/shared/CommandPalette.tsx`

**Features:**
- Universal keyboard access (/ or Cmd+K / Ctrl+K)
- Search commands, pages, data, UI controls
- Keyboard navigation (arrow keys, Enter, Escape)
- Recent commands tracking
- Custom command registration
- Tooltips and shortcuts display
- Full ARIA support
- Grouped by category

**Usage:**
```tsx
import { useCommandPalette, type CommandItem } from '@/components/shared/CommandPalette';

function MyApp() {
  // Define custom commands for your page/feature
  const customCommands: CommandItem[] = [
    {
      id: 'open-settings',
      title: 'Open Settings',
      subtitle: 'Configure your preferences',
      icon: Settings,
      action: () => setSettingsOpen(true),
      category: 'actions',
      keywords: ['settings', 'preferences', 'config'],
      shortcut: 'S',
    },
  ];

  const { CommandPaletteComponent } = useCommandPalette(customCommands);

  return (
    <>
      {/* Automatically opens with / or Cmd+K */}
      {CommandPaletteComponent}
      
      {/* Your page content */}
    </>
  );
}
```

**Keyboard Shortcuts:**
- `/` or `Cmd+K` (Ctrl+K) - Open palette
- `↑` `↓` - Navigate results
- `Enter` - Execute command
- `Escape` - Close palette
- `?` - Show shortcuts help

**Registering Data for Search:**
```tsx
// Make your strands searchable
const strandCommands = strands.map(strand => ({
  id: `strand-${strand.id}`,
  title: strand.title,
  subtitle: strand.description,
  icon: FileText,
  action: () => router.push(`/strands/${strand.id}`),
  category: 'data',
  keywords: [strand.title, ...strand.tags],
}));
```

**Registering UI Controls:**
```tsx
// Make buttons activatable from keyboard
const uiCommands: CommandItem[] = [
  {
    id: 'toggle-sidebar',
    title: 'Toggle Sidebar',
    icon: Menu,
    action: () => setSidebarOpen(prev => !prev),
    category: 'actions',
    shortcut: 'B',
  },
];
```

---

### 2. ProfileOnboardingWizard

**Purpose:** Guide new users through initial setup with a multi-step wizard.

**Location:** `src/components/profile/ProfileOnboardingWizard.tsx`

**Features:**
- 5-step onboarding flow
- Progress tracking with localStorage persistence
- Keyboard navigation (Arrow keys, Enter, Escape)
- Full ARIA support
- Responsive design

**Usage:**
```tsx
import { ProfileOnboardingWizard } from '@/components/profile/ProfileOnboardingWizard';

function ProfilePage() {
  const [showOnboarding, setShowOnboarding] = useState(true);

  return (
    <ProfileOnboardingWizard
      isOpen={showOnboarding}
      onComplete={() => {
        setShowOnboarding(false);
        // Save completion to backend
      }}
      onDismiss={() => setShowOnboarding(false)}
    />
  );
}
```

**Steps:**
1. **Goal Setting** - Understand user's primary use case
2. **Workspace Setup** - Configure storage and display name
3. **Feature Discovery** - Introduce key features
4. **Preferences** - Set notifications and auto-insights
5. **Success** - Show next steps and recommended actions

**Accessibility:**
- Full keyboard navigation
- ARIA labels and roles
- Screen reader announcements
- Focus management
- Skip functionality

---

### 2. SkeletonLoader

**Purpose:** Provide better loading UX with skeleton screens instead of spinners.

**Location:** `src/components/shared/SkeletonLoader.tsx`

**Features:**
- Multiple preset variants (card, list, table, profile, dashboard)
- Customizable layout
- Maintains layout stability
- Smooth animations
- ARIA support

**Usage:**
```tsx
import { SkeletonLoader } from '@/components/shared/SkeletonLoader';

// Card skeleton
<SkeletonLoader variant="card" count={3} />

// Table skeleton
<SkeletonLoader variant="table" rows={5} columns={4} />

// Profile skeleton
<SkeletonLoader variant="profile" />

// Dashboard skeleton
<SkeletonLoader variant="dashboard" />

// Custom skeleton
<SkeletonLoader variant="custom">
  <Skeleton className="h-8 w-48" />
  <Skeleton className="h-4 w-full" />
</SkeletonLoader>
```

**Best Practices:**
- Use skeletons instead of spinners for better perceived performance
- Match skeleton layout to actual content structure
- Show skeletons immediately (no delay)
- Replace skeletons with actual content atomically

---

### 3. EmptyState

**Purpose:** Display helpful empty states when there's no content.

**Location:** `src/components/shared/EmptyState.tsx`

**Features:**
- Multiple preset variants (empty, search, error, notFound, etc.)
- Customizable icon and illustration
- Primary and secondary actions
- Responsive design
- ARIA support

**Usage:**
```tsx
import { EmptyState, NoSearchResults, NoDataYet, ErrorState } from '@/components/shared/EmptyState';

// Generic empty state
<EmptyState
  variant="noData"
  title="No strands yet"
  description="Get started by creating your first strand"
  action={{
    label: "Create strand",
    href: "/pkms/strands/new",
    icon: Plus
  }}
  secondaryAction={{
    label: "Import data",
    onClick: () => openImport(),
    icon: Upload
  }}
/>

// Preset components
<NoSearchResults query="test" onClear={() => clearSearch()} />
<NoDataYet resource="Strands" onCreate={() => create()} />
<ErrorState onRetry={() => retry()} />
```

**Best Practices:**
- Always provide at least one action
- Use clear, action-oriented language
- Provide context about why it's empty
- Make it easy to get out of the empty state

---

### 4. TutorialSearch

**Purpose:** Comprehensive search and filtering for tutorials.

**Location:** `src/components/tutorials/TutorialSearch.tsx`

**Features:**
- Real-time search
- Multiple filter dimensions (difficulty, duration, category, tags)
- Sort options
- Active filter badges
- Keyboard accessible

**Usage:**
```tsx
import { TutorialSearch, type TutorialFilters } from '@/components/tutorials/TutorialSearch';

function TutorialsPage() {
  const [filters, setFilters] = useState<TutorialFilters>({
    query: '',
    difficulty: 'all',
    duration: 'all',
    categories: [],
    tags: [],
    sort: 'popular',
  });

  return (
    <TutorialSearch
      filters={filters}
      onFiltersChange={setFilters}
      categories={['Setup', 'Workflows', 'API']}
      tags={['ai', 'collaboration', 'analytics']}
    />
  );
}
```

**Filter Types:**
- **Query**: Free-text search
- **Difficulty**: Beginner, Intermediate, Advanced
- **Duration**: Quick (<5min), Medium (5-15min), Long (>15min)
- **Categories**: Predefined categories
- **Tags**: Free-form tags
- **Sort**: Popular, Newest, Duration, A-Z

---

### 5. LearningPath

**Purpose:** Structured learning paths with progress tracking.

**Location:** `src/components/tutorials/LearningPath.tsx`

**Features:**
- Visual progress indication
- Prerequisites and unlocking logic
- Time estimates
- Completion tracking
- Achievement badges
- Expandable tutorial details

**Usage:**
```tsx
import { LearningPath, type LearningPathData } from '@/components/tutorials/LearningPath';

const path: LearningPathData = {
  id: 'getting-started',
  title: 'Getting Started',
  description: 'Learn the basics in 30 minutes',
  difficulty: 'beginner',
  totalDuration: 30,
  tutorials: [
    {
      id: 'intro',
      title: 'Introduction',
      description: 'Overview of OpenStrand',
      href: '/tutorials/intro',
      duration: 5,
    },
    {
      id: 'setup',
      title: 'Setup',
      description: 'Configure your workspace',
      href: '/tutorials/setup',
      duration: 10,
      prerequisites: ['intro'], // Locked until intro is complete
    },
  ],
  badge: {
    title: 'OpenStrand Beginner',
  },
};

<LearningPath
  path={path}
  completedTutorials={['intro']}
  onCompleteTutorial={(id) => markComplete(id)}
/>
```

**Features:**
- Automatic prerequisite checking
- Progress percentage calculation
- Badge rewards for completion
- Estimated time remaining
- Expandable tutorial cards

---

## Pattern Library

### Loading States

**Principle:** Show skeleton screens that match the content structure.

```tsx
// ❌ DON'T: Generic spinner
<Spinner />

// ✅ DO: Skeleton matching content
<SkeletonLoader variant="card" count={3} />
```

**Implementation:**
1. Identify the content type (card, list, table, etc.)
2. Use matching skeleton variant
3. Show immediately (no delay)
4. Replace atomically when data loads

### Empty States

**Principle:** Guide users to take action when there's no content.

```tsx
// ❌ DON'T: Minimal message
<div>No items found</div>

// ✅ DO: Helpful empty state with action
<EmptyState
  title="No strands yet"
  description="Get started by creating your first strand"
  action={{ label: "Create strand", onClick: create }}
/>
```

**Components:**
- Title: Clear, concise (max 6 words)
- Description: Explain why it's empty and what to do
- Icon: Visual reinforcement
- Action: Primary CTA to exit empty state
- Secondary action: Alternative path (optional)

### Form Validation

**Principle:** Provide real-time, helpful feedback.

```tsx
// ❌ DON'T: Only show errors on submit
<Input name="email" />

// ✅ DO: Real-time validation with helpful messages
<Input
  name="email"
  error={errors.email}
  helperText="We'll never share your email"
  aria-invalid={!!errors.email}
  aria-describedby="email-error"
/>
```

**Best Practices:**
- Validate on blur, not on every keystroke
- Show success states (green checkmark)
- Provide specific error messages
- Include helper text for context
- Disable submit until valid

### Error Handling

**Principle:** Friendly, actionable error messages.

```tsx
// ❌ DON'T: Technical error messages
<div>Error: ECONNREFUSED</div>

// ✅ DO: User-friendly with retry
<ErrorState
  title="Connection failed"
  description="We couldn't connect to the server. Please check your internet connection and try again."
  onRetry={() => retry()}
/>
```

**Error Types:**
- **Network errors**: Explain and offer retry
- **Validation errors**: Show inline with fields
- **Permission errors**: Explain and suggest solution
- **Not found**: Helpful 404 with navigation
- **Unexpected errors**: Apologize and offer support

### Modal Dialogs

**Principle:** Clear purpose, easy to dismiss.

**Structure:**
```tsx
<Dialog>
  <DialogHeader>
    <DialogTitle>Clear title</DialogTitle>
    <DialogDescription>Explain the purpose</DialogDescription>
  </DialogHeader>
  <DialogContent>
    {/* Content */}
  </DialogContent>
  <DialogFooter>
    <Button variant="outline" onClick={onCancel}>Cancel</Button>
    <Button onClick={onConfirm}>Confirm</Button>
  </DialogFooter>
</Dialog>
```

**Best Practices:**
- Clear title and description
- Easy to dismiss (X button, Escape, click outside)
- Primary action on the right
- Destructive actions require confirmation
- Focus trap for accessibility
- Return focus on close

### Notifications (Toasts)

**Principle:** Timely, non-intrusive feedback.

```tsx
// Success
toast.success('Strand created successfully');

// Error with action
toast.error('Failed to save', {
  action: { label: 'Retry', onClick: () => retry() }
});

// Info with duration
toast.info('Changes auto-saved', { duration: 2000 });
```

**Best Practices:**
- Auto-dismiss after 3-5 seconds
- Allow manual dismiss
- Include action for errors
- Position consistently (usually bottom-right)
- Don't show multiple of the same type
- Use appropriate icon and color

---

## Accessibility Checklist

### Keyboard Navigation
- [ ] All interactive elements are keyboard accessible
- [ ] Tab order is logical
- [ ] Focus indicators are visible
- [ ] Shortcuts don't conflict with browser/screen reader
- [ ] Escape closes modals/dropdowns
- [ ] Enter/Space activates buttons

### Screen Readers
- [ ] Semantic HTML (button, nav, main, etc.)
- [ ] ARIA labels for icon-only buttons
- [ ] ARIA live regions for dynamic content
- [ ] Form labels are associated with inputs
- [ ] Error messages are announced
- [ ] Loading states are announced

### Visual
- [ ] Color contrast meets WCAG AA (4.5:1 for text)
- [ ] Interactive elements are 44x44px minimum
- [ ] Text is resizable to 200%
- [ ] No information conveyed by color alone
- [ ] Focus indicators are visible
- [ ] Animations respect prefers-reduced-motion

### Testing
- [ ] Test with keyboard only
- [ ] Test with screen reader (NVDA, JAWS, VoiceOver)
- [ ] Run axe DevTools
- [ ] Check with Lighthouse
- [ ] Test in high contrast mode
- [ ] Test with zoom at 200%

---

## Theming

### Color System

**Semantic Colors:**
```css
/* Light mode */
--background: 0 0% 100%;
--foreground: 222.2 84% 4.9%;
--primary: 221.2 83.2% 53.3%;
--primary-foreground: 210 40% 98%;
--muted: 210 40% 96.1%;
--muted-foreground: 215.4 16.3% 46.9%;

/* Dark mode */
--background: 222.2 84% 4.9%;
--foreground: 210 40% 98%;
--primary: 217.2 91.2% 59.8%;
--primary-foreground: 222.2 47.4% 11.2%;
--muted: 217.2 32.6% 17.5%;
--muted-foreground: 215 20.2% 65.1%;
```

**Usage:**
```tsx
// ✅ DO: Use semantic tokens
<div className="bg-background text-foreground" />
<Button className="bg-primary text-primary-foreground" />

// ❌ DON'T: Use hardcoded colors
<div className="bg-white text-black" />
<div className="bg-blue-500" />
```

### Responsive Breakpoints

```css
/* Tailwind breakpoints */
sm: 640px   /* Small tablets */
md: 768px   /* Tablets */
lg: 1024px  /* Small desktops */
xl: 1280px  /* Desktops */
2xl: 1536px /* Large desktops */
```

**Usage:**
```tsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3">
  {/* Responsive grid */}
</div>
```

---

## Performance Best Practices

### Code Splitting

```tsx
// Lazy load heavy components
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <SkeletonLoader variant="custom">
    <Skeleton className="h-64 w-full" />
  </SkeletonLoader>,
  ssr: false,
});
```

### Image Optimization

```tsx
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Hero image"
  width={1200}
  height={600}
  priority // For above-fold images
  placeholder="blur"
/>
```

### Virtual Scrolling

```tsx
// For long lists (>100 items)
import { useVirtualizer } from '@tanstack/react-virtual';

function LongList({ items }) {
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      {virtualizer.getVirtualItems().map((virtualRow) => (
        <div key={virtualRow.index}>{items[virtualRow.index]}</div>
      ))}
    </div>
  );
}
```

---

## Component Checklist

When creating a new component, ensure:

- [ ] TypeScript interfaces for all props
- [ ] TSDoc comments with examples
- [ ] Supports dark mode
- [ ] Keyboard accessible
- [ ] ARIA labels and roles
- [ ] Responsive design
- [ ] Loading state (if async)
- [ ] Empty state (if applicable)
- [ ] Error state (if applicable)
- [ ] Proper semantic HTML
- [ ] Focus management
- [ ] Unit tests (if complex logic)

---

## Resources

- [Radix UI](https://www.radix-ui.com/) - Accessible component primitives
- [shadcn/ui](https://ui.shadcn.com/) - Component library base
- [Tailwind CSS](https://tailwindcss.com/) - Utility-first CSS
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Inclusive Components](https://inclusive-components.design/)
- [A11y Project Checklist](https://www.a11yproject.com/checklist/)

---

Last Updated: November 2024

