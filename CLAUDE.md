# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GB Studio is an Electron-based drag-and-drop game creator for making Game Boy games. It consists of a visual game builder application and a C-based game engine using GBDK. Games compile to ROM files that run on emulators, web browsers, or real Game Boy hardware.

## Common Commands

```bash
# Install dependencies (requires Node 21.7.1 - see .nvmrc)
corepack enable
yarn
npm run fetch-deps    # Fetches GBVM, GBDK, and other submodule dependencies

# Development
npm start             # Start Electron app in development mode
npm run storybook     # Run Storybook for UI component development

# Build & Package
npm run package                      # Build for current platform
npm run make:mac                     # Build macOS ARM64
npm run make:mac-intel               # Build macOS x64
npm run make:win                     # Build Windows x64
npm run make:linux                   # Build Linux x64

# CLI Tool
npm run make:cli                     # Build CLI tool
yarn link                            # Link CLI globally (first time only)
$(yarn bin gb-studio-cli) --help     # Run CLI

# Quality
npm run lint          # Run ESLint
npm run prettier      # Format code
npm test              # Run all tests with Jest
npm test -- path/to/test.test.ts     # Run a single test file
npm test -- --watch                  # Run tests in watch mode
npm run dead-code     # Find unused code with Knip
```

## Architecture

### Electron Process Separation

The codebase strictly separates main process and renderer process code:

- **`src/lib/`** - Main process only. Can use Node.js APIs, fs, Electron main APIs. Cannot import from `renderer/`, `store/`, or `components/`.
- **`src/renderer/`** - Renderer process only. React components and browser APIs. Cannot import from `lib/`.
- **`src/shared/`** - Environment-agnostic utilities. Cannot import from `lib/`, `renderer/`, `store/`, or `components/`. Cannot use fs, React, or Electron APIs.
- **`src/components/`** - React UI components for renderer process.
- **`src/store/`** - Redux state management (renderer process).

ESLint enforces these boundaries - violations cause build errors.

### Key Directories

- **`src/lib/compiler/`** - Compiles GB Studio projects to GBDK C code and ROM files. `scriptBuilder.ts` (237KB) generates GBVM bytecode from visual scripts.
- **`src/lib/events/`** - Script event definitions. Each `event*.js` file defines a visual scripting block that generates GBVM output.
- **`src/lib/project/`** - Project loading, saving, and migration.
- **`src/store/features/entities/`** - Redux state for all game entities (scenes, actors, triggers, sprites, etc.).
- **`appData/`** - Game engine source (GBDK C code), templates, and WASM modules.

### Data Flow

1. User creates/edits project in UI (React components)
2. Changes flow through Redux store (`src/store/features/`)
3. Project saved as `.gbsproj` JSON file
4. Build process (`src/lib/compiler/`) converts project to:
   - GBVM bytecode (from visual scripts via `scriptBuilder.ts`)
   - GBDK C source files
   - Compiled ROM via GBDK toolchain

### Script Events System

Script events are the visual programming blocks users drag and drop. Each event:
- Lives in `src/lib/events/event*.js`
- Must have filename starting with `event`
- Exports metadata (name, description, fields) and a `compile()` function
- The `compile()` function generates GBVM bytecode via the script builder API

Plugins can add custom events in a project's `plugins/` folder following the same pattern.

### Redux Store Structure

Main state slices in `src/store/features/`:
- `entities` - Scenes, actors, triggers, sprites, palettes, scripts, etc.
- `editor` - UI state (selected scene, tool, zoom level)
- `settings` - Project settings (color mode, engine settings)
- `project` - Undo/redo wrapped combination of entities, settings, metadata

## Localization

Translation files live in `src/lang/`. To find missing translations:
```bash
npm run missing-translations de    # Check German
npm run missing-translations es    # Check Spanish
```

## Profiling

When running from source, enable "BGB Profiling" in Build settings. Use BGB emulator with:
```bash
./bgb -watch -rom game.gb
```
See `DEVELOPERS.md` for profiling toolkit details.

## Commit Message Style

Use imperative mood: "Add feature" not "Added feature". First line under 72 chars. Reference issues in header `[#123]` or body `Fixes #123`.
