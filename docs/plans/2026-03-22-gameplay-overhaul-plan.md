# Gameplay Overhaul Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add jumping, horizontal ground enemies, poop mechanic, vegan ally, sky/clouds/lightning, and fix roast pig rotation.

**Architecture:** Single-file HTML/Canvas game. All changes in `kasica prasica.html`. No build system, no tests — verify visually in browser after each task.

**Tech Stack:** Vanilla JavaScript, Canvas 2D API, Web Audio API

---

### Task 1: Fix Roast Pig Rotation (3D spit effect)

**Files:**
- Modify: `kasica prasica.html:4292-4389` (drawGameOver roast pig section)

**Step 1: Replace rotation with 3D Y-axis spin**

Replace lines 4293-4295 (`ctx.rotate(rotAngle)`) with a Y-axis 3D simulation:

```javascript
// 3D Y-axis rotation simulation (spinning on the spit)
const rotAngle = t * 1.5;
const yScale = Math.cos(rotAngle); // Simulates 3D flip around spit axis
const flipSide = Math.sin(rotAngle) > 0; // Which side is facing us

ctx.save();
ctx.translate(spitCenterX, spitCenterY);
ctx.scale(1, yScale); // Compress vertically to simulate Y-axis rotation
```

**Step 2: Scale up the head and apple**

In the head section (~line 4322), change:
- Head radius: `18` → `24`
- Head position: `arc(50, 0, ...)` → `arc(55, 0, ...)`
- Snout: `ellipse(64, 2, 8, 6, ...)` → `ellipse(76, 3, 11, 8, ...)`
- Apple radius: `6` → `9`
- Apple position: `arc(70, 3, ...)` → `arc(85, 4, ...)`
- Apple stem and highlight repositioned accordingly

**Step 3: Verify**

Open game, lose all 3 lives, confirm:
- Pig spins around the horizontal spit axis (flips over, not rotates like a wheel)
- Head is noticeably larger
- Apple in mouth is bigger and visible

**Step 4: Commit**

```bash
git add "kasica prasica.html"
git commit -m "fix: roast pig spins on spit axis with bigger head and apple"
```

---

### Task 2: Jump Physics

**Files:**
- Modify: `kasica prasica.html:219-228` (player object)
- Modify: `kasica prasica.html:~1738` (startLevel reset)
- Modify: `kasica prasica.html:~2073-2160` (update playing section — player movement)
- Modify: `kasica prasica.html:~1751` (keydown handler for jump trigger)

**Step 1: Add physics properties to player object**

At line 228, add to the player object:

```javascript
const player = {
  x: WIDTH / 2,
  y: HEIGHT - 70,
  width: 70,
  height: 50,
  speed: 300,
  emoji: '\u{1F437}',
  facing: 0,
  facingSmooth: 0,
  // Jump physics
  vy: 0,              // Vertical velocity (negative = up)
  grounded: true,     // True when on ground
  groundY: HEIGHT - 70, // Ground level Y position
  jumpForce: -600,    // Initial jump velocity
  gravity: 1200,      // Normal gravity
  slamGravity: 3600   // Fast-fall gravity (down arrow in air)
};
```

**Step 2: Add jump trigger in keydown handler**

Find where ArrowUp/ArrowDown are handled for shaking (around line 1751). Add BEFORE the shake logic:

```javascript
// Jump trigger (only when grounded and no latched devil doing shake)
if (e.key === 'ArrowUp' && player.grounded && !game.latchedDevil) {
  player.vy = player.jumpForce;
  player.grounded = false;
}
```

**Step 3: Add gravity and landing in update()**

In the playing update section (after facing smooth update, around line 2155), add:

```javascript
// Jump physics
if (!player.grounded) {
  // Slam: if pressing down while in air, triple gravity
  const grav = keys.ArrowDown ? player.slamGravity : player.gravity;
  player.vy += grav * dt;
  player.y += player.vy * dt;

  // Landing
  if (player.y >= player.groundY) {
    player.y = player.groundY;
    player.vy = 0;
    player.grounded = true;
  }
} else {
  player.y = player.groundY; // Stay on ground
}
```

