# Kasica Prasica 🐷 - Game Design

## Overview
Arcade-style game where the player controls a piggy bank (kasica prasica), catching falling coins while avoiding explosive money and devils with hammers.

## Technology
- Single HTML file with HTML5 Canvas
- Emoji-based sprites (no external assets)
- requestAnimationFrame game loop
- Web Audio API for sound effects (piggy squeal)

## Player - Piggy Bank
- Moves left/right with arrow keys
- 3-crack health system per level (healthy → 1 crack → 2 cracks → broken)
- Health resets between levels

## Falling Objects

| Object | Emoji | Effect |
|--------|-------|--------|
| Bronze coin | 🪙 | +1 point |
| Bill | 💵 | +5 points |
| Gold bag | 💰 | +10 points (rare) |
| Explosive money | 💣 | +1 crack, lose some coins |
| Devil with hammer | 😈🔨 | Latches onto piggy bank |

## Devil Mechanic
1. Devil falls from top of screen
2. On contact: latches onto piggy bank
3. While latched: hammer hits piggy, piggy squeals, danger bar fills
4. Player must rapidly press up/down arrows to shake
5. Shaking ejects a coin that knocks devil off (lose that coin's value)
6. If not shaken off in time: +1 crack, devil leaves on its own

## Level System
- **Level 1**: Coins only, slow falling speed, 1 devil near end
- **Level 2**: Coins + bills, faster, 2 devils
- **Level 3+**: Progressively faster, more devils and explosives
- Between levels: score screen, piggy repairs
- Game over: piggy breaks (3 cracks in one level)

## Controls
- ←/→: Move left/right
- ↑/↓: Shake (only when devil is latched)
