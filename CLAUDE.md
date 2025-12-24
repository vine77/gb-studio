# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GB Studio is a drag-and-drop game creator for making Game Boy games. It's designed to be usable by people with little to no programming knowledge using visual scripting, but also provides access to the game engine's virtual machine (GBVM) and allows direct modification of the C and assembly code through plugins. Games compile to ROM files that run on emulators, web browsers, or real Game Boy hardware.

---

## User Guide

### Scene Types

GB Studio supports multiple scene types, each with different player handling:

- **Top Down 2D** - Player moves in four directions on a grid. Good for RPGs, adventure games, puzzle games.
- **Platformer** - Side-scrolling with gravity, jumping, and optional features like wall jumps, double jump, dashing, slopes, ladders, and moving platforms.
- **Adventure** - Point-and-click style movement.
- **Shoot 'Em Up** - Scrolling shooter gameplay.
- **Point and Click** - Cursor-based interaction.

A single project can mix scene types. Plugins can add custom scene types.

### Core Concepts

#### Scenes
A scene is a single screen containing a background image, actors, and triggers. Games are built by connecting scenes together. Each scene can have:
- Up to 20 actors
- Up to 30 triggers
- An On Init script (runs when scene loads)
- On Player Hit scripts (for collision groups)
- Parallax scrolling (up to 3 layers at different speeds)
- Common tilesets for seamless transitions between scenes

Use `/` in scene names to organize them into folders in the Navigator (e.g., `caves/Underground`).

#### Actors
Actors are interactive characters and objects. Properties include:
- Sprite sheet, animation speed, movement speed
- Collision group (for triggering On Hit scripts)
- Pin to Screen option (for HUD elements)
- Multiple scripts: On Init, On Interact, On Hit, On Update

#### Triggers
Invisible rectangular areas that run scripts when the player enters or leaves them. Commonly used for scene transitions and cutscenes.

#### The Player
The player start position is shown with a special icon that can be dragged between scenes. Each scene type can have a different default player sprite. The player can be hidden for title screens or cutscenes using the Hide Actor event.

### Asset Requirements

All assets go in the project's `assets/` folder and are automatically detected.

#### Sprites (`assets/sprites/`)
- PNG files using exactly 4 colors: `#071821`, `#86c06c`, `#e0f8cf`, `#65ff00` (transparent)
- Simple sprites: 16x16px per frame, laid out horizontally
- Actor sprites: 48x16px (3 frames: down, up, right - left is auto-flipped)
- Animated actors: 96x16px (6 frames: 2 each for down, up, right)
- Use the Sprite Editor for complex multi-state animations

#### Backgrounds (`assets/backgrounds/`)
- PNG files, 4 colors: `#071821`, `#306850`, `#86c06c`, `#e0f8cf`
- Minimum 160x144px (GB screen size), max 2040px per dimension
- Must be multiples of 8px
- Max 192 unique 8x8 tiles (384 in Color Only mode)
- For color images with automatic palettes: max 4 colors per 8x8 tile, max 8 unique palettes per scene
- Provide `.mono.png` override files for better monochrome appearance when using auto palettes

#### Music (`assets/music/`)
- `.uge` files (recommended) - editable in the built-in Music Editor
- `.mod` files - for legacy projects using GBT Player
- Project can only use one format (configured in Settings)

#### Sound Effects (`assets/sounds/`)
- `.wav` files (8-bit mono, under ~3.6 seconds)
- `.vgm` files (Game Boy format)
- `.sav` files (FX HAMMER format)
- Also available: built-in beep, tone, and crash sounds

#### UI Elements (`assets/ui/`)
- `frame.png` - dialogue window frame (9-slice scaled)
- `cursor.png` - menu selection cursor

#### Other Assets
- **Fonts** (`assets/fonts/`) - PNG + JSON pairs for custom fonts
- **Emotes** (`assets/emotes/`) - 16x16px emotion bubbles
- **Avatars** (`assets/avatars/`) - 16x16px faces for dialogue
- **Tilesets** (`assets/tilesets/`) - for Replace Tile events and common tilesets

### Color Modes

