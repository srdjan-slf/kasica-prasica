# Kasica Prasica Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build an arcade game where a piggy bank catches falling coins, avoids explosives, and shakes off devils.

**Architecture:** Single HTML file with embedded CSS and JavaScript. HTML5 Canvas for rendering, requestAnimationFrame game loop, emoji sprites, Web Audio API for sound. All game state in a single `game` object.

**Tech Stack:** HTML5, Canvas API, vanilla JavaScript, Web Audio API

---

### Task 1: Canvas Boilerplate + Game Loop

**Files:**
- Create: `index.html`

**Step 1: Create the HTML shell with Canvas**

Create `index.html` with:
- Full-viewport Canvas (800x600 logical, scaled to fit)
- Basic CSS: black background, centered canvas
- Game state object: `{ screen: 'title', score: 0, level: 1, cracks: 0 }`
- Game loop with `requestAnimationFrame` calling `update()` and `draw()`
- `draw()` renders a colored background and "Kasica Prasica" title text
- Keyboard input tracking object for arrow keys

```javascript
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const WIDTH = 800, HEIGHT = 600;

const keys = {};
window.addEventListener('keydown', e => keys[e.key] = true);
window.addEventListener('keyup', e => keys[e.key] = false);

const game = {
  screen: 'title', // 'title', 'playing', 'levelComplete', 'gameOver'
  score: 0,
  level: 1,
  cracks: 0,
  lastTime: 0
};

function update(dt) {}
function draw() {
  ctx.fillStyle = '#87CEEB';
  ctx.fillRect(0, 0, WIDTH, HEIGHT);
  ctx.fillStyle = '#333';
  ctx.font = '48px Arial';
  ctx.textAlign = 'center';
  ctx.fillText('🐷 Kasica Prasica 🐷', WIDTH/2, HEIGHT/3);
}

function loop(time) {
  const dt = (time - game.lastTime) / 1000;
  game.lastTime = time;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```

**Step 2: Open in browser to verify**

Open `index.html` in browser. Expected: blue sky background with "🐷 Kasica Prasica 🐷" title centered.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: canvas boilerplate with game loop and title screen"
```

---

### Task 2: Piggy Bank Player with Movement

**Files:**
- Modify: `index.html`

**Step 1: Add piggy bank player object and rendering**

Add player object and movement:

```javascript
const player = {
  x: WIDTH / 2,
  y: HEIGHT - 80,
  width: 60,
  height: 60,
  speed: 300,
  emoji: '🐷'
};
```

In `update(dt)`:
- If `keys.ArrowLeft` → move player left (`player.x -= player.speed * dt`)
- If `keys.ArrowRight` → move player right
- Clamp player.x between 0 and WIDTH - player.width

In `draw()`:
- Draw player emoji at player position using `ctx.font = '50px Arial'` and `ctx.fillText`
- Draw a simple "ground" line/rectangle at bottom

**Step 2: Verify in browser**

Expected: piggy emoji at bottom, moves left/right with arrow keys, stays within canvas.

**Step 3: Commit**

```bash
git commit -am "feat: piggy bank player with left/right movement"
```

---

### Task 3: Falling Coins + Collision Detection

**Files:**
- Modify: `index.html`

**Step 1: Add coin spawning and falling**

Add coins array and spawning system:

```javascript
const objects = []; // all falling objects
let spawnTimer = 0;
const SPAWN_INTERVAL = 1.0; // seconds between spawns

