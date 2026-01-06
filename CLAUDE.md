# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Obsidian plugin that adds a navigation bar to daily notes, allowing users to quickly navigate between sequential daily notes. The plugin replaces the default view header title container in daily note files with a custom navbar showing a week's worth of dates.

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

- **obsidian-daily-notes-interface**: Core library for daily note operations (`getAllDailyNotes`, `getDailyNote`, `createDailyNote`)
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