**Step 4: Reset jump state in startLevel()**

In startLevel() (~line 1738), add:

```javascript
player.vy = 0;
player.grounded = true;
player.y = player.groundY;
```

**Step 5: Update drawPiggy for airborne state**

In drawPiggySprite, when `!player.grounded`, tuck the legs slightly (scale leg height to 60%).

**Step 6: Verify**

Open game, press UP arrow — piggy should jump ~140px. Press DOWN while airborne — should slam down fast. Shake mechanic (when devil latched, on ground) should still work with alternating up/down.

**Step 7: Commit**

```bash
git add "kasica prasica.html"
git commit -m "feat: jump physics with up-arrow and slam with down-arrow"
```

---

### Task 3: Sky, Clouds, Sun Background

**Files:**
- Modify: `kasica prasica.html` — find `drawBackground()` or background drawing section
- Add new cloud state variables near line 230

**Step 1: Add cloud state**

```javascript
// --- Sky & Weather ---
const skyClouds = [];
for (let i = 0; i < 5; i++) {
  skyClouds.push({
    x: Math.random() * WIDTH,
    y: 30 + Math.random() * 80,
    w: 80 + Math.random() * 60,
    speed: 10 + Math.random() * 20,
    opacity: 0.5 + Math.random() * 0.3
  });
}
```

**Step 2: Draw sky gradient, clouds, and sun in background**

Find existing background drawing. Add AFTER existing sky gradient but BEFORE ground:

```javascript
// Draw fluffy clouds
for (const cloud of skyClouds) {
  ctx.fillStyle = `rgba(255, 255, 255, ${cloud.opacity})`;
  // Cloud = 3 overlapping circles
  ctx.beginPath();
  ctx.arc(cloud.x, cloud.y, cloud.w * 0.25, 0, Math.PI * 2);
  ctx.arc(cloud.x + cloud.w * 0.25, cloud.y - cloud.w * 0.12, cloud.w * 0.3, 0, Math.PI * 2);
  ctx.arc(cloud.x + cloud.w * 0.5, cloud.y, cloud.w * 0.22, 0, Math.PI * 2);
  ctx.fill();
}

// Sun in top-right corner
ctx.fillStyle = '#FFD700';
ctx.beginPath();
ctx.arc(WIDTH - 60, 50, 30, 0, Math.PI * 2);
ctx.fill();
// Sun rays
ctx.strokeStyle = '#FFD700';
ctx.lineWidth = 2;
for (let i = 0; i < 8; i++) {
  const angle = (i / 8) * Math.PI * 2 + Date.now() / 3000;
  ctx.beginPath();
  ctx.moveTo(WIDTH - 60 + Math.cos(angle) * 35, 50 + Math.sin(angle) * 35);
  ctx.lineTo(WIDTH - 60 + Math.cos(angle) * 48, 50 + Math.sin(angle) * 48);
  ctx.stroke();
}
```

**Step 3: Update clouds in update()**

```javascript
// Update clouds
for (const cloud of skyClouds) {
  cloud.x += cloud.speed * dt;
  if (cloud.x > WIDTH + cloud.w) cloud.x = -cloud.w;
}
```

**Step 4: Verify & Commit**

```bash
git commit -am "feat: sky background with moving clouds and sun"
```

---

### Task 4: Lightning System (Level 4+)

**Files:**
- Modify: `kasica prasica.html` — add lightning state, spawn logic, drawing, collision

**Step 1: Add lightning state variables**

```javascript
// --- Lightning ---
const lightningBolts = [];
let lightningWarning = null; // {x, timer} — cloud flash before strike
let lightningSpawnTimer = 0;
```

**Step 2: Lightning spawn logic in update()**

