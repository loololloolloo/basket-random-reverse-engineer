# Basket Random - Reverse Engineering Analysis

**Target Repository**: https://github.com/p0pgames/basket  
**Original Source**: https://github.com/Unblocked-Games-786/Basket-Random  
**Status**: In Progress - Decompilation & Analysis

## Project Overview

Basket Random is a 2-player physics-based basketball game built with **Construct 3** and exported to HTML5/JavaScript. This repository documents the complete reverse engineering and decompilation process.

### Key Technical Details
- **Engine**: Construct 3 (https://www.construct.net)
- **Language**: JavaScript (minified/obfuscated)
- **Physics**: Box2D WebAssembly (246KB .wasm binary)
- **Deployment**: GitHub Pages (gh-pages branch)
- **Game Size**: ~2.3 MB total

---

## Architecture Overview

### File Structure
```
p0pgames/basket/
├── index.html                    # Main entry point
├── style.css                     # Minimal styling
├── sw.js                         # Service worker (disabled)
├── appmanifest.json              # PWA manifest
├── manifest.json                 # Web manifest
├── data.json                     # 202KB serialized game data
├── box2d.wasm                    # 246KB physics engine binary
├── box2d.wasm.js                 # WASM glue code (282KB)
│
├── scripts/
│   ├── c3runtime.js              # 967KB Construct 3 engine (MINIFIED)
│   ├── main.js                   # 89KB Game logic (MINIFIED)
│   ├── supportcheck.js           # Browser capability detection
│   ├── offlineclient.js          # Offline support
│   ├── register-sw.js            # Service worker registration
│   ├── dispatchworker.js         # Web worker dispatcher
│   └── jobworker.js              # Worker job processor
│
├── js/
│   ├── ubg235_client_v1_1.js     # UBG235 external server loader
│   ├── analytics_ubg_v1_4.js     # Analytics (readable)
│   └── analytics_games235.js     # Analytics variant
│
├── media/                        # Game assets (empty in published)
├── images/                       # Image assets (empty)
└── patch/                        # Patch files (empty)
```

---

## Readable Source Files (EXTRACTED)

These files are **already decompiled** and human-readable:

### 1. Analytics Configuration
- `js/analytics_ubg_v1_4.js` - Bot detection + Google Analytics setup
- `js/analytics_games235.js` - Multi-domain analytics routing

### 2. Support & Registration
- `scripts/supportcheck.js` - Browser capability checks (WebGL, Canvas, etc.)
- `scripts/register-sw.js` - Service worker registration logic
- `scripts/offlineclient.js` - Offline client setup

### 3. External Integration
- `js/ubg235_client_v1_1.js` - Loads `https://www.ubg235.com/js/ubg235_server_v1_0.js` at runtime
- `index.html` - HTML structure and entry point
- `style.css` - Styling

---

## MINIFIED/OBFUSCATED FILES (TO BE DECOMPILED)

### High Priority
1. **scripts/main.js** (89KB)
   - Contains compiled game logic
   - Player controls, ball physics, scoring, game state
   - Status: MINIFIED - needs beautification + analysis

2. **scripts/c3runtime.js** (967KB)
   - Construct 3 engine core
   - Event system, object management, rendering pipeline
   - Status: MINIFIED - proprietary code

### Medium Priority
3. **data.json** (202KB)
   - Serialized game project data
   - Contains layouts, object types, event definitions
   - Status: JSON format - needs parsing

4. **box2d.wasm.js** (282KB)
   - WebAssembly loader and glue code
   - Physics engine initialization
   - Status: MINIFIED

### Low Priority
5. **box2d.wasm** (246KB)
   - Binary WebAssembly physics engine
   - Status: BINARY - requires wasm2wat conversion

---

## Decompilation Strategy

### Phase 1: Extract Readable Content ✓
- [x] Download all source files
- [x] Identify minified vs readable files
- [x] Extract analytics and support code

### Phase 2: Beautify & Deobfuscate (IN PROGRESS)
- [ ] Beautify main.js using js-beautify
- [ ] Beautify c3runtime.js using prettier
- [ ] Apply de4js for advanced deobfuscation
- [ ] Manually rename variables based on context

### Phase 3: Parse Game Data
- [ ] Parse data.json structure
- [ ] Extract event definitions
- [ ] Identify object types and behaviors
- [ ] Map game mechanics

### Phase 4: Analyze Physics Engine
- [ ] Convert box2d.wasm to WAT using wasm2wat
- [ ] Identify physics function calls
- [ ] Map collision handlers

### Phase 5: Reconstruct Game Logic
- [ ] Create annotated pseudocode
- [ ] Document game mechanics
- [ ] Identify event-condition-action flows
- [ ] Create gameplay documentation

---

## Tools Used

### Beautification & Deobfuscation
- [jsbeautifier.io](https://beautifier.io/) - Online JS beautifier
- [de4js](https://lelinhtinh.github.io/de4js/) - Advanced deobfuscation
- [jsnice.org](http://www.jsnice.org/) - Variable name suggestions
- Prettier - Code formatting

### Binary Analysis
- [WebAssembly Explorer](https://mbebenita.github.io/WasmExplorer/) - WASM to WAT
- [WebAssembly Studio](https://webassembly.studio/) - WASM debugging
- WABT (wasm2wat) - Offline conversion

### Analysis
- Browser DevTools (F12) - Runtime analysis
- Node.js - JSON parsing
- Regex search - Pattern identification

---

## Key Findings So Far

### Game Mechanics (Observed)
1. **2-Player Basketball**: Local co-op on same keyboard
2. **Ragdoll Physics**: Players controlled via Physics behavior
3. **Box2D Integration**: Real-time collision detection
4. **Randomization**: "Random" courts and conditions each game
5. **Scoring System**: First to X points wins
6. **Web Worker Threading**: Offline support via web workers

### External Dependencies
- Google Analytics (dual IDs for bot/human detection)
- UBG235 server integration (ubg235.com)
- Box2D physics library (WebAssembly compiled)

### Construct 3 Specifics
- Entire game logic compiled to minified JS
- Game state stored in serialized data.json
- Event sheet logic converted to function calls
- Visual editor output → binary-like output

---

## Expected Game Logic Structure

Based on Construct 3 architecture:

```
Game State:
  - Player1 (sprite with physics)
  - Player2 (sprite with physics)
  - Ball (sprite with physics)
  - Baskets (left/right)
  - Score (Player1, Player2)
  - Game Over flag
  - Current Layout/Level

Event Sheets (compiled into main.js):
  - "On Start of Layout" → Initialize players, ball, layout
  - "On Key Down (W/Up Arrow)" → Apply jump impulse
  - "On Key Down (A/D/Left/Right)" → Apply lateral force
  - "On Ball collision with Basket" → Add score
  - "On Score >= 5" → Show win screen
  - "Every tick" → Update physics, render, check win condition
  - "On Reset key" → Reset game state

Physics (Box2D):
  - Gravity simulation
  - Collision detection (player-ball, ball-basket, player-ground)
  - Impulse/force application
  - Ragdoll body constraints
```

---

## Next Steps

1. **Download main.js** from raw GitHub URL
2. **Beautify** using online tool or CLI
3. **Search for patterns**: `keyCode`, `score`, `collision`, `physics`
4. **Extract event handlers**: Look for `on`, `addEventListener`, callbacks
5. **Identify objects**: Search for sprite/object definitions
6. **Map data.json**: Parse and extract game structure
7. **Document findings**: Create pseudocode version

---

## Resources

- [Construct 3 Manual](https://www.construct.net/en/make-games/manuals/construct-3)
- [Box2D Documentation](https://box2d.org/)
- [WebAssembly Documentation](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [Minified Code Analysis Best Practices](https://github.com/topics/decompilation)

---

## Legal Notice

This reverse engineering is for educational purposes only. The original game is hosted on GitHub under public licenses. Analysis respects Scirra's Construct 3 proprietary code and seeks only to understand game mechanics and design patterns.

**Date Started**: 2026-09-16  
**Status**: Active Research  
**Progress**: 25%