function spawnCoin() {
  objects.push({
    type: 'coin',
    emoji: '🪙',
    value: 1,
    x: Math.random() * (WIDTH - 30),
    y: -30,
    width: 30,
    height: 30,
    speed: 150
  });
}
```

In `update(dt)`:
- Increment spawnTimer, spawn coin when it exceeds SPAWN_INTERVAL
- Move all objects down: `obj.y += obj.speed * dt`
- Remove objects that fall below canvas
- Check collision with player (AABB rectangle overlap)
- On coin collision: add value to `game.score`, remove coin, play a small bounce animation

In `draw()`:
- Draw all falling objects as emoji
- Draw score in top-left: `"Score: " + game.score`

**Step 2: Verify in browser**

Expected: coins fall from top, catching them increases score displayed at top.

**Step 3: Commit**

```bash
git commit -am "feat: falling coins with collision detection and scoring"
```

---

### Task 4: Multiple Coin Types (Bills, Gold Bags)

**Files:**
- Modify: `index.html`

**Step 1: Add bill and gold bag types to spawning**

Modify `spawnCoin()` to randomly choose type:

```javascript
function spawnObject() {
  const roll = Math.random();
  let type, emoji, value;
  if (roll < 0.60) {
    type = 'coin'; emoji = '🪙'; value = 1;
  } else if (roll < 0.85) {
    type = 'bill'; emoji = '💵'; value = 5;
  } else {
    type = 'gold'; emoji = '💰'; value = 10;
  }
  objects.push({ type, emoji, value, x: Math.random()*(WIDTH-30), y:-30, width:30, height:30, speed:150 });
}
```

Adjust spawn rates based on `game.level`.

**Step 2: Verify in browser**

Expected: mix of 🪙💵💰 falling, different values added to score.

**Step 3: Commit**

```bash
git commit -am "feat: multiple coin types with different values"
```

---

### Task 5: Explosive Money 💣

**Files:**
- Modify: `index.html`

**Step 1: Add explosive type and damage logic**

Add bomb type to spawning (5-10% chance depending on level):

```javascript
// In spawnObject, add before the regular coin logic:
if (Math.random() < 0.05 + game.level * 0.02) {
  objects.push({
    type: 'bomb', emoji: '💣', value: 0,
    x: Math.random()*(WIDTH-30), y:-30, width:30, height:30, speed:180
  });
  return;
}
```

On bomb collision:
- `game.cracks++`
- `game.score = Math.max(0, game.score - 5)` (lose some coins)
- Screen flash red briefly (set a `flashTimer`)
- Explosion particle effect (expanding circle + emoji fragments)
- Check if `game.cracks >= 3` → game over

**Step 2: Add visual crack indicator**

Draw cracks on the HUD: show 🐷 with crack overlay based on `game.cracks` count.
- 0 cracks: 🐷 (happy)
- 1 crack: 🐷💔
- 2 cracks: 🐷💔💔
- 3 cracks: game over screen

**Step 3: Verify in browser**

Expected: bombs fall occasionally, catching one flashes red, adds crack, reduces score.

**Step 4: Commit**

```bash
git commit -am "feat: explosive money with crack system and visual feedback"
```

---

### Task 6: Devil Enemy - Falling + Latching

**Files:**
- Modify: `index.html`

**Step 1: Add devil entity that falls and latches**

```javascript
// Devil spawn (separate from regular objects)
function spawnDevil() {
  objects.push({
    type: 'devil',
    emoji: '😈',
    value: 0,
    x: Math.random() * (WIDTH - 40),
    y: -40,
    width: 40,
    height: 40,
    speed: 120,
    latched: false,
    hammerTimer: 0,
    shakeProgress: 0,
    shakeRequired: 8 // number of up/down presses needed
  });
}
```

On devil-player collision:
- Set `devil.latched = true`
- Devil snaps to sit on top of piggy bank
- Devil moves with player

**Step 2: Add latched devil rendering**

When devil is latched:
- Draw devil on top of piggy with hammer animation (alternating 🔨 position)
- Start `hammerTimer` countdown (5 seconds to shake off)

**Step 3: Verify in browser**

Expected: devil falls, touches piggy, sits on top and follows piggy movement.

**Step 4: Commit**

```bash
git commit -am "feat: devil enemy that falls and latches onto piggy bank"
```

---

### Task 7: Shake Mechanic - Eject Coin to Remove Devil

**Files:**
- Modify: `index.html`

**Step 1: Implement shake detection**

Track alternating up/down key presses:

```javascript
let lastShakeKey = null;
// In keydown handler:
if (devil.latched) {
  if (e.key === 'ArrowUp' && lastShakeKey !== 'up') {
    devil.shakeProgress++;
    lastShakeKey = 'up';
    // Visual: piggy jumps up slightly
  }
  if (e.key === 'ArrowDown' && lastShakeKey !== 'down') {
    devil.shakeProgress++;
    lastShakeKey = 'down';
    // Visual: piggy slams down
  }
}
```

**Step 2: Implement shake success - coin ejects devil**

When `shakeProgress >= shakeRequired`:
- Spawn a coin that flies UP from piggy toward devil
- Devil gets hit, flies off screen with spin animation
- `game.score -= 3` (lost the ejected coin)
- Devil removed from objects array

**Step 3: Implement shake failure - timeout damage**

In `update(dt)`, when devil is latched:
- Increment `hammerTimer`
- If `hammerTimer >= 5.0` (5 seconds): devil leaves, `game.cracks++`, `game.score -= 5`
- Screen shake effect during hammering

**Step 4: Add piggy squeal sound**

Generate squeal with Web Audio API oscillator:

```javascript
function playSqueal() {
  const actx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = actx.createOscillator();
  osc.type = 'square';
  osc.frequency.setValueAtTime(800, actx.currentTime);
  osc.frequency.linearRampToValueAtTime(1200, actx.currentTime + 0.1);
  osc.connect(actx.destination);
  osc.start(); osc.stop(actx.currentTime + 0.15);
}
```

Play squeal periodically while devil is hammering.

**Step 5: Add danger bar UI**

Draw a red bar above piggy showing `hammerTimer / 5.0` progress. Pulses when almost full.

**Step 6: Verify in browser**

Expected: devil latches, hammer animation plays with squeal sound, mash up/down to eject coin that knocks devil off, or take damage after 5 seconds.

**Step 7: Commit**

```bash
git commit -am "feat: shake mechanic with coin ejection, squeal sound, and danger bar"
```

---

### Task 8: Level System

**Files:**
- Modify: `index.html`

**Step 1: Add level configuration**

```javascript
function getLevelConfig(level) {
  return {
    duration: 30 + level * 5,       // seconds per level
    fallSpeed: 120 + level * 20,     // base fall speed
    spawnRate: Math.max(0.3, 1.0 - level * 0.1), // faster spawns
    devilCount: level,                // devils per level
    bombChance: 0.03 + level * 0.02, // bomb probability
    billChance: Math.min(0.3, 0.1 + level * 0.05)
  };
}
```

**Step 2: Add level timer and transitions**

- Add `game.levelTimer` that counts down
- When timer reaches 0 → show "Level Complete!" screen
- Show score for that level
- "Press SPACE to continue" → next level
- Reset `game.cracks = 0` on new level
- Schedule devil spawns spread across level duration

**Step 3: Add game over screen**

When `game.cracks >= 3`:
- `game.screen = 'gameOver'`
- Show "Game Over!" with final score and level reached
- "Press SPACE to restart"

**Step 4: Add title screen with "Press SPACE to start"**

**Step 5: Verify in browser**

Expected: title → press space → level 1 plays → level complete → level 2 (harder) → etc → game over shows final score.

**Step 6: Commit**

```bash
git commit -am "feat: level system with progression, transitions, and game over"
```

---

### Task 9: Visual Polish + Particles

**Files:**
- Modify: `index.html`

**Step 1: Add particle system**

```javascript
const particles = [];
function spawnParticles(x, y, emoji, count) {
  for (let i = 0; i < count; i++) {
    particles.push({
      x, y, emoji,
      vx: (Math.random()-0.5) * 200,
      vy: -Math.random() * 300,
      life: 1.0
    });
  }
}
```

Use particles for:
- Coin collection: small sparkle ✨ burst
- Bomb explosion: 💥 + fragments
- Devil knocked off: coin flies up, devil spins away
- Level complete: confetti 🎉

**Step 2: Add screen shake**

Screen shake on bomb hit and devil hammering (translate canvas context randomly).

**Step 3: Add background elements**

- Gradient sky background
- Simple ground with grass emoji 🌿
- Clouds drifting slowly
- Score counter with bounce animation when it changes

**Step 4: Add piggy bank visual states**

Draw piggy differently based on cracks:
- 0: 🐷 (happy, slight bounce idle animation)
- 1: 🐷 with drawn crack line
- 2: 🐷 with two crack lines, slight wobble
- 3: breaking animation → 💔 game over

**Step 5: Verify in browser**

Expected: polished visuals with particles, screen shake, animated piggy states.

**Step 6: Commit**

```bash
git commit -am "feat: visual polish with particles, screen shake, and animations"
```

---

### Task 10: Sound Effects + Final Touches

**Files:**
- Modify: `index.html`

**Step 1: Add all sound effects with Web Audio API**

Generate all sounds procedurally:
- Coin collect: short high-pitched "ding"
- Bill collect: double "ding ding"
- Gold collect: triumphant chord
- Bomb explosion: low rumble + noise burst
- Devil latch: ominous tone
- Piggy squeal: repeated while devil hammers
- Shake success: satisfying "whoosh" + "bonk"
- Level complete: ascending melody
- Game over: descending sad tones

**Step 2: Add HUD polish**

- Level indicator top-center
- Score with coin icon top-left
- Crack status top-right (3 piggy icons showing state)
- Level progress bar at top

**Step 3: Add difficulty scaling refinement**

Fine-tune level configs so:
- Level 1 is gentle tutorial (just coins, 1 slow devil)
- Level 3-4 feels challenging but fair
- Level 7+ is intense chaos

**Step 4: Final browser test**

Play through levels 1-5, verify all mechanics work together.

**Step 5: Commit**

```bash
git commit -am "feat: sound effects, HUD polish, and difficulty tuning"
```

---

## Summary

| Task | Feature | Verifiable Result |
|------|---------|-------------------|
| 1 | Canvas + game loop | Title screen renders |
| 2 | Piggy bank movement | Piggy moves left/right |
| 3 | Falling coins + collision | Catch coins, score increases |
| 4 | Multiple coin types | 🪙💵💰 with different values |
| 5 | Explosive money | 💣 adds cracks, loses coins |
| 6 | Devil falls + latches | 😈 sits on piggy |
| 7 | Shake mechanic | Up/down ejects coin to remove devil |
| 8 | Level system | Progressive difficulty, game over |
| 9 | Visual polish | Particles, animations, screen shake |
| 10 | Sound + final | Complete playable game |
