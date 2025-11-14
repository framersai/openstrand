# OpenStrand Keyboard Shortcuts Reference

> Complete guide to keyboard-first workflows in OpenStrand

## Quick Reference

Press `?` anywhere in the app to see this quick reference card, or press `/` or `Cmd+K` to open the command palette and search for anything.

---

## Universal Shortcuts

| Shortcut | Action | Description |
|----------|--------|-------------|
| `/` | Open Command Palette | Quick access to search |
| `Cmd+K` or `Ctrl+K` | Open Command Palette | Universal shortcut |
| `Escape` | Close/Cancel | Close modals, dialogs, palette |
| `?` | Show Shortcuts Help | Display this reference |
| `Tab` | Next Field/Element | Navigate forward |
| `Shift+Tab` | Previous Field/Element | Navigate backward |

---

## Navigation Shortcuts

All navigation shortcuts start with `G` (for "Go"):

| Shortcut | Action |
|----------|--------|
| `G` then `H` | Go to Home |
| `G` then `D` | Go to Dashboard |
| `G` then `S` | Go to Strands |
| `G` then `T` | Go to Tutorials |
| `G` then `P` | Go to Profile |
| `G` then `W` | Go to Weave (Knowledge Graph) |

---

## Action Shortcuts

| Shortcut | Action |
|----------|--------|
| `C` | Create new strand |
| `I` | Import data |
| `E` | Export data |
| `A` | Open AI assistant |
| `T` | Toggle theme |
| `L` | Change language |

---

## Editing Shortcuts

### Text Formatting

| Shortcut | Action |
|----------|--------|
| `Cmd+B` or `Ctrl+B` | **Bold** text |
| `Cmd+I` or `Ctrl+I` | *Italic* text |
| `Cmd+U` or `Ctrl+U` | <u>Underline</u> text |
| `Cmd+K` or `Ctrl+K` | Insert link |
| `Cmd+Shift+X` or `Ctrl+Shift+X` | Strikethrough |

### Undo/Redo

| Shortcut | Action |
|----------|--------|
| `Cmd+Z` or `Ctrl+Z` | Undo |
| `Cmd+Shift+Z` or `Ctrl+Shift+Z` | Redo |
| `Cmd+Y` or `Ctrl+Y` | Redo (alternative) |

### Document Operations

| Shortcut | Action |
|----------|--------|
| `Cmd+S` or `Ctrl+S` | Save strand |
| `Cmd+P` or `Ctrl+P` | Print |
| `Cmd+F` or `Ctrl+F` | Find in page |
| `Cmd+G` or `Ctrl+G` | Find next |

---

## The Command Palette

### What It Does

The command palette is your keyboard-powered control center:

- **Search everything** - Pages, actions, data, UI controls
- **Navigate instantly** - Jump to any page or section
- **Trigger actions** - Execute commands without clicking
- **Find data** - Search through your strands and notes
- **Control UI** - Activate buttons, toggles, modals

### How to Use It

1. **Open with** `/` or `Cmd+K` (Ctrl+K)
2. **Type what you want** - "create strand", "dark theme", "profile"
3. **Navigate with arrows** - `↑` and `↓` keys
4. **Execute with Enter** - Activate selected command
5. **Close with Escape** - Exit the palette

### Search Examples

Try typing these in the command palette:

- **"create"** - Shows all create actions
- **"strand"** - Shows strand-related commands
- **"theme"** - Theme switching options
- **"dark"** - Toggle dark mode
- **"tutorial"** - Navigate to tutorials
- **"export"** - Export options
- **Your strand name** - Jump directly to that strand

### Command Categories

- **Recent** - Your last 5 commands (appears first)
- **Navigation** - Pages and sections
- **Actions** - Create, import, export, etc.
- **Data** - Your strands and content
- **Settings** - Theme, language, preferences
- **Help** - Documentation and support

---

## List and Table Navigation

| Shortcut | Action |
|----------|--------|
| `↑` | Previous item |
| `↓` | Next item |
| `Enter` | Open/select item |
| `Space` | Toggle checkbox |
| `Delete` or `Backspace` | Delete item |
| `Cmd+A` or `Ctrl+A` | Select all |
| `Cmd+Click` or `Ctrl+Click` | Multi-select |
| `Shift+Click` | Range select |

