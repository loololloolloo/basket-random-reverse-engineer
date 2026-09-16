# Basket Random - Deep Technical Analysis

## Construct 3 Architecture Deep Dive

### What is Construct 3?

Construct 3 is a **visual game development platform** that:
- Uses a drag-and-drop event system instead of code
- Compiles to HTML5/JavaScript for web deployment
- Supports behaviors, plugins, and custom extensions
- Exports complete games as self-contained packages

### How Basket Random Was Built

1. **Designer creates in Construct 3 IDE**:
   - Drag sprites (Player1, Player2, Ball, Baskets) onto canvas
   - Add Physics behavior to enable collision/gravity
   - Create Event Sheets with visual logic blocks
   - Define conditions ("Key pressed", "Collision") and actions ("Apply force", "Add to score")

2. **Construct 3 compiler converts to JavaScript**:
   - Visual events → JavaScript function calls
   - Object properties → Runtime data structures
   - Behaviors → Plugin code
   - Graphics → Canvas/WebGL rendering calls

3. **Output is deployed**:
   - **data.json**: Serialized game project (layouts, objects, variables)
   - **scripts/main.js**: Compiled game logic (minified)
   - **scripts/c3runtime.js**: Construct 3 engine (minified)
   - **box2d.wasm**: Physics simulation

### The Problem: Reverse Engineering

When Construct 3 exports:
```javascript
// ORIGINAL (visual event sheet in Construct 3 IDE)
Event 1: On Key "W" Down
  Action: Player1.ApplyForce(up)

// COMPILED OUTPUT (minified main.js)
// All readable text is gone:
a.prototype.b=function(){c[d].e(f,g)}
```