Only for level 4+. Every 5-8 seconds, pick random cloud, flash warning for 1 second, then spawn bolt:

```javascript
if (game.level >= 4) {
  lightningSpawnTimer -= dt;
  if (lightningSpawnTimer <= 0) {
    lightningSpawnTimer = 5 + Math.random() * 3;
    // Pick random cloud for warning
    const cloud = skyClouds[Math.floor(Math.random() * skyClouds.length)];
    lightningWarning = { x: cloud.x + cloud.w * 0.25, y: cloud.y + 20, timer: 1.0 };
  }

  if (lightningWarning) {
    lightningWarning.timer -= dt;
    if (lightningWarning.timer <= 0) {
      // Spawn actual bolt
      lightningBolts.push({
        x: lightningWarning.x,
        y: lightningWarning.y,
        width: 20,
        height: 30,
        speed: 800,
        segments: generateLightningSegments(lightningWarning.x, lightningWarning.y)
      });
      lightningWarning = null;
      triggerScreenShake(5, 0.2);
    }
  }
}
```

**Step 3: Lightning movement, collision, and drawing**

Lightning falls at 800 px/s with jagged segments. If it hits player, triggerDamage(true).

**Step 4: Draw warning flash on cloud**

When lightningWarning active: pulsing yellow glow on the source cloud.

**Step 5: Verify & Commit**

```bash
git commit -am "feat: lightning strikes from clouds at level 4+"
```

---

### Task 5: Horizontal Ground Enemies

**Files:**
- Modify: `kasica prasica.html` — add groundEnemies array, spawn logic, AI, collision, drawing

**Step 1: Add ground enemy state**

```javascript
// --- Ground Enemies ---
const groundEnemies = [];
let groundEnemySpawnTimer = 0;

const GROUND_ENEMY_TYPES = {
  wolf: {
    emoji: '🐺', name: 'Vuk', speed: 350, ai: 'straight',
    fromLevel: 3, damage: 1
  },
  carnivore: {
    emoji: '🍖', name: 'Mesožder', speed: 200, ai: 'chase',
    fromLevel: 4, damage: 1
  },
  fox: {
    emoji: '🦊', name: 'Lisica', speed: 280, ai: 'cunning',
    fromLevel: 5, damage: 1
  },
  farmer: {
    emoji: '👨‍🌾', name: 'Farmer', speed: 150, ai: 'relentless',
    fromLevel: 6, damage: 1
  }
};
```

**Step 2: Spawn logic**

Every 3-6 seconds (faster at higher levels), spawn ground enemy from left or right edge:

```javascript
if (game.level >= 3) {
  groundEnemySpawnTimer -= dt;
  if (groundEnemySpawnTimer <= 0) {
    groundEnemySpawnTimer = Math.max(1.5, 4 - game.level * 0.3);
    // Pick available type based on level
    const available = Object.entries(GROUND_ENEMY_TYPES)
      .filter(([_, t]) => game.level >= t.fromLevel);
    const [key, type] = available[Math.floor(Math.random() * available.length)];
    const fromLeft = Math.random() > 0.5;
    groundEnemies.push({
      type: key,
      emoji: type.emoji,
      x: fromLeft ? -30 : WIDTH + 30,
      y: HEIGHT - 45, // Ground level
      width: 40,
      height: 40,
      speed: type.speed,
      dir: fromLeft ? 1 : -1,
      ai: type.ai,
      // Cunning fox state
      pauseTimer: 0,
      pauseDuration: 0,
      reversed: false
    });
  }
}
```

**Step 3: AI movement per type**

