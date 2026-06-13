# SKILL.md — Iron Man: Stark Protocol
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Master build instructions for Claude Opus  

---

## Who You Are

You are Claude Opus 4.6 building a browser-based Iron Man experience called **Iron Man: Stark Protocol**. You have been given 4 documents to read before writing a single line of code:

1. `ironman-PRD.md` — What to build and why
2. `ironman-TRD.md` — Exact technical implementation with code samples
3. `ironman-GSD.md` — Every gesture, state, transition, and edge case
4. `ironman-UIUX.md` — Complete visual, motion, and layout specification

Read all four completely before writing any code.

---

## What You Are Building

A single HTML file (`index.html`) that:

- Renders a full 3D Iron Man experience using Three.js
- Tracks hands via MediaPipe Hands for gesture-based weapon firing
- Tracks nose landmark via MediaPipe Face Mesh for dodge mechanics
- Has three deliberate modes: Lab, Suit Up, Combat
- Is a fully playable shooting game with 5 levels, bosses, scoring, and upgrades
- Has an Avengers Assemble Easter egg triggered by both fists raised

---

## Asset Reference

All assets are already downloaded and named. Use these exact paths:

### Models
```
models/iron_man_mark_85.glb     — Player suit, assembles during suit up
models/sci-fi_lab.glb           — Lab environment, Mode 1
models/buster_drone.glb         — Basic drone enemy
models/sci-fi_drone.glb         — Heavy drone enemy
models/fighter_jet.glb          — Fighter jet enemy
models/turret.glb               — Ground turret enemy
models/mothership.glb           — Level 1 boss
models/shield.glb               — Captain America shield for Easter egg
```

### Images
```
images/avengers_logo.png        — Avengers Easter egg sky overlay
```

### Audio
```
audio/repulsor.mp3              — Repulsor blast sound
audio/explosion.mp3             — Enemy destruction
audio/unibeam_charge.mp3        — Unibeam charge and fire
audio/suit_up.mp3               — Full suit up sequence audio
audio/hud_startup.mp3           — HUD activation after helmet locks
audio/The Avengers.mp3          — Avengers Easter egg theme
audio/Battle Finale.mp3         — Combat background music, loops
```

---

## Build Order

Follow this exact sequence. Do not skip steps.

### Step 1 — HTML Structure
```
- Canvas for Three.js (full viewport, z-index 0)
- Canvas for effects layer (full viewport, z-index 5, transparent)
- Canvas for HUD (full viewport, z-index 10, transparent)
- #webcam-container (bottom right, fixed)
  - #webcam-preview (video element, mirrored)
  - #skeleton-canvas (overlaid on webcam)
- #start-screen (full viewport overlay)
- #damage-overlay (full viewport, hidden by default)
- Hidden video element for MediaPipe input
```

### Step 2 — Three.js Scene
- Initialize renderer with ACESFilmic tone mapping
- Set up Arc Reactor blue ambient and point lighting
- Set up camera at lab position
- Initialize GLTFLoader
- Load all 8 models at startup, set non-lab models to invisible

### Step 3 — MediaPipe Hands
- Initialize with maxNumHands: 2
- Wire onResults callback
- Render skeleton on skeleton-canvas overlay

### Step 4 — MediaPipe Face Mesh
- Initialize with refineLandmarks: false for performance
- Extract nose landmark index 1
- Implement dodge detection with slow recenter logic

### Step 5 — Gesture System
- Implement 3D vector finger extension detection
- Implement all 5 weapon gestures
- Implement suit up gesture
- Implement lab interaction gestures
- Implement Avengers Easter egg gesture with 2 second hold
- Implement gesture priority order from GSD.md

### Step 6 — Lab Mode
- Render sci-fi_lab.glb environment
- Create 3 floating holographic panels (left, center, right)
- Implement panel bob animation
- Implement hand interaction with panels
- Wire level select and upgrade panels

### Step 7 — Suit Up Sequence
- Implement exact 8 second timed sequence from UIUX.md
- Play suit_up.mp3 synchronized to sequence
- Arc Reactor pulse at 3.5 seconds
- Transition to combat at 8 seconds

### Step 8 — Combat System
- Implement Enemy class with all 5 types
- Implement wave spawning from LEVELS config in TRD.md
- Implement all 5 weapons with correct damage, cooldown, ammo
- Implement health system with partial wave restore
- Implement scoring with multipliers
- Implement boss spawning and behavior

### Step 9 — HUD System
- Render all HUD elements on HUD canvas
- Health bar with critical state below 30%
- Unibeam cooldown arc around right hand
- Enemy radar top left
- Score and level display top
- Targeting reticle for missile lock
- Weapon and ammo readout bottom right

