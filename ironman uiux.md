# UIUX — Iron Man: Stark Protocol
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Complete visual, motion, and experience design spec  

---

## 1. Design Philosophy

This is not a game UI. It is a **living suit interface.**

Everything the user sees should feel like it belongs inside Tony Stark's HUD or his lab. No generic game menus. No health bars that look like mobile games. Every UI element should feel engineered by the most brilliant mind on the planet.

Core aesthetic: Stark Industries dark luxury. Deep blacks, Arc Reactor blue, clean geometric lines, holographic translucency.

Reference energy: Iron Man 1 workshop scenes × MCU HUD interfaces × Tron Legacy

---

## 2. Color Palette

| Name | Hex | Usage |
|---|---|---|
| Arc Reactor Blue | #00CFFF | Primary accent, energy effects, HUD lines |
| Deep Stark Black | #050A14 | Background, suit shadows |
| Hologram Blue | #0066FF | Holographic panels in lab |
| Danger Red | #FF2020 | Enemy indicators, low health, threat alerts |
| Charge Gold | #FFB700 | Unibeam charge, boss weak points |
| HUD White | #E8F4FF | Text, targeting reticles |
| Missile Red | #FF4444 | Missile lock targeting |
| Avengers Gold | #C9A84C | Avengers Easter egg accent |

---

## 3. Typography

| Element | Font | Size | Style |
|---|---|---|---|
| HUD data readouts | Orbitron (Google Fonts) | 12-14px | Uppercase, letter-spacing 0.2em |
| Level titles | Rajdhani (Google Fonts) | 32px | Bold, uppercase |
| Score display | Orbitron | 24px | Bold |
| Upgrade labels | Rajdhani | 16px | Medium |
| Warning text | Orbitron | 11px | Uppercase, flashing |

Load via: @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Rajdhani:wght@400;600;700&display=swap');

---

## 4. Start Screen

```
[Full black background]
[Subtle arc reactor glow pulse in center]

        IRON MAN
    STARK PROTOCOL

  [small arc reactor icon]

  GESTURE CONTROL ACTIVE
  position hands in frame to begin

  [ CLICK TO INITIALIZE ]
```

The start screen slowly reveals itself — text fades in line by line. The arc reactor icon pulses continuously. Clicking initializes audio context and transitions to lab.

---

## 5. Lab Mode Visual Design

### 5.1 Environment
- sci-fi_lab.glb rendered at full resolution
- Arc Reactor blue rim lighting on all surfaces
- Subtle dust particle system floating in the air
- Background hum audio — mechanical, low, intelligent

### 5.2 Holographic Panels

```css
.holo-panel {
  background: linear-gradient(135deg, rgba(0,102,255,0.08), rgba(0,207,255,0.04));
  border: 1px solid rgba(0,207,255,0.4);
  border-radius: 4px;
  box-shadow:
    0 0 20px rgba(0,207,255,0.15),
    inset 0 0 20px rgba(0,102,255,0.05);
  backdrop-filter: blur(8px);
  font-family: 'Orbitron', sans-serif;
  color: #E8F4FF;
}
```

Panels float with a gentle sine wave bob animation — never perfectly still.

### 5.3 Panel Layout in Lab

```
[LEFT PANEL]          [CENTER PANEL]         [RIGHT PANEL]
Suit Stats            Level Select           Upgrade Tree
- Health cap          Level 1-5 grid         Weapon upgrades
- Weapon loadout      (locked/unlocked)      Defense upgrades
- Upgrade points      Difficulty preview     HUD upgrades
```

### 5.4 Hand Interaction Visual Feedback
- Approaching a panel within range: panel brightens, border pulses
- Grabbing: panel scales up slightly, blue grip effect on hand overlay
- Dismissing: panel flies away with motion blur trail
- Selecting: panel expands, others dim

---

## 6. Suit Up Sequence Visual Design

Full cinematic. No HUD visible yet. Just the suit assembling.

### Assembly Order and Timing

