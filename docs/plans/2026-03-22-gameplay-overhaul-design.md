# Gameplay Overhaul Design - 2026-03-22

## Overview
Major gameplay expansion: jumping physics, horizontal ground enemies, poop mechanic, vegan ally NPC, sky/weather system, and roast pig animation fix.

## 1. Jump Physics
- **UP arrow** = jump (velocity -600 px/s, gravity 1200 px/s²) — ~140px max height
- **DOWN arrow in air** = slam (gravity × 3 = 3600 px/s²) — ultra-fast landing
- **DOWN arrow on ground** = shake point (for latched devil/butcher — existing alternation mechanic preserved)
- Player gains `vy`, `grounded`, `groundY` properties
- Legs animate (tuck) when airborne

## 2. Horizontal Ground Enemies (level 3+)

| Enemy | Emoji | AI | Speed | From Level |
|-------|-------|----|-------|------------|
| Wolf (3 little pigs) | 🐺 | Runs straight, no direction change | 350 px/s | 3 |
| Carnivore (meat-eater) | 🍖👨 | Chases piggy, changes direction | 200 px/s | 4 |
| Fox | 🦊 | Cunning — runs, stops, reverses | 280 px/s | 5 |
| Farmer w/ pitchfork | 👨‍🌾 | Slow but relentless pursuit | 150 px/s | 6 |

- Enter from left or right edge
- Dodged by jumping over them
- Ground contact = lose 1 life
- Killed by poop on ground

## 3. Poop Mechanic (LEFT + RIGHT simultaneously)
- Piggy squats, turns red, drops 💩
- Poop stays on ground 10 seconds with green stink waves
- Cooldown: 8 seconds
- Kills horizontal enemies on contact (choking death animation)
- Kills vegan (triggers angel death scene)
- Mobile: dedicated button or both arrow buttons

## 4. Vegan Ally NPC
- 10% spawn chance per level (level 2+)
- Emoji: 🌿 with green aura, walks on ground
- 3 HP — blocks meat-eaters/butchers/farmers
- Each block costs 1 HP
- Vulnerable to: bombs (1-hit), devils (1-hit), poop (1-hit)
- Death animation: angel wings + halo, rises to sky, sad music
- Death by poop: "O ne, USRAO si vegana! 💩😇"
- Death by enemy: "O ne, ubio si vegana! 😇"

## 5. Sky, Clouds, Lightning
- Sky gradient background (blue) with 3-5 moving clouds
- Sun in corner (decorative)
- Lightning from level 4+: cloud flashes yellow 1s warning, then bolt drops at 800 px/s
- Lightning hit = 1 life lost

## 6. Roast Pig Fix
- Rotate around Y-axis (3D spit effect) using `ctx.scale(Math.cos(angle), 1)`
- Larger head and apple in mouth
- Better styled pig body
