# OpenStrand Design System Documentation

## Table of Contents

1. [Overview](#overview)
2. [Design Principles](#design-principles)
3. [Mobile-First Approach](#mobile-first-approach)
4. [Theme System](#theme-system)
5. [Typography](#typography)
6. [Color System](#color-system)
7. [Spacing & Layout](#spacing--layout)
8. [Components](#components)
9. [Responsive Design](#responsive-design)
10. [Accessibility](#accessibility)
11. [Performance](#performance)
12. [SCSS Architecture](#scss-architecture)
13. [Implementation Guide](#implementation-guide)

---

## Overview

The OpenStrand/OpenStrand design system is a comprehensive, mobile-first styling framework built with SCSS and Tailwind CSS integration. It provides multiple theme variations, robust responsive utilities, and a component-based architecture optimized for data visualization applications.

### Key Features

- **Mobile-First Responsive Design**: Built from the ground up for mobile devices
- **Multi-Theme Support**: 6 unique themes with light/dark mode variations
- **SCSS Architecture**: Modular, maintainable styling with advanced features
- **Tailwind Integration**: Seamless integration with Tailwind utilities
- **Accessibility First**: WCAG 2.1 AA compliant components
- **Performance Optimized**: Minimal CSS output with tree-shaking support

---

## Design Principles

### 1. Mobile-First

All components and layouts are designed for mobile devices first, then enhanced for larger screens. This ensures optimal performance and user experience on all devices.

### 2. Consistency

Unified design tokens ensure consistent spacing, typography, and colors across all themes and components.

### 3. Accessibility

Every component is built with accessibility in mind, including proper contrast ratios, focus indicators, and screen reader support.

### 4. Performance

Optimized for fast load times with minimal CSS, efficient animations, and GPU-accelerated transitions.

### 5. Flexibility

Modular architecture allows for easy customization and extension of existing components.

---

## Mobile-First Approach

### Breakpoints

```scss
$breakpoints: (
  'xs': 320px,    // Small phones
  'sm': 480px,    // Phones
  'md': 768px,    // Tablets
  'lg': 1024px,   // Desktop
  'xl': 1280px,   // Large desktop
  '2xl': 1536px,  // Extra large desktop
  '3xl': 1920px   // Ultra wide
);
```
:memo: **Implementation snapshot (Oct 2025)** - `_container.scss`, `_grid.scss`, `_header.scss`, `_footer.scss`, `_sidebar.scss`, and `utilities/_spacing.scss` ship the container grid, chrome, and stack utilities described above. They are imported via `src/styles/main.scss` and consumed automatically through `app/globals.scss`, so product pages now rely on these shared primitives instead of placeholder comments.

### Usage Example

```scss
// Mobile styles (default)
.component {
  padding: 1rem;
  font-size: 14px;

  // Tablet and up
  @include min-width('md') {
    padding: 1.5rem;
    font-size: 16px;
  }

  // Desktop and up
  @include min-width('lg') {
    padding: 2rem;
    font-size: 18px;
  }
}
```

### Touch Targets

All interactive elements maintain a minimum touch target of 44px (iOS) / 48px (Android):

```scss
$touch-target-min: 44px;
$touch-target-comfortable: 48px;
```

---

## Theme System

### Available Themes

1. **Modern** - Clean, minimalist design with gradients and smooth shadows
2. **Classic** - Professional, traditional design with navy and gold accents
3. **Vintage** - Retro design with warm colors and nostalgic elements
4. **Paper** - Clean, document-like aesthetic inspired by print
5. **Cyberpunk** - Futuristic neon design with glowing effects
6. **Minimal** - Ultra-clean, whitespace-focused design

### Theme Structure

Each theme defines:

- **Core Colors**: Primary, Secondary, Tertiary, Accent
- **Backgrounds**: Primary, Secondary, Tertiary, Surface
- **Text Colors**: Primary, Secondary, Muted, Inverse
- **UI Elements**: Borders, Shadows, Radius values
- **Semantic Colors**: Success, Warning, Error, Info
- **Typography**: Font families for base, heading, and mono text

### Applying Themes

Themes are applied using data attributes:

```html
<!-- Light mode -->
<html data-theme="modern-light">

<!-- Dark mode -->
<html data-theme="modern-light" class="dark">
```

### Theme Switching (React)

```tsx
const ThemeSelector = () => {
  const themes = [
    'modern-light',
    'classic-light',
    'vintage-light',
    'paper-light',
    'cyberpunk-light',
    'minimal-light'
  ];

  const setTheme = (theme: string) => {
    document.documentElement.setAttribute('data-theme', theme);
  };

  return (
    <select onChange={(e) => setTheme(e.target.value)}>
      {themes.map(theme => (
        <option key={theme} value={theme}>{theme}</option>
      ))}
    </select>
  );
};
```

---

## Typography

### Font Stack

```scss
// Base fonts by theme
--font-base: 'Inter', system-ui, -apple-system, sans-serif;
--font-heading: 'Inter', system-ui, -apple-system, sans-serif;
--font-mono: 'Fira Code', 'Courier New', monospace;
```

### Type Scale

Mobile-first responsive type scale using Major Third ratio (1.25):

```scss
$font-sizes: (
  'xs': 12px,    // 0.75rem
  'sm': 14px,    // 0.875rem
  'base': 16px,  // 1rem
  'lg': 18px,    // 1.125rem
  'xl': 20px,    // 1.25rem
  '2xl': 24px,   // 1.5rem
  '3xl': 30px,   // 1.875rem
  '4xl': 36px,   // 2.25rem
  '5xl': 48px,   // 3rem
  '6xl': 60px,   // 3.75rem
);
```

### Responsive Typography

```scss
// Heading sizes adjust based on screen size
h1 {
  font-size: 30px; // Mobile

  @include min-width('md') {
    font-size: 36px; // Tablet
  }

  @include min-width('lg') {
    font-size: 48px; // Desktop
  }
}
```

### Font Weights

```scss
$font-weights: (
  'thin': 100,
  'light': 300,
  'normal': 400,
  'medium': 500,
  'semibold': 600,
  'bold': 700,
  'extrabold': 800,
  'black': 900,
);
```

---

## Color System

### CSS Custom Properties

Each theme defines colors as CSS custom properties:

```css
:root {
  /* Core colors */
  --color-primary: #3b82f6;
  --color-primary-light: #60a5fa;
  --color-primary-dark: #2563eb;

  /* Backgrounds */
  --bg-primary: #ffffff;
  --bg-secondary: #f8fafc;

  /* Text */
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --text-muted: #94a3b8;

  /* Semantic */
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;
  --color-info: #3b82f6;
}
```

### Chart Colors

Predefined chart color palette:

```scss
--chart-color-1: var(--color-primary);
--chart-color-2: var(--color-secondary);
--chart-color-3: var(--color-accent);
--chart-color-4: var(--color-tertiary);
--chart-color-5: var(--color-primary-light);
--chart-color-6: var(--color-secondary-light);
```

---

## Spacing & Layout

### Spacing Scale (8px base unit)

```scss
$spacing: (
  '0': 0,
  '1': 2px,
  '2': 4px,
  '3': 6px,
  '4': 8px,
  '6': 12px,
  '8': 16px,
  '10': 20px,
  '12': 24px,
  '16': 32px,
  '20': 40px,
  '24': 48px,
  '32': 64px,
);
```

### Container System

```scss
$container-max-widths: (
  'sm': 640px,
  'md': 768px,
  'lg': 1024px,
  'xl': 1280px,
  '2xl': 1536px,
);
```

### Grid System

12-column grid with responsive utilities:

```scss
@mixin grid($columns: 12, $gap: 8px) {
  display: grid;
  grid-template-columns: repeat($columns, 1fr);
  gap: $gap;
}
```

---

## Components

### Button Styles

Each theme provides unique button styles:

```scss
// Modern theme
.btn-gradient-border
.btn-cyber

// Classic theme
.btn-classic

// Vintage theme
.btn-retro

// Paper theme
.btn-paper

// Minimal theme
.btn-minimal
```

### Card Variants

```scss
// Modern
.card-glass    // Glassmorphism effect

// Classic
.card-classic  // Traditional card with shadow

// Vintage
.card-vintage  // Torn edges effect

// Paper
.document      // Document-like layout

// Minimal
.card-minimal  // Borderless card
```

### Form Elements

Mobile-optimized form controls:

```scss
.input-minimal   // Minimal underline style
.input-paper     // Paper theme input
.input-cyber     // Cyberpunk glowing input
```

---

## Responsive Design

### Responsive Utilities

```scss
// Display utilities
.hidden-mobile    // Hidden on mobile
.hidden-tablet    // Hidden on tablet
.hidden-desktop   // Hidden on desktop

// Text alignment
.text-center
.text-md-left    // Left aligned on tablet+
.text-lg-right   // Right aligned on desktop+

// Spacing
.p-4             // Padding all sides
.p-md-6          // Larger padding on tablet+
.p-lg-8          // Even larger on desktop+
```

### Responsive Images

```scss
img {
  max-width: 100%;
  height: auto;
  object-fit: cover;
}
```

### Responsive Tables

Tables scroll horizontally on mobile:

```scss
.table-wrapper {
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;
}
```

---

## Accessibility

### Focus Management

All interactive elements have visible focus indicators:

```scss
:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

### Color Contrast

All color combinations meet WCAG 2.1 AA standards:

- Normal text: 4.5:1 contrast ratio
- Large text: 3:1 contrast ratio
- UI components: 3:1 contrast ratio

### Screen Reader Support

```scss
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
```

### Reduced Motion

Respects user's motion preferences:

```scss
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## Performance

### Optimization Techniques

1. **CSS Custom Properties**: Dynamic theming without duplicated styles
2. **Tree Shaking**: Unused styles are automatically removed
3. **GPU Acceleration**: Transform and opacity animations
4. **Critical CSS**: Above-the-fold styles loaded first
5. **Lazy Loading**: Theme-specific styles loaded on demand

### Animation Performance

```scss
// Use GPU-accelerated properties
.animated {
  transform: translateX(0);
  opacity: 1;
  will-change: transform, opacity;
}
```

### Font Loading Strategy

```scss
// Progressive font loading
@font-face {
  font-family: 'Inter';
  font-display: swap; // Show fallback immediately
}
```

---

## SCSS Architecture

### File Structure

```
src/styles/
├── abstracts/
│   ├── _variables.scss     // Design tokens
│   ├── _functions.scss     // Helper functions
│   ├── _mixins.scss        // Reusable mixins
│   └── _breakpoints.scss   // Responsive utilities
├── base/
│   ├── _reset.scss         // CSS reset
│   ├── _typography.scss    // Typography styles
│   └── _animations.scss    // Animation keyframes
├── components/
│   ├── _buttons.scss       // Button components
│   ├── _cards.scss         // Card components
│   ├── _forms.scss         // Form elements
│   ├── _charts.scss        // Chart styles
│   └── _tables.scss        // Table styles
├── layout/
│   ├── _grid.scss          // Grid system
│   ├── _container.scss     // Container layouts
│   ├── _header.scss        // Header styles
│   └── _footer.scss        // Footer styles
├── themes/
│   ├── _theme-engine.scss  // Theme system
│   ├── _modern.scss        // Modern theme
│   ├── _classic.scss       // Classic theme
│   ├── _vintage.scss       // Vintage theme
│   ├── _paper.scss         // Paper theme
│   ├── _cyberpunk.scss     // Cyberpunk theme
│   └── _minimal.scss       // Minimal theme
├── utilities/
│   ├── _spacing.scss       // Spacing utilities
│   ├── _colors.scss        // Color utilities
│   └── _responsive.scss    // Responsive utilities
└── main.scss               // Main entry point
```

---

## Implementation Guide

### 1. Setup SCSS in Next.js

```bash
npm install sass --save-dev
```

### 2. Import Main SCSS File

In `app/layout.tsx`:

```tsx
import '@/styles/main.scss';
```

### 3. Configure Theme Provider

```tsx
import { ThemeProvider } from '@/components/theme-provider';

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        <ThemeProvider
          attribute="data-theme"
          defaultTheme="modern-light"
          enableSystem
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 4. Use Theme-Aware Components

```tsx
const Button = ({ variant = 'primary', children }) => {
  return (
    <button className={`btn btn-${variant}`}>
      {children}
    </button>
  );
};
```

### 5. Responsive Component Example

```tsx
const ResponsiveCard = ({ children }) => {
  return (
    <div className="card-wrapper p-4 p-md-6 p-lg-8">
      <div className="card-content text-center text-md-left">
        {children}
      </div>
    </div>
  );
};
```

---

## Best Practices

### 1. Mobile-First Development

Always start with mobile styles and enhance for larger screens:

```scss
.component {
  // Mobile styles (default)
  padding: 1rem;

  // Tablet enhancement
  @include min-width('md') {
    padding: 1.5rem;
  }
}
```

### 2. Use CSS Custom Properties

Leverage CSS custom properties for dynamic theming:

```scss
.component {
  background: var(--bg-surface);
  color: var(--text-primary);
}
```

### 3. Semantic Class Names

Use descriptive, semantic class names:

```scss
// Good
.navigation-menu
.data-table-header
.chart-container

// Avoid
.nav
.header
.container
```

### 4. Component Composition

Build complex components from simple ones:

```tsx
const DataCard = ({ data }) => (
  <Card>
    <CardHeader>
      <CardTitle>{data.title}</CardTitle>
    </CardHeader>
    <CardBody>
      <Chart data={data.values} />
    </CardBody>
  </Card>
);
```

### 5. Performance Optimization

- Use `transform` and `opacity` for animations
- Lazy load theme-specific styles
- Minimize CSS specificity
- Use CSS containment where appropriate

---

## Migration Guide

### From CSS to SCSS

1. Rename `.css` files to `.scss`
2. Update imports in JavaScript/TypeScript files
3. Refactor styles to use SCSS features:
   - Variables -> SCSS variables or CSS custom properties
   - Nested selectors
   - Mixins for repeated patterns
   - Functions for calculations

### Adding New Themes

1. Create new theme file in `themes/` directory
2. Use `define-theme` mixin with color palette
3. Add theme-specific styles
4. Import in `main.scss`
5. Add to theme selector component

---

## Resources

- [SCSS Documentation](https://sass-lang.com/documentation)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [Mobile First Design](https://www.lukew.com/ff/entry.asp?933)
- [CSS Custom Properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

---

## Support

For questions or issues with the design system:

- Review the [component examples](./examples/)
- Check the [theme playground](./playground/)
- Open an issue on GitHub
- Contact: team@frame.dev


## Placeholder & Visibility Guidelines

- **Purpose**: Placeholders communicate that a strand exists but is hidden due to permissions or pending approvals.
- **Visual treatment**: muted surface, dashed border, customizable icon/text from placeholder preferences, and lock/state badge.
- **Interaction**: non-interactive for viewers without `structure` permission; approvers see inline action buttons that open the approval drawer.
- **Responsive layout**: On mobile, group placeholders into collapsible stacks with counts; on desktop, show inline cards within the structure tree.
- **Assistive tech**: Use `aria-label="Placeholder strand"` and explain the reason via `aria-describedby`. Ensure contrast ratios ≥ 4.5:1.
- **Customization**: Global defaults managed via `/meta/placeholders`, with scope-level overrides for teams/projects.



