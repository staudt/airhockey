# Air Hockey Game - Development Documentation

## Project Overview

This is a browser-based air hockey game built with vanilla JavaScript and HTML5 Canvas. The game features player vs adaptive CPU gameplay, power-up bonuses, and works equally well on desktop (mouse) and mobile (touch).

**Key Features:**
- 3rd person overhead view of air hockey rink
- Click/tap disk to take control, move mouse/finger to control paddle
- Adaptive CPU AI that adjusts to player skill
- First to 7 goals wins
- Power-up bonuses: multi-disk, speed changes, size changes, shields
- Gradient and glow visual effects
- Realistic physics with energy loss on bounces
- Responsive design for desktop and mobile

## Technology Stack

- **HTML5** - Structure and canvas element
- **CSS3** - Styling and UI overlay
- **JavaScript (ES6)** - All game logic
- **Canvas 2D API** - Rendering

## File Structure

The project consists of 2 files:

1. **[index.html](index.html)** - Complete game implementation (~1000 lines)
   - HTML structure with canvas and UI overlay
   - Embedded CSS for styling
   - All JavaScript game code

2. **[CLAUDE.md](CLAUDE.md)** - This documentation file

## Architecture

### Code Organization in index.html

The JavaScript code is organized into clear sections:

1. **Configuration Constants** (lines ~110-180)
   - `CONFIG` object with all tunable parameters
   - `BONUS_TYPES` enum
   - `GAME_STATES` enum

2. **Utility Classes** (lines ~185-250)
   - `Vector2D` - 2D vector math operations

3. **Game Objects** (lines ~255-380)
   - `Disk` - Hockey disk with position, velocity, physics
   - `Paddle` - Player/CPU paddle with boundaries
   - `Bonus` - Power-up boxes

4. **Bonus Effect Classes** (lines ~385-550)
   - `BonusEffect` - Base class
   - `MultiDiskEffect` - Spawns extra disks
   - `SpeedEffect` - Changes disk speed
   - `SizeEffect` - Changes paddle/disk size
   - `ShieldEffect` - Blocks goals

5. **Physics Engine** (lines ~555-670)
   - Wall collision detection
   - Paddle-disk collision detection
   - Collision resolution with momentum transfer

6. **AI Controller** (lines ~675-780)
   - Disk tracking and prediction
   - Adaptive difficulty algorithm
   - Skill-based adjustment

7. **Bonus System** (lines ~785-920)
   - Bonus spawning logic
   - Collision detection with disks
   - Effect activation and management

8. **Input Handler** (lines ~925-1050)
   - Unified mouse/touch event handling
   - Velocity tracking
   - Adaptive max speed detection

9. **Renderer** (lines ~1055-1250)
   - Canvas drawing with gradients
   - Glow effects
   - UI rendering

10. **Game State Manager** (lines ~1255-1520)
    - Game loop
    - State machine
    - Score tracking
    - Goal detection

11. **Game Loop** (lines ~1525-1570)
    - Fixed timestep loop
    - Update/render separation

12. **Initialization** (lines ~1575-end)
    - Canvas setup
    - Game instantiation
    - Start loop

## Key Systems

### 1. Physics System

**Constants:**
- `friction: 0.98` - Applied each frame to slow disk
- `wallRestitution: 0.85` - Energy loss on wall bounce (15%)
- `paddleRestitution: 1.0` - Perfect elastic collision with paddle
- `maxDiskSpeed: 15` - Speed cap for disk

**Collision Detection:**
- Circle-circle collision for paddle-disk (distance < r1 + r2)
- AABB for wall collisions with goal area exceptions
- Collision resolution uses impulse-based physics

**Momentum Transfer:**
```javascript
// Disk gains velocity from paddle movement
disk.velocity += impulseScalar * collisionNormal;
disk.velocity += paddle.velocity * 0.5; // 50% paddle influence
```

**Key Code Locations:**
- Wall collision: [index.html:590-650](index.html:590-650)
- Paddle collision: [index.html:652-670](index.html:652-670)
- Collision resolution: [index.html:672-705](index.html:672-705)

### 2. AI System (Adaptive Difficulty)

