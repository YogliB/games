# Copilot Instructions

## Project Overview

This is a **Phaser 3 + Svelte + SvelteKit** game template using Vite for bundling. The key architectural pattern is a bridge between Phaser (canvas game engine) and Svelte (reactive UI framework), communicating via a custom EventBus.

## Critical Architecture

### Svelte-Phaser Bridge (`src/PhaserGame.svelte`)
- **Single source of truth**: This component initializes Phaser and exposes game/scene references via `phaserRef` prop
- **EventBus pattern**: All Svelte ↔ Phaser communication goes through `EventBus` (Phaser's EventEmitter)
- **Scene lifecycle**: Each Phaser scene MUST emit `'current-scene-ready'` in its `create()` method to notify Svelte
  ```typescript
  EventBus.emit('current-scene-ready', this);  // Required in every scene
  ```

### SSR Disabled (`src/routes/+layout.js`)
- **CRITICAL**: `export const ssr = false;` is required - Phaser requires browser APIs (canvas, WebGL)
- Do NOT enable SSR or the game will break on server rendering

### Game Configuration (`src/game/main.ts`)
- Phaser config uses `parent: 'game-container'` to mount into Svelte's `<div id="game-container">`
- Scene order in config array determines boot sequence: `[Boot, Preloader, MainMenu, MainGame, GameOver]`

## Development Workflow

### Running the Project
```bash
npm run dev          # Dev server on port 8080 with telemetry
npm run dev-nolog    # Dev server WITHOUT telemetry to gryzor.co
npm run build        # Production build to build/ folder
```

### Adding New Phaser Scenes
1. Create scene file in `src/game/scenes/`
2. Add to scene array in `src/game/main.ts` config
3. **MUST** emit `EventBus.emit('current-scene-ready', this)` in `create()` method if Svelte needs access
4. Use `scene.changeScene()` pattern (see `MainMenu.ts`) for Svelte-triggered transitions

### Asset Loading
- Place assets in `static/assets/` (flat structure: bg.png, logo.png, star.png)
- Load with relative path: `this.load.image('background', 'assets/bg.png')`
- Assets auto-copy to `build/assets/` on production build

## Code Patterns

### Accessing Phaser from Svelte
```typescript
// In Svelte component
let phaserRef: TPhaserRef = { game: null, scene: null };

// Access current scene (typed)
const scene = phaserRef.scene as MainMenu;
scene.moveLogo((pos) => { /* callback */ });
```

### Scene-Specific Logic
- Type-check scene before calling methods: `const scene = phaserRef.scene as MainMenu`
- Check scene key: `canMoveSprite = (scene.scene.key !== "MainMenu")`
- See `src/routes/+page.svelte` for examples of calling scene methods, adding sprites dynamically

### EventBus Usage
- **Svelte → Phaser**: `EventBus.emit('event-name', data)`
- **Phaser → Svelte**: `EventBus.on('event-name', callback)` (setup in `onMount`)
- Only used for lifecycle events (`current-scene-ready`); direct scene method calls preferred for game logic

## TypeScript Configuration

- App code uses `tsconfig.app.json` (extends Svelte preset)
- `checkJs: true` - JS files are type-checked
- Imports use relative paths; no path aliases configured

## Build System

- **Dev**: Vite dev server with HMR on `http://localhost:8080`
- **Prod**: Static site adapter (`@sveltejs/adapter-static`) with `fallback: 'index.html'` for SPA
- Vite configs split: `vite/config.dev.mjs` (dev) and `vite/config.prod.mjs` (prod)
- Output: `build/` folder (not `dist/`)

## Conventions

- Phaser scenes use PascalCase class names: `Game`, `MainMenu`, `GameOver`
- Scene keys match class names: `super('MainMenu')`
- Svelte components use PascalCase: `PhaserGame.svelte`
- Game objects stored as scene properties: `this.logo`, `this.background`
- Methods like `changeScene()` are custom scene methods for Svelte integration (not Phaser built-ins)

## Common Pitfalls

1. **Forgetting `current-scene-ready` event** - Svelte won't track scene changes
2. **Enabling SSR** - Breaks Phaser initialization
3. **Wrong asset paths** - Use `'assets/file.png'` not `'/assets/file.png'` in Phaser loaders
4. **Scene not in config** - Must add new scenes to array in `main.ts`
5. **Type mismatches** - Cast scene to specific type before calling custom methods
