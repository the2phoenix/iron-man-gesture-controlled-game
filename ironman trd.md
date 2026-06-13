# TRD — Iron Man: Stark Protocol
**Version:** 1.0  
**Author:** Arnesh  
**Stack:** Vanilla HTML/CSS/JS + Three.js, Single File, No Build Step  

---

## 1. Tech Stack

| Layer | Technology | Reason |
|---|---|---|
| 3D Rendering | Three.js r128 (CDN) | GLB model loading, scene, camera, lighting |
| Hand Tracking | MediaPipe Hands (CDN) | 21 landmark real-time detection |
| Head/Dodge Tracking | MediaPipe Face Mesh (CDN) | Nose landmark for left/right/duck detection |
| Audio | Howler.js (CDN) | Reliable cross-browser audio with spatial support |
| Model Loading | Three.js GLTFLoader | Loads all GLB/GLTF assets |
| Physics | Custom JS | Simple collision detection, no heavy physics engine needed |
| Deployment | Single HTML file | Drop in folder, open via local server |

---

## 2. File Structure

```
/iron-man-stark-protocol/
│
├── index.html
│
├── /models/
│   ├── iron_man_mark_85.glb
│   ├── sci-fi_lab.glb
│   ├── buster_drone.glb
│   ├── sci-fi_drone.glb
│   ├── fighter_jet.glb
│   ├── turret.glb
│   ├── mothership.glb
│   └── shield.glb
│
├── /images/
│   └── avengers_logo.png
│
└── /audio/
    ├── repulsor.mp3
    ├── explosion.mp3
    ├── unibeam_charge.mp3
    ├── suit_up.mp3
    ├── hud_startup.mp3
    ├── The Avengers.mp3
    └── Battle Finale.mp3
```

---

## 3. Three.js Scene Setup

```js
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.toneMapping = THREE.ACESFilmicToneMapping;
renderer.toneMappingExposure = 1.2;
document.body.appendChild(renderer.domElement);

// Arc Reactor blue ambient
const ambientLight = new THREE.AmbientLight(0x0a1628, 0.5);
scene.add(ambientLight);

// Primary blue point light — Arc Reactor source
const arcLight = new THREE.PointLight(0x00cfff, 2, 50);
arcLight.position.set(0, 2, 0);
scene.add(arcLight);

// Rim lighting for depth
const rimLight = new THREE.DirectionalLight(0x0066ff, 0.8);
rimLight.position.set(-5, 5, -5);
scene.add(rimLight);
```

---

## 4. Model Loading

```js
const loader = new THREE.GLTFLoader();

function loadModel(path, onLoad) {
  loader.load(
    path,
    (gltf) => {
      const model = gltf.scene;
      scene.add(model);
      onLoad(model);
    },
    undefined,
    (error) => console.error(`Failed to load ${path}:`, error)
  );
}

// Load all models at startup
const models = {};

loadModel('models/sci-fi_lab.glb', (m) => { models.lab = m; });
loadModel('models/iron_man_mark_85.glb', (m) => { models.suit = m; m.visible = false; });
loadModel('models/buster_drone.glb', (m) => { models.drone = m; m.visible = false; });
loadModel('models/sci-fi_drone.glb', (m) => { models.heavyDrone = m; m.visible = false; });
loadModel('models/fighter_jet.glb', (m) => { models.jet = m; m.visible = false; });
loadModel('models/turret.glb', (m) => { models.turret = m; m.visible = false; });
loadModel('models/mothership.glb', (m) => { models.mothership = m; m.visible = false; });
loadModel('models/shield.glb', (m) => { models.shield = m; m.visible = false; });
```

---

## 5. MediaPipe Hands Setup

```js
const hands = new Hands({
  locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
});

hands.setOptions({
  maxNumHands: 2,
  modelComplexity: 1,
  minDetectionConfidence: 0.7,
  minTrackingConfidence: 0.6
});

hands.onResults(onHandResults);
```