**How it Works:**
The AI tracks player performance and adjusts its difficulty to stay challenging but not overwhelming.

**Parameters (0-1 scale):**
- `difficulty`: Master difficulty level (starts at 0.5)
- `reactionSpeed`: How fast AI responds (0.3 to 1.0)
- `predictionAccuracy`: How well AI predicts disk path (0.4 to 1.0)
- `errorMargin`: Pixels of positioning error (60 to 0)
- `reactionDelay`: Milliseconds before responding (210 to 0)

**Adaptation Algorithm:**
```javascript
// When player scores: difficulty += 0.05
// When CPU scores: difficulty -= 0.05
// Clamped between 0.1 and 0.95
```

**Behavior:**
- Finds most threatening disk (moving toward CPU)
- Predicts future position with wall bounces
- Adds random error based on difficulty
- Returns to center when no threat

**Key Code Locations:**
- AI update: [index.html:710-740](index.html:710-740)
- Prediction: [index.html:760-780](index.html:760-780)
- Adaptation: [index.html:782-790](index.html:782-790)

### 3. Bonus System

**Spawning:**
- Every 10 seconds (configurable)
- Max 2 bonuses on field at once
- Random position in safe zones (away from center)
- Weighted random type selection

**Bonus Types:**

1. **Multi-Disk** (15% weight)
   - Spawns 2-3 extra disks from sides
   - Disks removed when effect expires

2. **Speed Boost** (20% weight)
   - Multiplies all disk velocities by 1.5x

3. **Speed Slow** (20% weight)
   - Multiplies all disk velocities by 0.5x

4. **Large Paddle** (15% weight)
   - Increases paddle size by 1.5x for hitting player

5. **Small Paddle** (10% weight)
   - Decreases opponent paddle to 0.6x

6. **Large Disk** (10% weight)
   - Increases all disk sizes by 1.5x

7. **Small Disk** (10% weight)
   - Decreases all disk sizes to 0.6x

8. **Shield** (10% weight)
   - Blocks goals for hitting player

**Effect Duration:**
- Random between 5-8 seconds
- Countdown displayed in UI
- Original values restored on expiration

**Key Code Locations:**
- Spawning: [index.html:810-845](index.html:810-845)
- Activation: [index.html:865-910](index.html:865-910)
- Effect classes: [index.html:415-550](index.html:415-550)

### 4. Input System

**Control Activation:**
- Click/tap within disk radius + 40px buffer
- Sets `isActive = true`
- Paddle follows mouse/finger position

**Velocity Tracking:**
- Calculates position delta per frame
- Normalizes to 60fps baseline
- Tracks speed samples (last 20 frames)

**Adaptive Max Speed:**
- Takes top 10% of speed samples
- Averages them for max speed threshold
- Allows player skill to set speed ceiling

**Paddle Movement:**
- Lerps toward target position (15% per frame)
- Constrained to own half of rink
- Velocity calculated from position change

**Key Code Locations:**
- Pointer down: [index.html:950-970](index.html:950-970)
- Pointer move: [index.html:972-1010](index.html:972-1010)
- Position calculation: [index.html:1025-1045](index.html:1025-1045)

### 5. Game State Machine

**States:**
- `MENU` - Initial screen with instructions
- `COUNTDOWN` - 3-2-1-GO before serve
- `PLAYING` - Active gameplay
- `GOAL_SCORED` - 2 second pause after goal
- `GAME_OVER` - Someone reached 7 goals

**Flow:**
```
MENU → (click start) → COUNTDOWN → PLAYING ↔ GOAL_SCORED → GAME_OVER
```

**Update Logic:**
- Each state has its own update behavior
- Fixed timestep (16.67ms = 60 FPS)
- Accumulator prevents spiral of death

**Key Code Locations:**
- State changes: [index.html:1350-1380](index.html:1350-1380)
- Update loop: [index.html:1395-1480](index.html:1395-1480)
- Game loop: [index.html:1525-1570](index.html:1525-1570)

### 6. Rendering System

**Rendering Order:**
1. Clear canvas
2. Render rink (base color)
3. Render goals
4. Render bonuses
5. Render shields
6. Render paddles (with gradients/glows)
7. Render disks (with gradients/glows)
8. Render score