| Second | Event |
|---|---|
| 0.0 | Screen subtly darkens |
| 0.5 | Boot pieces fly in from bottom, lock with spark effect |
| 1.5 | Shin guards slide up |
| 2.5 | Thigh armor clicks into place |
| 3.5 | Chest piece slams in — Arc Reactor PULSES blue white |
| 4.5 | Shoulder pieces lock |
| 5.0 | Left gauntlet slides on — hand overlay changes |
| 5.5 | Right gauntlet slides on |
| 6.5 | Helmet descends — screen goes completely dark |
| 7.0 | HUD boots up — scan lines, data floods in |
| 7.5 | Full HUD visible — combat environment revealed |
| 8.0 | First wave countdown begins |

### Arc Reactor Pulse
At second 3.5 when chest piece locks, the Arc Reactor activation is the visual centerpiece:

```js
// Radial blue white burst emanating from chest position
// Lights up entire scene briefly
// arcLight intensity spikes from 2 to 20 then back to 2 over 0.5 seconds
```

---

## 7. Combat HUD Design

The HUD is drawn on a transparent canvas overlay (z-index 100) on top of the Three.js canvas.

### 7.1 HUD Layout

```
┌─────────────────────────────────────────────────────────────┐
│  ◈ STARK PROTOCOL          LEVEL 2 — CITY SIEGE    SCORE: 4200 │
│                                                               │
│  [RADAR - top left]                    [THREAT ALERTS - top right] │
│                                                               │
│         [ THREE.JS COMBAT SCENE ]                            │
│                                                               │
│  ████████████ HEALTH 87/100          WEAPON: REPULSOR        │
│  ░░░░░░░░░░░░ UNIBEAM COOLDOWN       MISSILES: 3/5           │
│  [HAND SKELETON OVERLAYS ON WEBCAM - bottom right corner]    │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 Health Bar

```css
.health-bar {
  position: fixed;
  bottom: 60px;
  left: 40px;
  width: 220px;
  height: 6px;
  background: rgba(255,255,255,0.1);
  border: 1px solid rgba(0,207,255,0.4);
}

.health-fill {
  height: 100%;
  background: linear-gradient(90deg, #00CFFF, #0066FF);
  transition: width 0.3s ease;
  box-shadow: 0 0 10px rgba(0,207,255,0.6);
}

/* Below 30% health */
.health-fill.critical {
  background: linear-gradient(90deg, #FF2020, #FF6600);
  animation: criticalPulse 0.5s ease-in-out infinite alternate;
}
```

### 7.3 Unibeam Cooldown Arc

Circular arc around the right hand overlay. Fills clockwise as cooldown recharges. Gold color. Only visible when unibeam recently used.

### 7.4 Enemy Radar

Top left corner. 80x80px circular radar. Dots representing enemies — red for basic, orange for heavy, white for jets, yellow for turrets. Player at center. Updates every 100ms.

### 7.5 Targeting Reticle

When missile lock gesture detected:

```js
// Red brackets appear around nearest enemy
// Bracket corners animate inward as lock progresses
// Lock complete: brackets flash white, tone plays
// Locked indicator appears above enemy health bar
```

### 7.6 Damage Flash

When player takes damage:

```css
.damage-overlay {
  position: fixed;
  inset: 0;
  background: radial-gradient(ellipse at center, transparent 40%, rgba(255,0,0,0.3) 100%);
  opacity: 0;
  pointer-events: none;
  animation: damageFlash 0.3s ease-out;
}

@keyframes damageFlash {
  0% { opacity: 1; }
  100% { opacity: 0; }
}
```

### 7.7 Webcam Preview with Skeleton

```css
#webcam-container {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 200px;
  height: 150px;
  border: 1px solid rgba(0,207,255,0.4);
  border-radius: 4px;
  overflow: hidden;
}

#webcam-preview {
  width: 100%;
  height: 100%;
  transform: scaleX(-1);
  opacity: 0.7;
  object-fit: cover;
}