---

## Modal and Dialog Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` | Next field |
| `Shift+Tab` | Previous field |
| `Enter` | Submit/confirm |
| `Escape` | Cancel/close |

---

## Accessibility Shortcuts

| Shortcut | Action |
|----------|--------|
| `Tab` (on page load) | Reveal "Skip to main content" link |
| `Alt+1` through `Alt+9` | Jump to headings (screen readers) |

---

## Browser Shortcuts (Still Work!)

| Shortcut | Action |
|----------|--------|
| `Cmd+R` or `Ctrl+R` | Refresh page |
| `Cmd++` or `Ctrl++` | Zoom in |
| `Cmd+-` or `Ctrl+-` | Zoom out |
| `Cmd+0` or `Ctrl+0` | Reset zoom |
| `Cmd+W` or `Ctrl+W` | Close tab |
| `Cmd+T` or `Ctrl+T` | New tab |

---

## For Plugin Developers

### Registering Custom Commands

Make your plugin's actions searchable and keyboard-accessible:

```typescript
import { definePlugin } from '@framers/openstrand-plugins/api';

export default definePlugin({
  name: 'my-plugin',
  
  contributes: {
    commands: [
      {
        id: 'my-plugin.custom-action',
        title: 'My Custom Action',
        subtitle: 'Does something amazing',
        icon: 'sparkles',
        handler: async (context) => {
          // Your action logic
        },
        keywords: ['custom', 'action', 'special'],
        shortcut: 'M', // Single letter shortcut
        category: 'actions',
      },
    ],
  },
});
```

Your command will automatically appear in the command palette when users search for it!

### Registering UI Controls

Make your UI controls keyboard-accessible:

```typescript
contributes: {
  ui: {
    buttons: [
      {
        id: 'my-button',
        label: 'My Button',
        tooltip: 'Click or press M to activate',
        commandId: 'my-plugin.custom-action', // Links to command above
        icon: 'star',
      }
    ]
  }
}
```

---

## Tips for Power Users

### 1. Command Chaining

Quickly execute multiple actions:

1. `/` → "theme" → `Enter` - Change theme
2. `/` → "create" → `Enter` - Create strand
3. `/` → "export" → `Enter` - Export data

### 2. Recent Commands

Your last 5 commands appear first in the palette. No need to search again!

### 3. Fuzzy Search

Don't remember the exact name? Type partial words:

- "dash" finds "Dashboard"
- "tut" finds "Tutorials"
- "exp" finds "Export"

### 4. Learn by Searching

Don't know where something is? Search for it in the command palette:

- Type "theme" to find theme settings
- Type "import" to find import options
- Type "help" to find documentation

---

## Troubleshooting

### Command Palette Won't Open

- **Check if another app captured the shortcut** (e.g., macOS Spotlight uses `Cmd+Space`)
- **Try the alternative** - Use `/` instead of `Cmd+K`
- **Refresh the page**

### Shortcut Conflicts

- Some shortcuts may conflict with browser or OS shortcuts
- Use the command palette instead (`/` or `Cmd+K`)
- Customize shortcuts in Settings (coming soon)

### Keyboard Not Working

- **Check focus** - Click on the page first
- **Disable browser extensions** that might interfere
- **Check browser console** for errors

---

## Quick Reference Card

Save this for quick access:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         OPENSTRAND KEYBOARD SHORTCUTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Universal
  /  or  Cmd+K     Command Palette
  ?               Show Help
  Esc             Close/Cancel

Navigation (G + letter)
  G H             Home
  G D             Dashboard
  G S             Strands
  G T             Tutorials
  G P             Profile

Actions
  C               Create
  I               Import
  E               Export
  A               AI Assistant
  T               Toggle Theme

Editing
  Cmd+B           Bold
  Cmd+I           Italic
  Cmd+S           Save
  Cmd+Z           Undo

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## See Also

- [UI Component Guide](UI_COMPONENT_GUIDE.md) - Component documentation
- [Design System](DESIGN_SYSTEM.md) - Design patterns
- [Plugin Development](../packages/openstrand-plugins/README.md) - Building plugins
- [Accessibility Guide](#) - Screen reader support

---

**Last Updated:** November 14, 2024  
**Maintained By:** OpenStrand Team