```javascript
for (let i = groundEnemies.length - 1; i >= 0; i--) {
  const e = groundEnemies[i];
  switch (e.ai) {
    case 'straight': // Wolf — runs straight
      e.x += e.speed * e.dir * dt;
      break;
    case 'chase': // Carnivore — changes direction toward piggy
      e.dir = player.x > e.x ? 1 : -1;
      e.x += e.speed * e.dir * dt;
      break;
    case 'cunning': // Fox — runs, pauses, reverses
      if (e.pauseTimer > 0) {
        e.pauseTimer -= dt;
      } else {
        e.x += e.speed * e.dir * dt;
        // Random pause and reverse
        if (Math.random() < 0.005) {
          e.pauseTimer = 0.3 + Math.random() * 0.5;
          e.dir *= -1;
        }
      }
      break;
    case 'relentless': // Farmer — slow but always follows
      e.dir = player.x > e.x ? 1 : -1;
      e.x += e.speed * e.dir * dt;
      break;
  }
  // Remove if off-screen (only straight runners)
  if (e.ai === 'straight' && (e.x < -60 || e.x > WIDTH + 60)) {
    groundEnemies.splice(i, 1);
    continue;
  }
  // Collision with player (only if player is on ground or low enough)
  if (player.y + player.height / 2 > e.y - e.height / 2 && aabb(player, e)) {
    triggerDamage(false);
    groundEnemies.splice(i, 1);
  }
}
```

**Step 4: Draw ground enemies**

Each enemy drawn as emoji at their position, flipped based on direction.

**Step 5: Reset in startLevel()**

```javascript
groundEnemies.length = 0;
groundEnemySpawnTimer = 3; // Initial delay
```

**Step 6: Verify & Commit**

Open game, reach level 3. Wolf should run across screen. Jump over it to survive.

```bash
git commit -am "feat: horizontal ground enemies with unique AI per type"
```

---

### Task 6: Poop Mechanic (LEFT + RIGHT simultaneously)

**Files:**
- Modify: `kasica prasica.html` — add poop state, activation, drawing, ground enemy collision

**Step 1: Add poop state**

```javascript
// --- Poop Mechanic ---
const poops = [];
let poopCooldown = 0;
const POOP_COOLDOWN = 8; // seconds
const POOP_DURATION = 10; // seconds poop stays on ground
let isPooping = false;
let poopAnimTimer = 0;
```

**Step 2: Detect LEFT+RIGHT simultaneous press**

In update(), check:
```javascript
// Poop: both LEFT and RIGHT held simultaneously
if (keys.ArrowLeft && keys.ArrowRight && player.grounded && poopCooldown <= 0) {
  if (!isPooping) {
    isPooping = true;
    poopAnimTimer = 0.6; // squat animation duration
  }
}

if (isPooping) {
  poopAnimTimer -= dt;
  if (poopAnimTimer <= 0) {
    isPooping = false;
    poopCooldown = POOP_COOLDOWN;
    poops.push({
      x: player.x,
      y: HEIGHT - 35,
      width: 30,
      height: 25,
      life: POOP_DURATION,
      stinkTimer: 0
    });
    playPoopSound(); // squelch sound
    spawnParticles(player.x, HEIGHT - 35, '💩', 3, 50);
    triggerScreenShake(4, 0.2);
  }
}
poopCooldown = Math.max(0, poopCooldown - dt);
```

**Step 3: Poop collision with ground enemies**

```javascript
for (let p = poops.length - 1; p >= 0; p--) {
  const poop = poops[p];
  poop.life -= dt;
  poop.stinkTimer += dt;
  if (poop.life <= 0) { poops.splice(p, 1); continue; }

  // Check collision with ground enemies
  for (let e = groundEnemies.length - 1; e >= 0; e--) {
    if (aabb(poop, groundEnemies[e])) {
      // Enemy dies in agony
      const enemy = groundEnemies[e];
      chokingDevils.push({
        emoji: enemy.emoji,
        x: enemy.x, y: enemy.y,
        timer: 2.0, coughTimer: 0,
        opacity: 1.0, greenTint: 1.0
      });
      groundEnemies.splice(e, 1);
      playChokingSound();
      spawnParticles(enemy.x, enemy.y, '🤢', 4, 100);
      spawnParticles(enemy.x, enemy.y, '💩', 2, 60);
      spawnScorePopup(enemy.x, enemy.y, '💀💩', '#885500');
    }
  }
}
```

