# GSD — Iron Man: Stark Protocol
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Every gesture, state, transition, and edge case  

---

## 1. System States

```
[START SCREEN]
      ↓  click anywhere
[LAB MODE]
      ↓  arms crossed then spread
[SUIT UP]
      ↓  sequence completes (8 seconds)
[COMBAT MODE]
      ↓  level complete
[LAB MODE]  ← upgrade, select next level, repeat
      ↓  any time during combat
[AVENGERS EASTER EGG]  → returns to combat after 10s
```

---

## 2. Mode Transitions

### 2.1 Start Screen → Lab

- User clicks anywhere on start screen
- Tone.js / Howler.js audio context initializes
- Lab environment loads
- Camera settles into lab position
- Holographic panels fade in

### 2.2 Lab → Suit Up

- Trigger: both arms crossed over chest then spread wide within 2 seconds
- Lab holograms fade out
- suit_up.mp3 begins immediately
- Iron Man suit assembly sequence begins
- Cannot be cancelled once started

### 2.3 Suit Up → Combat

- Triggered automatically after 8 second assembly sequence completes
- hud_startup.mp3 plays
- HUD elements fade in
- Screen flashes white briefly
- Combat environment loads based on selected level
- Battle Finale.mp3 begins
- First wave spawns after 3 second countdown

### 2.4 Combat → Lab (level complete)

- All enemies in final wave defeated including boss
- Victory camera flyby plays (3 seconds)
- Score tallied and displayed
- Upgrade points awarded
- Fade to black
- Lab loads with updated upgrade panels
- Suit remains on — hand overlays stay as gauntlets

### 2.5 Combat → Lab (player death)

- Health reaches 0
- Suit damage explosion effect
- HUD flickers and goes dark
- Death screen shows score and level reached
- Retry or return to lab option shown as holographic panels
- Selecting retry reloads same level at full health

---

## 3. Gesture Specifications

### 3.1 Repulsor Blast

| Property | Value |
|---|---|
| Gesture | Single open palm facing forward, thrust toward screen |
| Detection | All 4 fingers extended, wrist z-velocity forward |
| Response time | Under 100ms |
| Cooldown | 200ms |
| Visual | Blue energy projectile fires from palm position |
| Audio | repulsor.mp3 |
| Damage | 25 base, modified by upgrades |

### 3.2 Charged Repulsor

| Property | Value |
|---|---|
| Gesture | Hold open palm for 1.5 seconds then thrust |
| Detection | PALM_THRUST held for 1500ms then z-velocity forward |
| Charge visual | Palm glows progressively brighter during hold |
| Response | Fires on thrust after charge complete |
| Cooldown | 1500ms |
| Visual | Large expanding energy blast |
| Audio | unibeam_charge.mp3 at low volume during charge, repulsor.mp3 on fire |
| Damage | 100 base |

### 3.3 Missile Lock

| Property | Value |
|---|---|
| Gesture | Both index fingers pointing outward hands apart |
| Detection | Index extended, all others curled, on both hands |
| Lock mechanic | Targeting reticle appears, locks onto nearest enemy after 0.8s |
| Fire | Hold gesture until locked, drop gesture to fire |
| Ammo | 5 per level, refills on level complete |
| Visual | Red targeting brackets, missile trail |
| Audio | Lock tone during aim, explosion.mp3 on hit |
| Damage | 150 base |

### 3.4 Unibeam

| Property | Value |
|---|---|
| Gesture | Both palms facing forward pushed together within 0.15 normalized units |
| Detection | Both hands PALM_THRUST, palm distance under threshold |
| Charge visual | Screen center charges with white/blue energy, camera zooms in |
| Fire | Hold gesture — continuous beam while palms together |
| Cooldown | 8 seconds after release |
| Visual | Massive sustained energy beam, camera pulls wide |
| Audio | unibeam_charge.mp3 sustained |
| Damage | 400 per second continuous |

### 3.5 Scatter Shot

| Property | Value |
|---|---|
| Gesture | Fist then open fast (under 300ms) |
| Detection | FIST then immediate PALM_THRUST within 300ms |
| Ammo | 8 per level |
| Visual | 8 energy pellets spread in cone pattern |
| Audio | repulsor.mp3 × 3 rapid |
| Damage | 60 total area |

