# PRD — Iron Man: Stark Protocol
**Version:** 1.0  
**Author:** Arnesh  
**Status:** Ready for Build  

---

## 1. Overview

Iron Man: Stark Protocol is a browser-based hand gesture driven Iron Man experience with three deliberate modes — Lab, Suit Up, and Combat. The user controls everything through hand gestures tracked via webcam using MediaPipe Hands and nose landmark tracking for dodge mechanics. It is a fully playable shooting game with scoring, health, 5 levels, boss enemies per level, an upgrade system, and an Avengers Easter egg. There are no buttons, no menus, no mouse or keyboard interaction. The hands and head ARE the controller.

---

## 2. Goals

- Build a fully gesture driven Iron Man experience with zero traditional UI interaction
- Create three distinct modes that flow deliberately from one to the next
- Build a complete shooting game with progression, upgrades, and replayability
- Make the experience feel cinematic not just a tech demo
- Include an Avengers Assemble Easter egg as the unforgettable wow moment
- Run entirely in the browser as a single HTML file

---

## 3. User

Single user. Personal project. No authentication, no accounts, no multiplayer.

---

## 4. Three Modes

### 4.1 Mode 1 — The Lab

The starting environment. Dark sleek Stark Industries workshop. Arc Reactor blue lighting. sci-fi_lab.glb rendered as the environment.

What the user can do:
- Grab and manipulate floating holographic panels showing suit stats, level select, upgrade tree
- Push panels away to dismiss
- Pull panels closer to select
- Access upgrade system between levels
- Select next level to deploy

Feel: Calm, intelligent, Tony Stark casually reorganizing his workspace before a mission.

Transition out: Both arms crossed over chest then spread wide triggers Mode 2.

---

### 4.2 Mode 2 — Suit Up

Triggered by: both arms crossed over chest then spread wide.

The 8 second cinematic sequence:
1. Boots fly up and lock onto feet position
2. Leg armor assembles upward piece by piece
3. Chest piece slams in — Arc Reactor pulses to life
4. Gauntlets slide onto hands — hand overlays change to Iron Man gauntlet visuals
5. Helmet drops last — screen goes dark for 1 second
6. HUD activates — full Iron Man heads up display lights up
7. Combat mode begins

Audio: suit_up.mp3 plays through entire sequence. hud_startup.mp3 plays on HUD activation.

Visual: iron_man_mark_85.glb assembles piece by piece around the camera position.

---

### 4.3 Mode 3 — Combat

Wave based shooting game. 5 levels. Boss per level. Dynamic cinematic camera.

Player weapons:

| Gesture | Weapon | Damage | Notes |
|---|---|---|---|
| Single palm thrust | Repulsor blast | Low, fast | Infinite ammo |
| Hold palm open 1.5s then thrust | Charged repulsor | High, slow | Infinite ammo |
| Two index fingers pointing hands apart | Missile lock | High | Limited ammo refills per level |
| Both palms together pushed forward | Unibeam | Massive | Long cooldown |
| Fist then open fast | Scatter shot | Medium area | Limited ammo |

Dodge mechanics:
- Nose landmark tracked via MediaPipe Face Mesh
- Lean head left → character dodges left
- Lean head right → character dodges right
- Duck head down → character rolls under attack

Health system:
- 100 health per level
- Partial restore between waves not between levels
- Full restore on level complete

Scoring:
- Points per enemy killed
- Multiplier for consecutive kills without taking damage
- Speed bonus for finishing waves quickly
- Level completion bonus

---

## 5. Enemy Types

| Enemy | Model | Behavior | Kill Method |
|---|---|---|---|
| Basic drone | buster_drone.glb | Fast, swarms in groups | Single repulsor |
| Heavy drone | sci-fi_drone.glb | Slow, tanky, high damage | Charged repulsor or missile |
| Fighter jet | fighter_jet.glb | Fast linear attack runs | Missile lock |
| Ground turret | turret.glb | Stationary, rapid fire | Unibeam |
| Boss per level | See level table | Unique mechanics | Combination |

---

## 6. Levels

| Level | Setting | Enemy Mix | Boss | Model |
|---|---|---|---|---|
| 1 | Low orbit daytime | Basic drones only | Drone mothership | mothership.glb |
| 2 | City under attack | Drones + heavy drones | Armored tank | Generated in Three.js |
| 3 | Ocean platform | Jets + turrets | Submarine carrier | Generated in Three.js |
| 4 | Deep space | All enemy types | Massive warship | Generated in Three.js |
| 5 | Stark Tower | Everything + new variants | Final boss Iron Monger scale | Generated in Three.js |

---

## 7. Upgrade System

Upgrade points earned by completing levels and achieving high scores. Spent in the Lab between levels.

| Category | Upgrades Available |
|---|---|
| Repulsor | Damage, fire rate, charge speed, piercing shots |
| Missiles | Ammo capacity, lock speed, blast radius, homing accuracy |
| Unibeam | Damage, cooldown reduction, beam width, duration |
| Scatter Shot | Pellet count, spread pattern, reload speed |
| Suit Defense | Max health, damage reduction, dodge window, shield |
| HUD Systems | Enemy radar, weak point detection, ammo counter, threat alerts |

---

## 8. Avengers Easter Egg

Trigger: Both fists raised above head simultaneously held for 2 seconds.

Sequence:
1. Combat pauses instantly
2. Sky tears open with a dramatic energy rift
3. Avengers logo (avengers_logo.png) appears in the sky
4. Avengers theme (The Avengers.mp3) hits
5. Captain America shield (shield.glb) ricochets across battlefield clearing drones
6. Thor lightning strikes heavy enemies
7. Hulk shockwave clears ground turrets
8. All current enemies on screen are destroyed
9. Avengers logo fades, rift closes, combat resumes
10. Total duration: 10 seconds

This is the most shareable moment in the entire game. It rewards the player by clearing all enemies and serves as an emergency reset when overwhelmed.

---

## 9. Audio System

| File | Used For |
|---|---|
| repulsor.mp3 | Every repulsor blast |
| explosion.mp3 | Enemy destruction |
| unibeam_charge.mp3 | Unibeam charging and firing |
| suit_up.mp3 | Full suit up sequence |
| hud_startup.mp3 | HUD activation after helmet locks |
| The Avengers.mp3 | Avengers Easter egg sequence |
| Battle Finale.mp3 | Combat background music all levels |

---

## 10. Out of Scope

- Mobile support — desktop only, needs webcam
- Multiplayer
- Save states or cloud saves
- Online leaderboards
- Any backend, database, or authentication
- Commercial use — personal project only

---

## 11. Success Criteria

- All three modes transition smoothly and deliberately
- Suit up sequence feels cinematic and satisfying every time
- Combat feels responsive — gesture to weapon fire under 100ms
- Dodge mechanic works reliably from head lean detection
- Avengers Easter egg triggers cleanly after 2 second hold
- 5 levels each feel distinct in environment and enemy mix
- Upgrade system meaningfully changes combat feel
- Entire experience runs in browser via local server on Chrome or Edge