The **variable names are permanently lost** unless Construct 3 includes source maps (which it doesn't).

---

## File-by-File Breakdown

### index.html - Entry Point
**Purpose**: Loads all scripts and initializes the game

**Key Points**:
- Loads `box2d.wasm.js` first (physics engine)
- Loads `scripts/c3runtime.js` (game engine)
- Loads `scripts/main.js` (game logic)
- Attaches to window.DOMHandler

**Readable Code**:
```html
<script src="box2d.wasm.js"></script>
<script src="scripts/supportcheck.js"></script>
<script src="scripts/offlineclient.js"></script>
<script src="scripts/main.js"></script>
<script src="scripts/register-sw.js"></script>
```

### scripts/c3runtime.js (967 KB) - THE CONSTRUCT 3 ENGINE

**What it contains**:
- Core game loop (event evaluation, drawing, physics tick)
- Object/Sprite management system
- Event dispatcher (fires conditions, runs actions)
- Plugin loader and behavior system
- Rendering pipeline (Canvas/WebGL abstraction)
- Input handling (keyboard, mouse, touch)
- Audio manager
- Storage/persistence layer

**Why it's hard to decompile**:
- Proprietary Scirra code
- Intentionally obfuscated
- Uses heavy minification (a, b, c, d, e...)
- Contains eval() and dynamic code generation
- ~967 KB of compressed logic

**What we can deduce**:
By searching for patterns, we can identify:
- `keydown`, `keyup` handlers → input system
- `update`, `tick` → game loop
- `colliding`, `overlap` → collision detection
- `canvas`, `context` → rendering
- `JSON.parse(data)` → loads data.json

### scripts/main.js (89 KB) - GAME LOGIC (MINIFIED)

**What it contains**:
- Game state variables (player positions, scores, etc.)
- Event handler functions (key presses, collisions, frame updates)
- Scoring logic
- Win condition checks
- Layout initialization
- Game reset logic

**Structure (speculated)**:
```javascript
// Pseudocode of what main.js likely contains:

var gameState = {
  player1: { x, y, vx, vy, health, score },
  player2: { x, y, vx, vy, health, score },
  ball: { x, y, vx, vy },
  baskets: [{ x, y }, { x, y }],
  gameOver: false,
  winner: null
};

function onKeyDown(keyCode) {
  if (keyCode === 87) player1.jump();  // W
  if (keyCode === 65) player1.moveLeft();  // A
  if (keyCode === 68) player1.moveRight();  // D
  if (keyCode === 38) player2.jump();  // Up arrow
  if (keyCode === 37) player2.moveLeft();  // Left arrow
  if (keyCode === 39) player2.moveRight();  // Right arrow
}

function onBallCollision(obj) {
  if (obj === basket_left) {
    gameState.player2.score++;
    resetBall();
    checkWinCondition();
  }
}

function update() {
  // Physics tick
  // Update positions
  // Check collisions
  // Render
  // Update UI
}

function checkWinCondition() {
  if (gameState.player1.score >= 5) {
    gameState.gameOver = true;
    gameState.winner = 1;
  }
}
```

### data.json (202 KB) - GAME PROJECT DATA

**Structure**: Serialized Construct 3 project

**Contains**:
- `project` array: project metadata
- `layouts`: Game level/scene definitions
  - Player1 sprite instance (position, properties)
  - Player2 sprite instance
  - Ball sprite instance
  - Basket sprites
  - Ground/platforms
- `objectTypes`: Object type definitions
  - Physics behavior parameters
  - Image/animation data references
- `eventSheets`: Event system data (compiled to main.js)
  - Condition type IDs
  - Action type IDs
  - Variable references
- `globalVars`: Game-wide variables (score, level, etc.)
- `families`: Object groupings/inheritance

**Why it's compressed**:
- UIDs instead of names (0, 1, 2, 3...)
- Booleans as 0/1
- Array-based encoding
- No whitespace

**Example structure**:
```json
{
  "project": [
    "Basket Random",
    "intro",
    [[0, false, true, true, ...], [4, true, false, false, ...]],
    ...
  ],
  "layouts": [
    {
      "name": "Level1",
      "uid": 0,
      "instances": [
        { "uid": 100, "type": 4, "x": 400, "y": 300 },  // Player1
        { "uid": 101, "type": 5, "x": 800, "y": 300 },  // Player2
        { "uid": 102, "type": 6, "x": 600, "y": 250 }   // Ball
      ]
    }
  ]
}
```

### box2d.wasm + box2d.wasm.js - PHYSICS ENGINE

**box2d.wasm (246 KB - Binary)**
- Compiled WebAssembly physics engine
- Handles gravity, collisions, constraints
- Can be decompiled to .wat format using wasm2wat
- Not human-readable without reverse engineering

**box2d.wasm.js (282 KB - Glue Code)**
- JavaScript wrapper for WASM physics
- Loads and initializes WASM module
- Provides interface for creating bodies, applying forces
- Minified

**Physics Functions Called**:
```javascript
// Estimated physics API calls in main.js:
world.CreateBody(...)          // Create player/ball body
body.ApplyForce(force, point)  // Jump, move, shoot
body.SetLinearVelocity(...)    // Direct velocity control
world.Step(dt, ...)            // Physics tick
contact.GetFixtures()          // Collision detection
```

### Other Files

#### scripts/supportcheck.js
**Readable** - Checks browser capabilities
- WebGL support
- Canvas support
- Kaspersky antivirus detection
- Fallback handling

#### scripts/offlineclient.js
**Minified** - Offline/multi-tab support
- BroadcastChannel communication
- Message queueing
- State synchronization

#### scripts/register-sw.js
**Minified** - Service worker registration
- Registers sw.js
- Handles registration success/failure

#### js/ubg235_client_v1_1.js
**Readable** - External server integration
- Loads `https://www.ubg235.com/js/ubg235_server_v1_0.js` dynamically
- UBG235 is unblocked games platform
- Used for cross-domain game distribution

#### js/analytics_ubg_v1_4.js
**Readable** - Analytics with bot detection
```javascript
if (navigator.webdriver) {
  // Bot browser (Selenium, Puppeteer, etc.)
  loadGoogleAnalytics("G-LE1ZGTPC77");
} else {
  // Human browser
  loadGoogleAnalytics("G-E7D3EVY6HR");
}
```

---

## Game Mechanics (Inferred from Architecture)

### Player Control System
**Expected Controls**:
- Player 1: W (jump), A (left), D (right)
- Player 2: Up Arrow (jump), Left Arrow (left), Right Arrow (right)

**Physics**:
- Ragdoll bodies with gravity
- Impulse-based jumping
- Force-based movement
- Collision with ground prevents falling through

### Scoring
**Expected Logic**:
```
IF ball enters left basket:
  Player 2 scores +1 point
  Reset ball to center
  Show +1 animation

IF ball enters right basket:
  Player 1 scores +1 point
  Reset ball to center
  Show +1 animation

IF Player 1 score >= 5:
  Show "Player 1 Wins!"
  Game Over

IF Player 2 score >= 5:
  Show "Player 2 Wins!"
  Game Over
```

### Randomization
**What randomizes**:
- Player starting positions
- Ball starting position
- Court layout/variation
- Physics parameters (gravity, friction)
- Ball properties (size, weight)

**Why** ("Random" in title):
- Makes each match unique
- Adds chaos/unpredictability
- Prevents memorized strategies

---

## Decompilation Roadmap

### Step 1: Beautify main.js

**Command**:
```bash
npx js-beautify scripts/main.js -r -o scripts/main.beautified.js
```

**Expected Output Size**: ~200-300 KB (readable, formatted)

**What we'll see**:
- Class/object definitions
- Event handler functions
- Game logic (but with variable names like a, b, c)

### Step 2: Pattern Search

**Search for**:
- `keyCode`, `key`, `keyboard` → input handling
- `score`, `points`, `win` → scoring logic
- `collision`, `collide`, `overlap` → physics events
- `update`, `tick` → game loop
- `reset`, `restart` → game state
- `json`, `data` → data loading

### Step 3: Manual Variable Renaming

**Heuristics**:
```
IF variable appears in:
  - keyboard event → name it inputKey
  - score comparison → name it playerScore
  - collision → name it collidingObject
  - update loop → name it gameState/deltaTime
```

### Step 4: Extract Event System

**Look for**:
- addEventListener calls
- Event object patterns
- Callback functions
- Message passing

### Step 5: Reconstruct Logic

**Create pseudocode** version of discovered logic

---

## Expected Surprises

1. **Eval() usage**: Construct 3 may use eval() for dynamic behavior
2. **Obfuscated strings**: Important strings may be encoded
3. **Heavy recursion**: Event system may use deep call stacks
4. **Memory tricks**: Typed arrays for performance
5. **WASM communication**: Complex marshaling between JS and physics engine

---

## Success Criteria

✓ Main.js is beautified and searchable  
✓ Key functions are identified (init, update, keydown, collision)  
✓ Game state structure is understood  
✓ Control mappings are documented  
✓ Scoring logic is extracted  
✓ Pseudocode version of game logic exists  
✓ data.json structure is parsed  
✓ Physics calls are mapped  
✓ External dependencies are documented  
✓ Full game recreation is possible from analysis  

---

## Contact & Resources

**Original Repository**: https://github.com/Unblocked-Games-786/Basket-Random  
**Construct 3 Docs**: https://www.construct.net/en/make-games/manuals/construct-3  
**Box2D Docs**: https://box2d.org/  
**WASM Specs**: https://webassembly.org/