Note: maxNumHands is 2 here because the unibeam requires both palms together and the Avengers Easter egg requires both fists raised.

---

## 6. MediaPipe Face Mesh — Nose Tracking

```js
const faceMesh = new FaceMesh({
  locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`
});

faceMesh.setOptions({
  maxNumFaces: 1,
  refineLandmarks: false,
  minDetectionConfidence: 0.5,
  minTrackingConfidence: 0.5
});

faceMesh.onResults(onFaceResults);

// Nose tip is landmark index 1
function onFaceResults(results) {
  if (!results.multiFaceLandmarks?.length) return;
  const nose = results.multiFaceLandmarks[0][1];
  detectDodge(nose);
}

let baseNoseX = null;
let baseNoseY = null;

function detectDodge(nose) {
  if (!baseNoseX) { baseNoseX = nose.x; baseNoseY = nose.y; return; }

  const deltaX = nose.x - baseNoseX;
  const deltaY = nose.y - baseNoseY;

  if (deltaX > 0.08) triggerDodge('right');
  else if (deltaX < -0.08) triggerDodge('left');
  else if (deltaY > 0.06) triggerDodge('duck');
  else baseNoseX = nose.x * 0.1 + baseNoseX * 0.9; // slow recenter
}
```

---

## 7. Gesture Detection

### 7.1 3D Finger State Using Vector Math

```js
function isFingerExtended(landmarks, tipIdx, mcpIdx, pipIdx) {
  const tip = new THREE.Vector3(landmarks[tipIdx].x, landmarks[tipIdx].y, landmarks[tipIdx].z);
  const mcp = new THREE.Vector3(landmarks[mcpIdx].x, landmarks[mcpIdx].y, landmarks[mcpIdx].z);
  const pip = new THREE.Vector3(landmarks[pipIdx].x, landmarks[pipIdx].y, landmarks[pipIdx].z);

  const v1 = new THREE.Vector3().subVectors(pip, mcp).normalize();
  const v2 = new THREE.Vector3().subVectors(tip, pip).normalize();
  const angle = Math.acos(Math.max(-1, Math.min(1, v1.dot(v2))));

  return angle < Math.PI / 4; // less than 45 degrees = extended
}
```

### 7.2 Gesture Recognition

```js
function detectGestures(landmarks) {
  const thumbExt = isFingerExtended(landmarks, 4, 2, 3);
  const indexExt = isFingerExtended(landmarks, 8, 5, 6);
  const middleExt = isFingerExtended(landmarks, 12, 9, 10);
  const ringExt = isFingerExtended(landmarks, 16, 13, 14);
  const pinkyExt = isFingerExtended(landmarks, 20, 17, 18);

  // Palm thrust — all fingers extended, wrist moving forward (z delta)
  if (indexExt && middleExt && ringExt && pinkyExt) return 'PALM_THRUST';

  // Fist — all fingers curled
  if (!indexExt && !middleExt && !ringExt && !pinkyExt) return 'FIST';

  // Two finger point — index and middle extended only
  if (indexExt && middleExt && !ringExt && !pinkyExt) return 'TWO_FINGER_POINT';

  return null;
}
```

### 7.3 Charged Repulsor — Hold Detection

```js
let palmOpenStartTime = null;
const CHARGE_DURATION = 1500; // 1.5 seconds

function checkChargedRepulsor(gesture) {
  if (gesture === 'PALM_THRUST') {
    if (!palmOpenStartTime) palmOpenStartTime = Date.now();
    const held = Date.now() - palmOpenStartTime;
    if (held >= CHARGE_DURATION) return 'CHARGED_REPULSOR';
    return 'REPULSOR'; // not charged yet
  } else {
    palmOpenStartTime = null;
    return null;
  }
}
```

### 7.4 Unibeam — Both Palms Together

```js
function detectUnibeam(leftLandmarks, rightLandmarks) {
  if (!leftLandmarks || !rightLandmarks) return false;

  const leftPalm = leftLandmarks[9];
  const rightPalm = rightLandmarks[9];

  const dist = Math.sqrt(
    Math.pow(leftPalm.x - rightPalm.x, 2) +
    Math.pow(leftPalm.y - rightPalm.y, 2)
  );

  return dist < 0.15; // palms within 15% of screen width
}
```

### 7.5 Suit Up Gesture — Arms Crossed Then Spread

```js
let armsCrossedTime = null;