**Visual Effects:**
- Radial gradients on paddles and disks
- Shadow blur for glow effects
- Pulsing animation on bonuses
- Dashed center line

**Colors:**
- Background: `#0a1628` (dark blue)
- Rink: `#1a2940` (medium blue)
- Player: `#00ff88` (green)
- CPU: `#ff5555` (red)
- Disk: `#ffffff` (white)

**Key Code Locations:**
- Render loop: [index.html:1515-1550](index.html:1515-1550)
- Paddle rendering: [index.html:1140-1170](index.html:1140-1170)
- Glow effects: [index.html:1145-1150](index.html:1145-1150)

## Configuration & Tuning

All tunable parameters are in the `CONFIG` object at [index.html:112-180](index.html:112-180).

### Game Balance

**Making AI Harder:**
```javascript
ai: {
    startingDifficulty: 0.7,  // Start harder (default: 0.5)
    adaptationRate: 0.03,     // Adapt faster (default: 0.02)
    maxDifficulty: 1.0        // Allow perfect play (default: 0.95)
}
```

**Making AI Easier:**
```javascript
ai: {
    startingDifficulty: 0.3,  // Start easier
    minDifficulty: 0.05,      // Can get very easy (default: 0.1)
}
```

### Physics Tuning

**Faster Gameplay:**
```javascript
physics: {
    friction: 0.99,           // Less friction (default: 0.98)
    maxDiskSpeed: 20          // Higher cap (default: 15)
}
disk: {
    serveSpeed: 7             // Faster serves (default: 5)
}
```

**Slower Gameplay:**
```javascript
physics: {
    friction: 0.96,           // More friction
    maxDiskSpeed: 10,         // Lower cap
    wallRestitution: 0.7      // More energy loss (default: 0.85)
}
```

### Bonus Frequency

**More Bonuses:**
```javascript
bonus: {
    spawnInterval: 7000,      // Every 7s (default: 10000)
    maxConcurrent: 3,         // 3 at once (default: 2)
    effectDurationMax: 10000  // Longer effects (default: 8000)
}
```

**Fewer Bonuses:**
```javascript
bonus: {
    spawnInterval: 15000,     // Every 15s
    maxConcurrent: 1,         // Only 1 at a time
    effectDurationMin: 3000   // Shorter effects (default: 5000)
}
```

### Paddle Control

**More Responsive:**
```javascript
paddle: {
    lerpFactor: 0.25,         // Faster follow (default: 0.15)
    activationRadius: 60      // Easier to click (default: 40)
}
```

**Less Responsive:**
```javascript
paddle: {
    lerpFactor: 0.08,         // Slower follow
    activationRadius: 20      // Harder to click
}
```

## Testing & Verification

### Manual Testing Checklist

**Core Gameplay:**
- [x] Game starts with countdown
- [x] Disk serves from center in random direction
- [x] Player paddle follows mouse/touch
- [x] Player paddle cannot cross center line
- [x] CPU paddle stays in top half
- [x] Disk bounces realistically off walls
- [x] Disk bounces off paddles with velocity transfer
- [x] Goals are detected correctly
- [x] Score updates on goal
- [x] Game ends at 7 points

**Controls:**
- [x] Click on disk activates control
- [x] Mouse movement controls paddle smoothly
- [x] Touch on disk activates control
- [x] Touch movement controls paddle smoothly
- [x] Paddle speed affects disk velocity

**AI Behavior:**
- [x] CPU responds to disk movement
- [x] CPU attempts to block shots
- [x] CPU returns to center when no threat
- [x] AI difficulty increases when losing
- [x] AI difficulty decreases when winning

**Bonus System:**
- [ ] Bonuses spawn periodically (test: wait 10+ seconds)
- [ ] Bonuses can be hit by disk
- [ ] Multi-disk spawns 2-3 disks from sides
- [ ] Speed boost increases disk velocity
- [ ] Speed slow decreases disk velocity
- [ ] Size bonuses change paddle/disk size
- [ ] Shield blocks goals
- [ ] Effects expire after 5-8 seconds

