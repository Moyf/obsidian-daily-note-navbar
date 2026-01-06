# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Obsidian plugin that adds a navigation bar to daily notes, allowing users to quickly navigate between sequential daily notes and weekly notes. The plugin replaces the default view header title container in daily note files with a custom navbar showing a week's worth of dates.

## Development Commands

```bash
# Development mode (with watch and hot reload)
npm run dev

# Production build (with type checking)
npm run build

# Version bump (updates manifest.json and versions.json)
npm version
```

## Architecture

### Core Components

- **[src/main.ts](src/main.ts)** - `DailyNoteNavbarPlugin` class
  - Main plugin entry point extending Obsidian's `Plugin`
  - Manages navbar instance lifecycle via `Record<string, DailyNoteNavbar>`
  - Handles file opening operations with different pane types
  - Coordinates dependency checks (Daily Notes/Periodic Notes plugins)

- **[src/dailyNoteNavbar/dailyNoteNavbar.ts](src/dailyNoteNavbar/dailyNoteNavbar.ts)** - `DailyNoteNavbar` class
  - UI component that renders the navbar
  - Manages week offset state for navigating between weeks
  - Handles user interactions (clicks, context menus, week switching)
  - Attached to views via `daily-note-navbar-id` attribute

- **[src/settings.ts](src/settings.ts)** - Settings management
  - `DailyNoteNavbarSettingTab` - UI for plugin settings
  - `DailyNoteNavbarSettings` interface and `DEFAULT_SETTINGS`

- **[src/utils.ts](src/utils.ts)** - Utility functions
  - Date handling (`getDatesInWeekByDate`, `getDateFromFileName`)
  - File operations (`getDailyNoteFile` - creates notes if they don't exist)
  - DOM utilities (`hideChildren`, `showChildren`)

- **[src/types.ts](src/types.ts)** - Type definitions using `as const` pattern
- **[src/dailyNoteNavbar/consts.ts](src/dailyNoteNavbar/consts.ts)** - Constants for file open types and mappings

### Key Patterns

**Event-Driven Architecture**: Plugin listens to Obsidian events:
- `active-leaf-change`: Triggers navbar creation/update when switching files
- `css-change`, `create`, `rename`, `delete`: Triggers navbar re-render

**View Hijacking**: The plugin hides the default `.view-header-title-container` content and injects its custom navbar in its place using the `.daily-note-navbar__hidden` utility class.

**Component Reuse**: Each Markdown view can have an associated navbar. Navbars are tracked by ID and reused when switching between daily notes.

**Date Detection**: A file is considered a daily note if its basename matches the configured date format (`dailyNoteDateFormat` setting). Uses `moment(dateString, format, true)` for strict parsing.

**Dependency Injection**: The `DailyNoteNavbar` component receives the plugin instance for accessing settings and calling `plugin.openDailyNote()`.

### Dependencies

- **obsidian-daily-notes-interface**: Core library for daily/weekly note operations
  - Daily: `getAllDailyNotes`, `getDailyNote`, `createDailyNote`
  - Weekly: `getAllWeeklyNotes`, `getWeeklyNote`, `createWeeklyNote`
- Requires either Daily Notes (built-in) or Periodic Notes (community plugin) to be enabled

### Build System

- **esbuild** (via [esbuild.config.mjs](esbuild.config.mjs)): Bundles TypeScript to CommonJS
- External modules: Obsidian, Electron, CodeMirror, Lezer, and Node built-ins
- Development: inline sourcemaps, watch mode
- Production: no sourcemaps, tree-shaking enabled
- **TypeScript**: Type checking via `tsc -noEmit -skipLibCheck` before build

### CSS Class Naming Convention

Uses BEM-like pattern:
- Base: `.daily-note-navbar`
- States: `daily-note-navbar__active`, `daily-note-navbar__current`, `daily-note-navbar__not-exists`
- Actions: `daily-note-navbar__change-week`
- Component-specific: `daily-note-navbar__weekly` (for weekly note button)
- Utilities: `daily-note-navbar__hidden`

### Code Style

- Indent: Tabs (4 space width)
- Line endings: LF
- TypeScript: Strict mode with `strictNullChecks`
- Use `as const` for type-safe constant arrays
- Use `typeof CONSTANT[number]` to infer union types from `as const` arrays

### File Open Types

The plugin supports multiple ways to open daily notes:
- `Active` - Current leaf
- `New tab` - New tab
- `New window` - New window
- `Split right` - Vertical split
- `Split down` - Horizontal split

Opening behavior is determined by click modifiers (Ctrl+click, middle-click) or the `defaultOpenType` setting.

## Implementation Notes & Lessons Learned

### Weekly Notes Support (v0.3.0)

When adding weekly notes support, several important patterns were established:

1. **Settings Organization**:
   - General settings (First day of week, Open files as active, etc.) should be at the top WITHOUT a group title
   - Feature-specific settings (Daily Notes, Weekly Notes) get `h2` group titles
   - This makes the most common settings easily accessible

2. **TypeScript Type Handling**:
   - When accessing `app.plugins.getPlugin()`, use `// @ts-ignore` comment
   - The Obsidian type definitions don't include all plugin-related properties
   - Example: `this.app.plugins.getPlugin("periodic-notes")` requires `@ts-ignore`

3. **Weekly Note Date Calculation**:
   - Use `date.clone().startOf('isoWeek')` for ISO week start (Monday)
   - Handle Sunday start specially: if `firstDayOfWeek === "Sunday"`, subtract 1 day from Monday
   - The week start date is used as the key for `getWeeklyNote()`

4. **Button Rendering Order**:
   - Weekly note button should be the FIRST button (leftmost)
   - Before any extra buttons (Last Sunday)
   - Before the Previous week arrow button

5. **CSS Styling for Special Buttons**:
   - Add a specific class like `daily-note-navbar__weekly` to distinguish special buttons
   - Use `font-weight: var(--font-semibold)` for visual prominence
   - Add hover effects with `color: var(--interactive-accent)`

6. **Default Values Strategy**:
   - New features should default to DISABLED to avoid disrupting existing users
   - Example: `enableWeeklyNoteButton: false`
   - Weekly note format should match Periodic Notes default: `"gggg-[W]ww"`

7. **Display vs File Format Separation**:
   - File format (for parsing filenames): e.g., `dailyNoteDateFormat: "YYYY-MM-DD"`
   - Display format (for UI): e.g., `dateFormat: "ddd"`
   - Keep these separate - one is strict parsing, the other is user-facing

8. **Settings Naming Convention**:
   - Use descriptive names: `weeklyNoteDisplayFormat` vs just `displayFormat`
   - Group-related settings with consistent prefixes
   - Match display names to internal keys for clarity

### Git Workflow

When working on features with multiple Claude instances:

1. Use feature branches: `git checkout -b feature/description`
2. Work on the feature branch
3. Test with `npm run build` before committing
4. Merge with `git checkout master && git merge --no-ff feature-branch`
5. Delete feature branch after merge: `git branch -d feature-branch`

This avoids conflicts when multiple AI agents work simultaneously.