Configured in Settings:
- **Monochrome** - 4 colors, runs on all devices
- **Color + Monochrome** - Color palettes on supported devices, monochrome fallback
- **Color Only** - Doubles tile limits (384 background, 192 sprite tiles), but only runs on GB Color/Analogue Pocket

Each scene can use 8 background palettes and 8 sprite palettes. Palette 8 is also used for UI.

### Scripting System

Scripts are visual event sequences attached to scenes, actors, or triggers.

#### Script Types
- **Scene On Init** - Runs once when scene loads (after actor On Init scripts)
- **Actor On Init** - Runs once when scene loads (before scene On Init)
- **Actor On Interact** - When player presses A while facing actor
- **Actor On Hit** - When actor collides with specified collision groups
- **Actor On Update** - Runs every frame while actor is on screen
- **Trigger On Enter/Leave** - When player enters/exits trigger area

#### Key Event Categories
- **Actor** - Move, show/hide, change sprite, launch projectiles, set animation state
- **Camera** - Move, shake, lock/unlock, set bounds
- **Dialogue & Menus** - Display text, choices, avatars
- **Control Flow** - If/else, loops, switches, labels/goto
- **Math** - Evaluate expressions, random numbers
- **Music & Sound** - Play/stop music, sound effects
- **Scene** - Change scene, scene stack (push/pop for menus)
- **Variables** - Set, increment, flags, store actor positions
- **Screen** - Fade in/out, overlay, palettes
- **Timer** - Timed script execution
- **Save Data** - Save/load game state

#### Script Values and Math Expressions
Many events support Script Values - visual building blocks for combining variables, numbers, and operations. You can also type math expressions directly using operators (`+`, `-`, `*`, `/`, `==`, `!=`, `>=`, `&&`, `||`, `!`) and functions (`min`, `max`, `abs`, `atan2`, `isqrt`, `rnd`).

Reference variables in expressions with `$VariableName`.

#### Dialogue Variables
In text events, type `$` to insert variable values. Formatting prefixes:
- `%D5$Var` - Fixed length with leading zeros
- `%c$Var` - Display as ASCII character
- `%t$Var` - Set text speed
- `%f$Var` - Change font

Text commands (type `!`): `!Font`, `!Speed`, `!Instant`, `!Cursor`

#### Custom Scripts
Create reusable script functions that can be called from anywhere. Variables can be passed by reference (modifiable) or by value (copied).

### Music Editor

The built-in tracker for `.uge` files supports:
- 4 channels: Duty 1, Duty 2, Wave, Noise
- Piano Roll view (mouse-based) or Tracker view (keyboard-based)
- Pattern sequencing
- Instrument editing (duty cycle, envelopes, waveforms, noise macros)
- Effects: arpeggio, portamento, vibrato, volume, panning, and more

### Debugger

Access via the panel at bottom of Game World or `Game > Run With Debugging`:
- **VRAM Preview** - See tile usage and palettes
- **Variables** - View/edit variable values live, watch specific variables
- **Breakpoints** - Pause on script changes, variable changes, or specific events
- **Script Threads** - Step through scripts instruction-by-instruction or frame-by-frame
- **Build Log** - View compilation warnings and errors

Keyboard shortcuts: `F8` (pause/resume), `F9` (step instruction), `F10` (step frame)

### Scene Limits

Per scene:
- 20 actors max (keep under 10 in any 160x144px area to avoid visibility issues)
- 30 triggers max
- Background tiles: 192 (Mono/Color+Mono) or 384 (Color Only)
- Sprite tiles: 64-96 (Mono) or 128-192 (Color Only), depending on background complexity

### Building and Exporting

- **Play** - Test in built-in emulator window
- **Export ROM** - Creates `.gb` file in `build/rom/`
- **Export Web** - Creates HTML5 build in `build/web/` (uploadable to itch.io, recommended viewport 480x432px)
- **Export Pocket** - Creates `.pocket` file for Analogue Pocket

### Extending GB Studio

#### Plugins
Place in project's `plugins/` folder:
- **Asset Plugins** - Reusable sprites, backgrounds, fonts
- **Script Event Plugins** - Custom events (JS files in `plugins/yourPlugin/events/event*.js`)
- **Engine Plugins** - Modify game engine C code, add engine fields, add scene types