**Step 4: Draw poop with stink waves**

```javascript
function drawPoop(poop) {
  ctx.font = '28px sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText('💩', poop.x, poop.y);
  // Green stink waves
  const wave = Math.sin(poop.stinkTimer * 3) * 0.3 + 0.5;
  ctx.strokeStyle = `rgba(100, 200, 50, ${wave * (poop.life / POOP_DURATION)})`;
  ctx.lineWidth = 2;
  for (let i = 0; i < 3; i++) {
    ctx.beginPath();
    const offset = Math.sin(poop.stinkTimer * 2 + i * 2) * 10;
    ctx.moveTo(poop.x - 15 + offset, poop.y - 10 - i * 12);
    ctx.quadraticCurveTo(poop.x + offset, poop.y - 20 - i * 12, poop.x + 15 + offset, poop.y - 10 - i * 12);
    ctx.stroke();
  }
}
```

**Step 5: Pooping animation on piggy**

When `isPooping`: piggy squats (lower y by 10px), turns red, shakes.

**Step 6: Mobile touch - add dedicated poop button or detect both arrows**

For mobile: add a 💩 button between the existing controls, OR detect simultaneous touch of left+right buttons.

**Step 7: Verify & Commit**

```bash
git commit -am "feat: poop mechanic - hold left+right to drop 💩 that kills ground enemies"
```

---

### Task 7: Vegan Ally NPC

**Files:**
- Modify: `kasica prasica.html` — add vegan state, AI, drawing, interactions

**Step 1: Add vegan state**

```javascript
// --- Vegan Ally ---
let vegan = null; // Only one active at a time
let veganSpawnChance = 0.1; // 10% per level
let veganDeathAnim = null; // Angel rising animation
```

**Step 2: Spawn at level start (10% chance, level 2+)**

In startLevel(), after resetting:
```javascript
vegan = null;
veganDeathAnim = null;
if (game.level >= 2 && Math.random() < veganSpawnChance) {
  vegan = {
    x: WIDTH / 2 + (Math.random() - 0.5) * 200,
    y: HEIGHT - 50,
    width: 40,
    height: 50,
    hp: 3,
    maxHp: 3,
    dir: 1,
    speed: 80,
    state: 'patrol', // 'patrol', 'defend', 'dead'
    emoji: '🌿',
    blockTimer: 0,
    auraTimer: 0
  };
}
```

**Step 3: Vegan AI**

```javascript
if (vegan && vegan.state !== 'dead') {
  vegan.auraTimer += dt;

  // Check for nearby meat-eating threats
  let nearestThreat = null;
  let nearestDist = 200; // Protection radius
  for (const e of groundEnemies) {
    if (e.type === 'carnivore' || e.type === 'farmer') {
      const dist = Math.abs(e.x - player.x);
      if (dist < nearestDist) {
        nearestDist = dist;
        nearestThreat = e;
      }
    }
  }

  if (nearestThreat) {
    vegan.state = 'defend';
    // Run toward threat
    vegan.dir = nearestThreat.x > vegan.x ? 1 : -1;
    vegan.x += vegan.speed * 2 * vegan.dir * dt;
    // Block collision
    if (Math.abs(vegan.x - nearestThreat.x) < 30) {
      vegan.blockTimer -= dt;
      if (vegan.blockTimer <= 0) {
        vegan.hp--;
        vegan.blockTimer = 1.5;
        spawnParticles(vegan.x, vegan.y, '✌️', 3, 80);
        // Remove the blocked enemy
        const idx = groundEnemies.indexOf(nearestThreat);
        if (idx >= 0) groundEnemies.splice(idx, 1);
        if (vegan.hp <= 0) killVegan('enemy');
      }
    }
  } else {
    vegan.state = 'patrol';
    // Gentle patrol near player
    const targetX = player.x + Math.sin(Date.now() / 1000) * 100;
    vegan.dir = targetX > vegan.x ? 1 : -1;
    vegan.x += vegan.speed * vegan.dir * dt;
  }

  // Clamp to screen
  vegan.x = Math.max(30, Math.min(WIDTH - 30, vegan.x));

  // Check poop collision
  for (const poop of poops) {
    if (aabb(vegan, poop)) {
      killVegan('poop');
      break;
    }
  }

  // Check bomb/devil collision (from falling objects)
  for (let i = objects.length - 1; i >= 0; i--) {
    const obj = objects[i];
    if ((obj.type === 'bomb' || obj.type === 'devil') && aabb(vegan, obj)) {
      killVegan(obj.type === 'bomb' ? 'bomb' : 'devil');
      objects.splice(i, 1);
      break;
    }
  }
}
```