function detectSuitUp(leftLandmarks, rightLandmarks) {
  if (!leftLandmarks || !rightLandmarks) return false;

  const leftWrist = leftLandmarks[0];
  const rightWrist = rightLandmarks[0];

  // Arms crossed: right wrist is left of left wrist
  const crossed = rightWrist.x < leftWrist.x;

  if (crossed && !armsCrossedTime) {
    armsCrossedTime = Date.now();
  }

  // Arms spread: wrists far apart after being crossed
  const spread = Math.abs(leftWrist.x - rightWrist.x) > 0.6;

  if (spread && armsCrossedTime && Date.now() - armsCrossedTime < 2000) {
    armsCrossedTime = null;
    return true; // suit up triggered
  }

  if (!crossed) armsCrossedTime = null;
  return false;
}
```

### 7.6 Avengers Easter Egg — Both Fists Raised

```js
let bothFistsRaisedTime = null;
const AVENGERS_HOLD = 2000;

function detectAvengersPose(leftLandmarks, rightLandmarks) {
  if (!leftLandmarks || !rightLandmarks) {
    bothFistsRaisedTime = null;
    return false;
  }

  const leftFist = detectGestures(leftLandmarks) === 'FIST';
  const rightFist = detectGestures(rightLandmarks) === 'FIST';
  const leftHigh = leftLandmarks[0].y < 0.4;
  const rightHigh = rightLandmarks[0].y < 0.4;

  if (leftFist && rightFist && leftHigh && rightHigh) {
    if (!bothFistsRaisedTime) bothFistsRaisedTime = Date.now();
    if (Date.now() - bothFistsRaisedTime >= AVENGERS_HOLD) {
      bothFistsRaisedTime = null;
      return true;
    }
  } else {
    bothFistsRaisedTime = null;
  }
  return false;
}
```

---

## 8. Combat System

### 8.1 Enemy Data Structure

```js
class Enemy {
  constructor(type, model, health, speed, damage, points) {
    this.type = type;
    this.model = model.clone();
    this.health = health;
    this.maxHealth = health;
    this.speed = speed;
    this.damage = damage;
    this.points = points;
    this.position = new THREE.Vector3();
    this.alive = true;
  }
}