### Step 10 — Camera System
- Implement dynamic camera states from TRD.md
- Camera shake on damage
- Camera zoom in on unibeam charge
- Camera pull wide on unibeam fire
- Boss reveal cinematic pan

### Step 11 — Avengers Easter Egg
- Implement full 10 second sequence from UIUX.md
- Shield ricochet animation using shield.glb
- Lightning strike particle effect
- Hulk shockwave ring
- All enemy simultaneous explosion
- Score bonus floating numbers
- Avengers logo sky overlay
- Once per level enforcement

### Step 12 — Upgrade System
- Wire upgrade tree from TRD.md
- Apply multipliers to weapon and defense stats in combat
- Persist upgrade state across levels in session

### Step 13 — Start Screen and Audio Init
- Implement start screen from UIUX.md
- Initialize Howler.js audio context on click
- Preload all audio files

---

## Critical Rules

### Never do these:
- Do not use file:// — remind user to run local server
- Do not use localStorage or sessionStorage
- Do not use form tags anywhere
- Do not use alert() for errors — use console.error and graceful UI fallbacks
- Do not break the gesture priority order from GSD.md
- Do not allow Avengers Easter egg to trigger more than once per level

### Three.js Specific:
- Use r128 from CDN: https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js
- Do NOT use THREE.CapsuleGeometry — use CylinderGeometry or SphereGeometry instead
- GLTFLoader must be imported separately from Three.js CDN addons
- Always dispose of geometries and materials when removing objects from scene

### Performance:
- Enemy count cap at 20 simultaneously on screen
- Dispose enemy models when killed, not just set invisible
- Use object pooling for projectiles — reuse rather than create/destroy
- Face Mesh and Hands both running simultaneously — if frame rate drops below 30fps, reduce Face Mesh update frequency to every other frame

### Audio:
- All audio must initialize inside a user gesture handler (the start screen click)
- Battle Finale.mp3 loops continuously during all combat
- suit_up.mp3 plays once during suit up, does not loop
- Crossfade Battle Finale back in after Avengers Easter egg ends

---

## Error States

| Error | Behavior |
|---|---|
| Camera permission denied | Full screen message: "camera access required for hand tracking — refresh and allow" |
| Model fails to load | Console error, use placeholder BoxGeometry, continue |
| Audio file not found | Console warning, silent fallback |
| Face mesh not detecting | Disable dodge, show small warning icon in HUD corner |
| Both hands not visible | Disable unibeam and missile lock, HUD indicates |

---

## Local Server Reminder

Add this comment at the top of index.html:

```html
<!-- 
  IRON MAN: STARK PROTOCOL
  Run with a local server — do NOT open via file://
  Options:
  - python -m http.server 8080
  - VS Code Live Server extension
  - npx serve .
  Then open: http://localhost:8080
-->
```

---

## Definition of Done

- [ ] Start screen visible, click initializes everything
- [ ] Lab loads with floating holographic panels
- [ ] Hand interaction works with panels
- [ ] Suit up gesture triggers assembly sequence
- [ ] Assembly sequence runs 8 seconds with correct timing
- [ ] Arc Reactor pulse is dramatic at 3.5 seconds
- [ ] HUD boots after helmet locks
- [ ] All 5 weapons fire from correct gestures
- [ ] Dodge works from head lean in all 3 directions
- [ ] 5 levels each have correct enemy waves
- [ ] Boss spawns after final wave per level
- [ ] Avengers Easter egg triggers after 2 second hold
- [ ] Easter egg sequence runs 10 seconds and clears enemies
- [ ] Upgrade system works and persists across levels in session
- [ ] Dynamic camera reacts to combat situations
- [ ] Webcam shows mirrored with skeleton overlay
- [ ] Everything runs on Chrome/Edge via local server

---

## Notes from Arnesh

- Personal project, not commercial. All assets sourced from free/YouTube downloads.
- Boss models for levels 2-5 do not have downloaded GLB files — build them using Three.js geometry. Simple but recognizable shapes are fine. A tank can be boxes. A warship can be a large elongated form. The mothership.glb is the only real boss model.
- If any downloaded model looks wrong in the scene — wrong scale, wrong orientation, wrong materials — fix it programmatically in the loader callback. Do not tell the user to fix the model manually.
- The suit up sequence is the most important cinematic moment. Spend extra attention making it feel right.
- The Avengers Easter egg is the most shareable moment. Make it feel genuinely epic.
- If in doubt on visuals, refer to ironman-UIUX.md.
- If in doubt on gestures or states, refer to ironman-GSD.md.