**Step 4: killVegan function**

```javascript
function killVegan(cause) {
  if (!vegan) return;
  vegan.state = 'dead';
  const msg = cause === 'poop'
    ? 'O ne, USRAO si vegana! 💩😇'
    : 'O ne, ubio si vegana! 😇';
  veganDeathAnim = {
    x: vegan.x,
    y: vegan.y,
    timer: 4.0,
    targetY: -50, // Rise to sky
    msg: msg,
    msgTimer: 3.0,
    wingSpread: 0,
    haloSize: 0
  };
  vegan = null;
  playVeganDeathSound(); // Sad chord progression
  spawnParticles(veganDeathAnim.x, veganDeathAnim.y, '😇', 5, 100);
}
```

**Step 5: Draw vegan**

Green aura pulsing, peace sign emoji, HP bar above head.

**Step 6: Draw vegan death animation**

Angel wings growing, halo appearing, rising to sky, sad message text.

**Step 7: Verify & Commit**

```bash
git commit -am "feat: vegan ally NPC that defends piggy from meat-eaters"
```

---

### Task 8: Integration, Mobile Controls, Polish

**Files:**
- Modify: `kasica prasica.html` — touch controls, level config updates, startLevel resets

**Step 1: Update mobile controls**

Add 💩 poop button to touch controls layout. Update layout:
```
◀  💨  💩  ▲▼  ▶
```

OR: detect simultaneous touch of ◀ and ▶ to trigger poop (more elegant, fewer buttons).

**Step 2: Update level config**

Ensure levels 3+ spawn ground enemies, 4+ have lightning. Update `getLevelConfig()`.

**Step 3: Reset all new state in startLevel()**

```javascript
// In startLevel():
groundEnemies.length = 0;
groundEnemySpawnTimer = 3;
poops.length = 0;
poopCooldown = 0;
isPooping = false;
lightningBolts.length = 0;
lightningWarning = null;
lightningSpawnTimer = 5;
vegan = null;
veganDeathAnim = null;
player.vy = 0;
player.grounded = true;
player.y = player.groundY;
```

**Step 4: Update index.html**

```bash
cp "kasica prasica.html" index.html
```

**Step 5: Final verify & push**

Play through levels 1-5, verify:
- L1-2: Normal gameplay with jump
- L3: Wolf enemies appear, can jump over them
- L4: Carnivore chases, lightning strikes
- L5: Fox appears
- Poop kills ground enemies
- Vegan appears sometimes, defends, can die

```bash
git add "kasica prasica.html" index.html
git commit -m "feat: complete gameplay overhaul - jump, ground enemies, poop, vegan, lightning"
git push
```

---

**Execution order matters:** Tasks 1-4 are independent of each other but Tasks 5-7 depend on Task 2 (jump physics) and Task 3 (sky for vegan death). Task 8 depends on all others.

**Parallelizable:** Tasks 1, 3, 4 can run in parallel. Task 2 should go first as others depend on its physics.