#skeleton-canvas {
  position: absolute;
  inset: 0;
  /* MediaPipe skeleton drawn here in cyan/magenta */
}
```

---

## 8. Weapon Fire Visual Effects

All effects rendered on a dedicated effects canvas layer between Three.js canvas and HUD canvas.

### Repulsor Blast
- Blue circular energy ring expands from palm position
- Projectile: bright blue sphere with motion trail
- Impact: blue flash + particle burst at hit point

### Charged Repulsor
- Palm glows progressively from dim to blinding white-blue during charge
- Projectile: larger, brighter, leaves longer trail
- Impact: shockwave ring + larger particle explosion

### Missile
- Small rocket model spawns at hand, accelerates toward locked target
- Red exhaust trail
- Explosion: orange-red with debris particles

### Unibeam
- Screen center charges — radial blue-white gradient builds from chest
- Camera zooms in slightly during charge
- Fires: full screen width beam, camera pulls wide dramatically
- Everything in beam path explodes simultaneously
- Post-fire: screen briefly white then fades back

### Scatter Shot
- 8 small blue projectiles fan out in cone from palm
- Rapid fire sound burst
- Each projectile has small impact effect

---

## 9. Avengers Easter Egg Visual Sequence

### Progress During Hold (0-2 seconds)
- Avengers logo begins faintly appearing in sky — opacity 0 to 0.3
- Both fist outlines pulse with gold energy

### Trigger (2 seconds)
- Screen flash white
- The Avengers.mp3 hits immediately
- Sky tears open with energy rift — vertical split with electric edges

### Sequence (10 seconds)
| Time | Visual |
|---|---|
| 0.0s | Avengers logo fully visible in sky, gold glow |
| 1.0s | Shield ricochets across battlefield — shield.glb arc path |
| 2.0s | Lightning strikes from top — white bolt chains to heavy enemies |
| 3.0s | Shockwave ring from ground — Hulk impact effect |
| 4.0s | All enemies explode simultaneously |
| 5.0s | Score bonus numbers float up from each destroyed enemy |
| 6.0s | Logo begins fading |
| 8.0s | Rift closes |
| 10.0s | Combat resumes, Battle Finale.mp3 returns |

---

## 10. Level Environment Skyboxes

| Level | Skybox | Atmosphere |
|---|---|---|
| 1 — Low Orbit | Deep blue atmosphere, curvature of Earth visible below | Bright, clear |
| 2 — City | Urban skyline, day time, smoke and fire | Chaotic, orange tones |
| 3 — Ocean | Dark ocean, offshore platform, stormy sky | Moody, grey-green |
| 4 — Deep Space | Full black, stars, nebula | Dark, cold, purple tones |
| 5 — Stark Tower | Manhattan night, tower lights, dramatic | Personal, high stakes |

Use Three.js CubeTextureLoader or PMREMGenerator with Polyhaven HDRI for each environment.

---

## 11. Final Checklist for Builder

- [ ] Start screen fades in properly, click initializes audio
- [ ] Lab environment loads with correct blue lighting
- [ ] Holographic panels float and respond to hand interaction
- [ ] Suit up gesture detects reliably — arms crossed then spread
- [ ] Suit up sequence runs exactly 8 seconds with correct timing
- [ ] Arc Reactor pulse at second 3.5 is visually dramatic
- [ ] HUD boots up cleanly after helmet locks
- [ ] All 5 weapon gestures fire correctly
- [ ] Dodge detects from nose landmark — left, right, duck
- [ ] Enemy waves spawn correctly per level config
- [ ] Boss spawns after final wave cleared
- [ ] Health bar updates and flashes critical below 30%
- [ ] Unibeam cooldown arc visible and accurate
- [ ] Avengers Easter egg triggers after 2 second hold — once per level only
- [ ] Avengers sequence runs exactly 10 seconds
- [ ] Camera shake on damage hit
- [ ] Camera pulls wide on unibeam fire
- [ ] Webcam preview shows mirrored with skeleton overlay
- [ ] Upgrade system in lab correctly modifies combat stats
- [ ] Battle Finale.mp3 loops during all combat
- [ ] App runs on Chrome/Edge via local server only