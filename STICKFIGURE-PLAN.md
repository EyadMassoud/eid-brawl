# Stick Figure Animation — Implementation Plan

## Overview

Two parallel workstreams to add animated stick figure fighting to Eid Brawl.

## Current State

Stick figures are **static SVG poses** defined as raw coordinate strings in `FIGHTERS[id].svg`. Each fighter has:
- 1 head circle (`r="10"`)
- 1 torso line
- 2 arm segments (upper + forearm) + fist circles (`r="6"`)
- 2 leg segments (upper + lower)

Rendered in `<svg class="sf-sil" viewBox="0 0 80 120">` elements inside the SF overlay. Currently sit BELOW the portrait cards in a vertical layout.

## Prerequisite: Joint-Based SVG System

Before either team can animate, the static SVGs must be restructured into a joint system:

```
FIGHTERS[id].joints = {
  head:      { x: 40, y: 10 },
  neck:      { x: 40, y: 20 },
  shoulder:  { x: 40, y: 28 },
  hip:       { x: 40, y: 58 },
  // Arms
  elbowL:   { x: 24, y: 36 },
  handL:    { x: 28, y: 18 },
  elbowR:   { x: 56, y: 36 },
  handR:    { x: 52, y: 18 },
  // Legs
  kneeL:    { x: 24, y: 86 },
  footL:    { x: 18, y: 116 },
  kneeR:    { x: 56, y: 86 },
  footR:    { x: 62, y: 116 }
}
```

Each SVG element gets a data attribute (`data-joint`) so JS can target it. Render function builds the SVG from joint positions dynamically.

## Team A: Normal Mode — Finishing Combo

### Scope
Enhance the existing tap battle with:
1. **Idle animation** — subtle body bob/sway during the first 20 taps
2. **Finishing combo** (at 20/30 taps, ~30% HP remaining):
   - Slow-mo visual effect starts
   - Each of the final ~10 taps triggers a choreographed attack pose
   - Sequence: jab → cross → hook → uppercut → roundhouse kick → final KO blow
   - Opponent stick figure recoils with each hit
3. **Layout change** — portraits become small thumbnails next to HP bars. Stick figures become big, center-stage, in a shared arena facing each other.

### Technical Approach
- **Hybrid animation**: Keyframe poses with CSS `transition` smoothing (100-150ms ease)
- Define 6-8 attack poses per fighter as joint coordinate sets
- On each tap during combo phase, snap joints to next attack pose
- Opponent joints snap to a hit-reaction pose
- Use `requestAnimationFrame` for smooth interpolation between poses

### Key Files/Lines to Modify
- CSS: `.sf-sil` sizing, `.sf-fighter` layout (flexbox → arena layout)
- JS: `FIGHTERS[id]` — add `joints` and `poses` data
- JS: `applyTap()` ~line 1200 — add combo animation trigger at threshold
- JS: New `animateJoints(side, poseName)` function
- HTML: `#sf-overlay` structure — reposition portraits as thumbnails

### Deliverable
Working tap battle where the final 10 taps play out as an animated stick figure combo.

---

## Team B: Classic Mode — Full Fighting Game

### Scope
Unlocked after 5 wins with any fighter (check `record[id].w >= 5` in localStorage).

1. **New game mode** with button controls:
   - Square = Punch (arm extends toward opponent)
   - Circle = Kick (leg swings)
   - Triangle = Defend (arms cross in front)
   - X = Taunt (hands wave, wind gesture SVG animation)
2. **Full arena**: No portraits, full-screen stick figure battle
3. **CPU AI**: Pattern-based move selection, block/counter logic
4. **Damage system**: Each move does specific damage, blocks reduce damage, combos multiply
5. **Health bars** at top of screen

### Technical Approach
- Reuse the joint system from Team A
- Each move = a sequence of joint keyframes played over 300-500ms
- Block detection: if defender is in block pose when hit arrives, reduce damage
- CPU AI: weighted random with simple pattern detection (if player spams kick, CPU learns to block low)

### Key Files/Lines to Modify
- CSS: New `.classic-overlay` fullscreen arena styles
- JS: New game mode entry point (check wins threshold)
- JS: Button input handling (touch + keyboard)
- JS: CPU AI move selection engine
- JS: Hit detection + damage calculation
- HTML: New overlay div with control buttons

### Deliverable
Complete classic fighting mode accessible after 5 wins.

---

## Coordination

Both teams share:
- The joint system data structure
- The `animateJoints()` rendering function
- The pose library (attack/idle/hit/block poses)

**Build order**: Joint system first (either team), then both can work in parallel.

## Session Start Checklist

1. Read `eid-tournament-final.html` — it's the only file
2. Read this plan + `EID-BRAWL-PLAN.md` for context
3. Check git log for recent changes
4. Build the joint system as prerequisite
5. Launch Team A and Team B as parallel agents