**Visual Polish:**
- [x] Glow effects on paddles and disk
- [x] Smooth animations
- [x] Score displays correctly
- [x] UI is clear and readable
- [x] Bonus boxes have distinct colors
- [x] Shield effect is visible

### How to Test

1. **Open the game:**
   - Open `index.html` in a web browser
   - Chrome, Firefox, Safari, or Edge recommended

2. **Test basic gameplay:**
   - Click "Start Game"
   - Wait for countdown
   - Click the disk to take control
   - Move mouse to control paddle
   - Try to score goals

3. **Test AI adaptation:**
   - Play multiple games
   - Intentionally lose a game (AI should get easier)
   - Win a game (AI should get harder)

4. **Test bonuses:**
   - Play for 10+ seconds to see bonuses spawn
   - Hit bonuses with disk
   - Observe effects (check UI for countdown)
   - Test each bonus type

5. **Test mobile:**
   - Open on mobile device or use browser dev tools
   - Test touch controls
   - Verify responsiveness

### Known Issues

None currently identified. If you find issues:
1. Open browser console (F12) to check for errors
2. Verify CONFIG values are reasonable
3. Check if issue is reproducible

## Performance Notes

**Target:** 60 FPS on modern devices

**Optimization Strategies:**
- Fixed timestep game loop prevents physics issues
- Minimal DOM manipulation (only UI updates)
- Canvas redrawn each frame (no layering needed for this simple game)
- Collision checks limited to active objects only

**Performance Testing:**
- Monitor FPS in browser dev tools
- Test with multiple disks active (multi-disk bonus)
- Test with multiple bonuses on screen
- Verify smooth gameplay on mobile devices

## How to Resume Work

### Quick Start

1. **Read this documentation** to understand the architecture
2. **Open [index.html](index.html)** in your code editor
3. **Find the relevant section** using the line numbers in this doc
4. **Make changes** to the code
5. **Test in browser** by opening index.html

### Common Modifications

**Add a new bonus type:**
1. Add type to `BONUS_TYPES` enum (~165)
2. Create new effect class extending `BonusEffect` (~415-550)
3. Add weight to `bonusWeights` in BonusSystem (~800)
4. Add activation case in `activateBonus` (~865-910)
5. Add color in `getBonusColor` (~1230)

**Modify physics:**
1. Find `CONFIG.physics` object (~135)
2. Adjust values (friction, restitution, speeds)
3. Test in browser

**Change AI behavior:**
1. Find `AIController` class (~675)
2. Modify `update` or `predictDiskPosition` methods
3. Adjust `CONFIG.ai` values (~155)

**Update visuals:**
1. Find `Renderer` class (~1055)
2. Modify rendering methods
3. Adjust `CONFIG.visual` colors (~175)

### Development Workflow

1. **Make changes** to index.html
2. **Save file**
3. **Refresh browser** (Ctrl+R or Cmd+R)
4. **Test changes**
5. **Repeat**

No build process needed - it's all in one HTML file!

## Future Improvements

**Potential Enhancements:**
1. **Sound effects** - Add audio for hits, goals, bonuses
2. **Power-up visual effects** - Add particles when bonus activates
3. **Difficulty selector** - Let player choose AI difficulty
4. **Score tracking** - Save high scores to localStorage
5. **Two-player mode** - Both players control paddles
6. **More bonus types** - Curved shots, slow motion, paddle teleport
7. **Tournament mode** - Best of multiple games
8. **Replay system** - Record and replay goals
9. **Customization** - Choose paddle colors, rink themes
10. **Online multiplayer** - WebSockets for remote play

**Code Improvements:**
1. **Split into modules** - Separate files for better organization
2. **TypeScript** - Add type safety
3. **Unit tests** - Test physics calculations
4. **Performance profiling** - Optimize render loop
5. **Accessibility** - Keyboard controls, screen reader support

## Credits

Built with ❤️ using Claude Code (Sonnet 4.5)

**Technologies:**
- HTML5 Canvas
- Vanilla JavaScript (ES6)
- CSS3

**Game Design:**
- Classic air hockey mechanics
- Breakout/Arkanoid-inspired bonus system
- Adaptive AI difficulty concept

## License

Free to use and modify for any purpose.

---

**Last Updated:** 2026-02-09
**Version:** 1.0