#### Engine Eject
`Game > Advanced > Engine Eject` copies the GBDK engine to `assets/engine/` for direct modification. Delete files to revert to defaults.

### Keyboard Shortcuts

**Play Window:** Arrow keys/WASD (move), Alt/Z/J (A), Ctrl/K/X (B), Enter (Start), Shift (Select)

**Editor:**
- `V` Select, `A` Add Actor, `T` Add Trigger, `S` Add Scene, `E` Eraser, `C` Collisions
- `P` Set player start (while hovering)
- `Space` + drag to pan, `Ctrl/Cmd` + scroll to zoom
- `/` Focus scene search
- `Ctrl/Cmd + S` Save, `Ctrl/Cmd + B` Run

**Collision Tool:** `1` Solid, `2-5` Directional, `6` Ladder

**Music Editor:** Space (play/pause), `` ` `` (toggle tracker/piano roll)

### Project Files

- `.gbsproj` - Main project file (JSON, version control friendly)
- `.gbsproj.bak` - Automatic backup of previous save
- `build/` folder - Generated output (exclude from version control)

---

## Development Guide

### Build Commands

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

### Architecture

#### Electron Process Separation

The codebase strictly separates main process and renderer process code:

- **`src/lib/`** - Main process only. Can use Node.js APIs, fs, Electron main APIs. Cannot import from `renderer/`, `store/`, or `components/`.
- **`src/renderer/`** - Renderer process only. React components and browser APIs. Cannot import from `lib/`.
- **`src/shared/`** - Environment-agnostic utilities. Cannot import from `lib/`, `renderer/`, `store/`, or `components/`. Cannot use fs, React, or Electron APIs.
- **`src/components/`** - React UI components for renderer process.
- **`src/store/`** - Redux state management (renderer process).

ESLint enforces these boundaries - violations cause build errors.

#### Key Directories

- **`src/lib/compiler/`** - Compiles GB Studio projects to GBDK C code and ROM files. `scriptBuilder.ts` (237KB) generates GBVM bytecode from visual scripts.
- **`src/lib/events/`** - Script event definitions. Each `event*.js` file defines a visual scripting block that generates GBVM output.
- **`src/lib/project/`** - Project loading, saving, and migration.
- **`src/store/features/entities/`** - Redux state for all game entities (scenes, actors, triggers, sprites, etc.).
- **`appData/`** - Game engine source (GBDK C code), templates, and WASM modules.

#### Data Flow

1. User creates/edits project in UI (React components)
2. Changes flow through Redux store (`src/store/features/`)
3. Project saved as `.gbsproj` JSON file
4. Build process (`src/lib/compiler/`) converts project to:
   - GBVM bytecode (from visual scripts via `scriptBuilder.ts`)
   - GBDK C source files
   - Compiled ROM via GBDK toolchain

#### Script Events System

Script events are the visual programming blocks users drag and drop. Each event:
- Lives in `src/lib/events/event*.js`
- Must have filename starting with `event`
- Exports metadata (name, description, fields) and a `compile()` function
- The `compile()` function generates GBVM bytecode via the script builder API

Plugins can add custom events in a project's `plugins/` folder following the same pattern.

#### Redux Store Structure

Main state slices in `src/store/features/`:
- `entities` - Scenes, actors, triggers, sprites, palettes, scripts, etc.
- `editor` - UI state (selected scene, tool, zoom level)
- `settings` - Project settings (color mode, engine settings)
- `project` - Undo/redo wrapped combination of entities, settings, metadata

### Localization

Translation files live in `src/lang/`. To find missing translations:
```bash
npm run missing-translations de    # Check German
npm run missing-translations es    # Check Spanish
```

### Profiling

When running from source, enable "BGB Profiling" in Build settings. Use BGB emulator with:
```bash
./bgb -watch -rom game.gb
```
See `DEVELOPERS.md` for profiling toolkit details.

### Commit Message Style

Use imperative mood: "Add feature" not "Added feature". First line under 72 chars. Reference issues in header `[#123]` or body `Fixes #123`.