### 3.6 Lab Hologram Interaction

| Gesture | Action |
|---|---|
| Reach toward panel and close fingers | Select / grab panel |
| Push panel away with open palm | Dismiss panel |
| Pull panel toward you | Expand / confirm |
| Pinch two fingers | Zoom into detail |

### 3.7 Suit Up Gesture

| Property | Value |
|---|---|
| Gesture | Both arms crossed over chest then spread wide |
| Detection | Right wrist crosses left of left wrist, then both wrists spread beyond 0.6 normalized distance within 2 seconds |
| Only available | In Lab mode |
| Cannot be triggered | During combat or suit up sequence |

### 3.8 Avengers Easter Egg

| Property | Value |
|---|---|
| Gesture | Both fists raised above head simultaneously |
| Detection | Both hands FIST, both wrists y-position above 0.4 normalized |
| Hold duration | 2 seconds continuous |
| Progress indicator | Avengers logo faintly appears in sky growing more visible during hold |
| Only available | During combat |
| Effect | Clears all enemies, plays 10 second sequence |
| Cooldown | Once per level only |

---

## 4. Dodge System

| Head Movement | Action | Detection |
|---|---|---|
| Lean left | Dodge left | Nose x-delta > 0.08 |
| Lean right | Dodge right | Nose x-delta < -0.08 |
| Duck down | Roll under | Nose y-delta > 0.06 |

Dodge window: 500ms invincibility frames after dodge triggered.
Base nose position recalibrates slowly during combat to account for gradual position drift.

---

## 5. Gesture Priority Order

When multiple gesture conditions could be true simultaneously:

| Priority | Gesture | Notes |
|---|---|---|
| 1 | Avengers Easter Egg | Overrides everything during combat |
| 2 | Suit Up | Only in lab, overrides lab interactions |
| 3 | Unibeam | Both hands required, overrides single hand weapons |
| 4 | Missile Lock | Both hands required |
| 5 | Charged Repulsor | Single hand, held |
| 6 | Scatter Shot | Single hand, quick |
| 7 | Repulsor | Default firing gesture |
| 8 | Lab Interaction | Only active in lab mode |

---

## 6. Enemy Behavior Specifications

### Basic Drone (buster_drone.glb)
- Spawns at distance, flies toward player in swarm formation
- Fires single low damage shots every 2 seconds
- Erratic movement path — harder to track
- Dies in one repulsor hit

### Heavy Drone (sci-fi_drone.glb)
- Spawns at medium distance, approaches slowly
- Fires burst shots every 1.5 seconds, higher damage
- Tanks repulsor shots, needs charged repulsor or missile
- Has visible health bar above model

### Fighter Jet (fighter_jet.glb)
- Fast linear attack runs from left or right
- Fires while passing, then loops back
- Missile lock is most effective
- Cannot be hit easily during approach — wait for pass

### Ground Turret (turret.glb)
- Stationary, spawns on ground plane
- Rapid fire, high accuracy
- Unibeam most effective
- Has visible damage state — sparks and smoke at 50% health

### Mothership Boss (mothership.glb)
- Spawns at max distance, slowly approaches
- Releases drone waves periodically
- Has 3 weak points — targeting reticle highlights them
- Must hit all 3 weak points to deal full damage
- Phase 2 at 50% health — releases heavy drones and charges player

---

## 7. Edge Cases and Failure Handling

| Scenario | Behavior |
|---|---|
| Camera permission denied | Full screen prompt explaining both webcam and face tracking required |
| Model fails to load | Console error, placeholder geometry used, game continues |
| Audio file not found | Silent fallback, console warning |
| Face mesh not detecting | Dodge system disabled, HUD shows warning icon |
| Both hands not detected | Unibeam and missile lock disabled until both hands visible |
| Player stands too still in lab | Subtle pulse animation on nearest interactive panel as hint |
| Avengers Easter egg already used this level | Gesture detected but nothing triggers — Easter egg is once per level |
| Boss killed before all waves clear | Wave clears instantly, victory sequence triggers |
| Upgrade points insufficient | Panel shakes, cannot select |