const ENEMY_CONFIGS = {
  drone:      { health: 30,  speed: 0.05, damage: 10, points: 100 },
  heavyDrone: { health: 100, speed: 0.02, damage: 25, points: 300 },
  jet:        { health: 60,  speed: 0.08, damage: 20, points: 200 },
  turret:     { health: 150, speed: 0,    damage: 30, points: 400 },
  mothership: { health: 500, speed: 0.01, damage: 40, points: 2000 }
};
```

### 8.2 Weapon Data Structure

```js
const WEAPONS = {
  repulsor:        { damage: 25,  cooldown: 200,   ammo: Infinity },
  chargedRepulsor: { damage: 100, cooldown: 1500,  ammo: Infinity },
  missile:         { damage: 150, cooldown: 800,   ammo: 5 },
  unibeam:         { damage: 400, cooldown: 8000,  ammo: Infinity },
  scatter:         { damage: 60,  cooldown: 1000,  ammo: 8 }
};
```

### 8.3 Level Wave Configuration

```js
const LEVELS = [
  { // Level 1 — Low Orbit
    waves: [
      { drone: 5 },
      { drone: 8 },
      { drone: 12 },
      { boss: 'mothership' }
    ],
    skybox: 'orbit'
  },
  { // Level 2 — City
    waves: [
      { drone: 6, heavyDrone: 2 },
      { drone: 8, heavyDrone: 4 },
      { drone: 10, heavyDrone: 6 },
      { boss: 'tank' }
    ],
    skybox: 'city'
  },
  { // Level 3 — Ocean
    waves: [
      { jet: 3, turret: 2 },
      { jet: 5, turret: 3 },
      { jet: 6, turret: 4, drone: 4 },
      { boss: 'carrier' }
    ],
    skybox: 'ocean'
  },
  { // Level 4 — Deep Space
    waves: [
      { drone: 8, heavyDrone: 4, jet: 2 },
      { drone: 10, heavyDrone: 6, jet: 4 },
      { drone: 12, heavyDrone: 8, jet: 6, turret: 2 },
      { boss: 'warship' }
    ],
    skybox: 'space'
  },
  { // Level 5 — Stark Tower
    waves: [
      { drone: 10, heavyDrone: 5, jet: 4, turret: 3 },
      { drone: 15, heavyDrone: 8, jet: 6, turret: 4 },
      { drone: 20, heavyDrone: 10, jet: 8, turret: 6 },
      { boss: 'ironmonger' }
    ],
    skybox: 'stark_tower'
  }
];
```

---

## 9. Camera System

```js
// Combat camera — dynamic cinematic movement
const cameraStates = {
  normal:   { fov: 75, shake: 0 },
  incoming: { fov: 65, shake: 0 },    // enemy approaching — wider view
  charging: { fov: 70, shake: 0 },    // unibeam charging — slight zoom in
  firing:   { fov: 85, shake: 0 },    // unibeam fires — pulls wide
  hit:      { fov: 75, shake: 0.3 },  // taking damage — camera shake
  boss:     { fov: 60, shake: 0 }     // boss reveal — dramatic pan
};

function updateCamera(state) {
  const target = cameraStates[state];
  camera.fov += (target.fov - camera.fov) * 0.05; // smooth lerp
  camera.updateProjectionMatrix();

  if (target.shake > 0) {
    camera.position.x += (Math.random() - 0.5) * target.shake;
    camera.position.y += (Math.random() - 0.5) * target.shake;
  }
}
```

---

## 10. Upgrade System Data Structure

```js
const upgradeTree = {
  repulsorDamage:    { level: 0, max: 5, cost: [100,200,400,800,1600], multiplier: 1.25 },
  repulsorFireRate:  { level: 0, max: 5, cost: [100,200,400,800,1600], multiplier: 0.85 },
  missileAmmo:       { level: 0, max: 5, cost: [150,300,600,1200,2400], bonus: 2 },
  unibeamCooldown:   { level: 0, max: 5, cost: [200,400,800,1600,3200], multiplier: 0.80 },
  maxHealth:         { level: 0, max: 5, cost: [100,200,400,800,1600], bonus: 20 },
  damageReduction:   { level: 0, max: 5, cost: [150,300,600,1200,2400], multiplier: 0.90 },
  enemyRadar:        { level: 0, max: 1, cost: [500], unlocks: 'radar' },
  weakPointDetect:   { level: 0, max: 1, cost: [800], unlocks: 'weakpoints' }
};
```

---

## 11. Performance Targets

| Metric | Target |
|---|---|
| Three.js render framerate | 60fps |
| Hand tracking framerate | 30fps |
| Gesture to weapon fire | under 100ms |
| Enemy count on screen | up to 20 simultaneously |
| Model load time | under 5 seconds on local server |
| Suit up sequence duration | exactly 8 seconds |
| Avengers Easter egg duration | exactly 10 seconds |

---

## 12. Browser Requirements

- Chrome or Edge — MediaPipe and Three.js work best here
- Webcam permission required for both hands and face tracking
- Run via local server — python -m http.server 8080 or VS Code Live Server
- Do not open via file:// — audio and model loading will fail due to CORS