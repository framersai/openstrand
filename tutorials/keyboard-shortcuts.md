# Keyboard Shortcuts Guide

> Master OpenStrand's keyboard-first workflow for maximum productivity

## Overview

OpenStrand is designed for keyboard power users. Every feature can be accessed and every action triggered without touching your mouse. This guide covers all keyboard shortcuts and the powerful command palette.

---

## The Command Palette

### Opening the Command Palette

Press one of these shortcuts anywhere in the app:

- **`/`** - Quick access (works unless you're typing in a text field)
- **`Cmd+K`** (Mac) or **`Ctrl+K`** (Windows/Linux) - Universal shortcut

### Using the Command Palette

1. **Type to search** - Find commands, pages, data, or controls
2. **Arrow keys** (`↑` `↓`) - Navigate through results
3. **Enter** - Execute selected command
4. **Escape** - Close the palette

### What You Can Do

The command palette gives you instant access to:

- **Navigation** - Jump to any page or section
- **Actions** - Create strands, import data, export content
- **Data Search** - Find your strands, notes, visualizations
- **UI Controls** - Activate any button, toggle, or modal
- **Settings** - Change theme, language, preferences
- **Help** - Open docs, tutorials, support

### Command Categories

- **Recent** - Your last 5 commands for quick access
- **Navigation** - Go to different pages (shortcuts: `G D`, `G S`, `G T`, `G P`)
- **Actions** - Perform operations (shortcut: `C` for create)
- **Settings** - Configure preferences
- **Help** - Access documentation (shortcut: `?`)

---

## Global Shortcuts

### Navigation

| Shortcut | Action |
|----------|--------|
| `G` then `H` | Go to Home |
| `G` then `D` | Go to Dashboard |
| `G` then `S` | Go to Strands |
| `G` then `T` | Go to Tutorials |
| `G` then `P` | Go to Profile |

### Actions

| Shortcut | Action |
|----------|--------|
| `C` | Create new strand |
| `I` | Import data |
| `E` | Export data |
| `A` | Open AI assistant |
| `/` or `Cmd+K` | Open command palette |
| `?` | Show keyboard shortcuts help |

### Application

| Shortcut | Action |
|----------|--------|
| `T` | Toggle theme (light/dark) |
| `L` | Change language |
| `Escape` | Close modal/dialog |
| `Ctrl+,` or `Cmd+,` | Open settings |

---

## Context-Specific Shortcuts

### While Editing a Strand

| Shortcut | Action |
|----------|--------|
| `Cmd+S` or `Ctrl+S` | Save strand |
| `Cmd+B` or `Ctrl+B` | Bold text |
| `Cmd+I` or `Ctrl+I` | Italic text |
| `Cmd+K` or `Ctrl+K` | Insert link |
| `Cmd+Z` or `Ctrl+Z` | Undo |
| `Cmd+Shift+Z` or `Ctrl+Shift+Z` | Redo |

### In Tables and Lists

| Shortcut | Action |
|----------|--------|
| `↑` `↓` | Navigate rows |
| `Enter` | Open/select item |
| `Space` | Toggle checkbox |
| `Delete` or `Backspace` | Delete selected item |
| `Cmd+A` or `Ctrl+A` | Select all |

### In Modals and Dialogs

| Shortcut | Action |
|----------|--------|
| `Tab` | Move to next field |
| `Shift+Tab` | Move to previous field |
| `Enter` | Confirm/submit |
| `Escape` | Cancel/close |

---

## Search Features

### In Command Palette

The command palette uses fuzzy search. Try these queries:

- **"create"** - Shows all create actions
- **"strand"** - Shows strand-related commands
- **"theme"** - Shows theme settings
- **"tutorial"** - Navigates to tutorials
- **Your strand titles** - Jumps directly to strands
- **Any UI control** - Activates buttons, toggles, etc.

### Search Tips

- Type **partial words** - "dash" finds "Dashboard"
- Use **keywords** - "dark" finds theme toggle
- **Recent items** appear first
- **Arrow keys** to navigate results
- **Enter** to execute

---

## Accessibility Features

### Screen Reader Support

All keyboard shortcuts work seamlessly with screen readers:

- NVDA (Windows)
- JAWS (Windows)
- VoiceOver (Mac)
- TalkBack (Android)

### Focus Indicators

Every interactive element shows a visible focus ring when navigated with Tab or arrow keys.

### Skip Links

Press `Tab` on any page to reveal "Skip to main content" link.

---

## Customizing Shortcuts

### Adding Custom Commands

Developers can register custom commands for plugins or features:

```typescript
import { useCommandPalette, type CommandItem } from '@/components/shared/CommandPalette';

const customCommands: CommandItem[] = [
  {
    id: 'my-custom-action',
    title: 'My Custom Action',
    subtitle: 'Description of what it does',
    icon: Sparkles,
    action: () => myCustomAction(),
    category: 'actions',
    keywords: ['custom', 'special'],
    shortcut: 'M',
  },
];

function MyComponent() {
  const { CommandPaletteComponent } = useCommandPalette(customCommands);
  return <>{CommandPaletteComponent}</>;
}
```

### Registering Data for Search

Make your data searchable in the command palette:

```typescript
const strandCommands: CommandItem[] = strands.map(strand => ({
  id: `strand-${strand.id}`,
  title: strand.title,
  subtitle: strand.description,
  icon: FileText,
  action: () => router.push(`/strands/${strand.id}`),
  category: 'data',
  keywords: [strand.title, ...strand.tags],
}));

const { CommandPaletteComponent } = useCommandPalette(strandCommands);
```

---

## Pro Tips

### Power User Workflow

1. **Start with `/`** - Fastest way to open palette
2. **Type what you want** - "create strand", "dark theme", "profile"
3. **Hit Enter** - Execute immediately
4. **Repeat** - Your recent commands appear first

### Command Chaining

Navigate faster by chaining commands:

1. `Cmd+K` → "dashboard" → `Enter` - Jump to dashboard
2. `/` → "create" → `Enter` - Start creating
3. `G D` - Direct shortcut to dashboard

### Search Everything

The command palette searches:
- Page names and routes
- Action descriptions
- Your strand titles and content
- Settings and preferences
- UI control labels
- Help documentation

---

## Troubleshooting

### Command Palette Won't Open

- Check if another app is capturing `Cmd+K` (e.g., Spotlight)
- Try `/` as an alternative
- Refresh the page

### Shortcuts Not Working

- Ensure you're not typing in a text field
- Check if a modal is open (close with `Escape`)
- Verify JavaScript is enabled

### Custom Commands Not Appearing

- Ensure they're passed to `useCommandPalette` hook
- Check that `id` is unique
- Verify `category` matches a valid category

---

## Quick Reference Card

Press `?` in the app to see this quick reference:

```
Navigation     Actions           Settings
G H  Home      C  Create         T  Theme
G D  Dashboard I  Import         L  Language  
G S  Strands   E  Export         ?  Help
G T  Tutorials A  AI Assistant
G P  Profile   

Universal
/  or  Cmd+K    Open command palette
Escape           Close/Cancel
Tab              Next field
Shift+Tab        Previous field
```

---

## Learn More

- [UI Component Guide](../UI_COMPONENT_GUIDE.md) - Component documentation
- [Design System](../DESIGN_SYSTEM.md) - Design tokens and patterns
- [Accessibility Guide](#) - WCAG compliance details
- [Plugin Development](../../packages/openstrand-plugins/README.md) - Building plugins

---

**Last Updated:** November 14, 2024  
**Version:** 1.0

