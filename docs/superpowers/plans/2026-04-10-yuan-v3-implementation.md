# 緣 (Yuán) v3 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite index.html to match v3 spec — fewer particles with individuality, continuous 2D field, distance-driven phase transitions, no decorative overlays.

**Architecture:** Single HTML file. Field (energy × order → derived params) drives 40 agents with mass/lifespan. Breath cycle, macro rhythm, traces, true emptiness, buddha-nature retained from v1. All Thangka overlays removed. Gold is a natural color region.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, Canvas 2D API, no dependencies.

**Approach:** Rewrite index.html from scratch rather than patching v1. The structural changes (continuous field, individual agent lifecycle, removal of aspect/thangka systems) touch every section. A clean rewrite is faster and less error-prone.

---

## File Structure

Single file: `index.html`

```
<script>
├── // === CANVAS ===
├── // === ENTROPY ===
├── // === FIELD ===          — energy, order → derived params, distance-driven lerp
├── // === MACRO ===          — abundance (rain/drought)
├── // === AGENTS ===         — 40 agents with mass, lifespan, individual alpha
├── // === BREATH ===         — life cycle state machine
├── // === KARMA ===          — target selection biased by current position
├── // === TRACES ===         — stamp + fade
├── // === EMPTINESS ===      — true void flag
├── // === BUDDHA-NATURE ===  — one pixel
├── // === COLOR ===          — field → hue/sat/light, gold as natural region
├── // === RENDER ===         — draw agents, connections, trails
└── // === LOOP ===           — orchestration
```

---

### Task 1: Canvas + Entropy + Field

**Files:**
- Create: `index.html`

- [ ] **Step 1: Write the HTML shell and canvas setup**

```html
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>緣</title>
<style>
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
html, body { width: 100%; height: 100%; overflow: hidden; background: #000; cursor: none; }
canvas { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; }
#trace { z-index: 0; }
#main  { z-index: 1; }
</style>
</head>
<body>
<canvas id="trace"></canvas>
<canvas id="main"></canvas>
<script>
// === CANVAS ===
const mainCanvas = document.getElementById('main');
const traceCanvas = document.getElementById('trace');
const ctx = mainCanvas.getContext('2d');
const traceCtx = traceCanvas.getContext('2d');
let W, H;
function resize() {
  W = mainCanvas.width = traceCanvas.width = innerWidth;
  H = mainCanvas.height = traceCanvas.height = innerHeight;
}
resize();
addEventListener('resize', resize);
```

- [ ] **Step 2: Add entropy function**

```javascript
// === ENTROPY ===
function entropy() {
  const t = performance.now();
  const s = Math.sin(t * 12345.6789 + t * t * 0.001) * 0.5 + 0.5;
  return (Math.random() + s) % 1;
}
```

- [ ] **Step 3: Add the 2D continuous field with distance-driven interpolation**

```javascript
// === FIELD ===
// Primary axes
const field = { energy: 0.3, order: 0.5 };
const fieldTarget = { energy: 0.3, order: 0.5 };

// Derived parameters — computed from energy + order each frame
const derived = { warmth: 0.5, rotation: 0, flowX: 0, flowY: 0, pulse: 0 };

function deriveParams() {
  const e = field.energy, o = field.order;
  // warmth: high energy + low order → hot; low energy + high order → cold
  derived.warmth = e * 0.6 + (1 - o) * 0.4;
  // rotation: strongest at high energy + mid order; weak at extremes
  derived.rotation = (entropy() > 0.5 ? 1 : -1) * e * (1 - Math.abs(o - 0.5) * 2) * 0.8;
  // flow: high energy + low order → strong directional; otherwise weak
  const flowStrength = e * (1 - o) * 1.5;
  derived.flowX = (entropy() - 0.5) * 2 * flowStrength;
  derived.flowY = (entropy() - 0.5) * 2 * flowStrength;
  // pulse: moderate energy regions
  derived.pulse = (1 - Math.abs(e - 0.5) * 2) * 0.6;
}

// Distance-driven interpolation: speed depends on distance to target
function lerpField(dt) {
  const de = fieldTarget.energy - field.energy;
  const dor = fieldTarget.order - field.order;
  const dist = Math.sqrt(de * de + dor * dor);
  // rate = base + distance² × acceleration
  const rate = (0.0003 + dist * dist * 0.005) * dt;
  field.energy += de * rate;
  field.order += dor * rate;
}

function chooseNextTarget() {
  // Biased by current position: 70% nearby drift, 30% larger jump
  const isJump = entropy() < 0.3;
  const range = isJump ? 0.8 : 0.3;
  let ne = field.energy + (entropy() - 0.5) * 2 * range;
  let no = field.order + (entropy() - 0.5) * 2 * range;
  // Clamp to 0-1
  ne = Math.max(0, Math.min(1, ne));
  no = Math.max(0, Math.min(1, no));
  // Never repeat exactly
  if (Math.abs(ne - fieldTarget.energy) < 0.05 && Math.abs(no - fieldTarget.order) < 0.05) {
    ne = entropy();
    no = entropy();
  }
  fieldTarget.energy = ne;
  fieldTarget.order = no;
  // Re-derive flow/rotation targets for the new region
  deriveParams();
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v3: canvas + entropy + continuous 2D field with distance-driven lerp"
```

---

### Task 2: Agents with Individuality

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add agent system with mass, lifespan, individual alpha**

```javascript
// === AGENTS ===
const NUM_AGENTS = 40;
const agents = [];

function spawnAgent() {
  return {
    x: W * (0.2 + entropy() * 0.6),
    y: H * (0.2 + entropy() * 0.6),
    vx: 0, vy: 0,
    mass: 1 + entropy() * 4,           // 1-5: affects size, inertia
    lifespan: 3000 + entropy() * 60000, // 3s to 63s individual life
    age: 0,
    alpha: 0,                           // individual fade in/out
    alive: true,
    phase: entropy() * Math.PI * 2,
  };
}

for (let i = 0; i < NUM_AGENTS; i++) {
  const a = spawnAgent();
  a.age = entropy() * a.lifespan * 0.5; // stagger initial ages so they don't all die together
  a.alpha = 1;
  agents.push(a);
}

function updateAgentLife(a, dt) {
  a.age += dt;
  // Fade in: first 2 seconds
  if (a.age < 2000) {
    a.alpha = a.age / 2000;
  }
  // Fade out: last 2 seconds
  else if (a.age > a.lifespan - 2000) {
    a.alpha = Math.max(0, (a.lifespan - a.age) / 2000);
  }
  else {
    a.alpha = 1;
  }
  // Death and rebirth
  if (a.age >= a.lifespan) {
    const idx = agents.indexOf(a);
    if (idx !== -1) {
      agents[idx] = spawnAgent();
    }
  }
}
```

- [ ] **Step 2: Add physics update driven by field**

```javascript
function updateAgentPhysics(a, t, dt) {
  const cx = W / 2, cy = H / 2;
  const e = field.energy, o = field.order;
  const speed = (0.3 + e * 3.0) / a.mass; // mass affects responsiveness
  let fx = 0, fy = 0;

  // Flow force (strongest at high energy + low order)
  fx += derived.flowX * speed * 0.6;
  fy += derived.flowY * speed * 0.6;

  // Rotation (from derived)
  const dx = a.x - cx, dy = a.y - cy;
  const dist = Math.sqrt(dx * dx + dy * dy) + 1;
  fx += (-dy / dist) * derived.rotation * speed;
  fy += (dx / dist) * derived.rotation * speed;

  // Cohesion — driven by order: high order → attract to center
  const cohForce = (o - 0.3) * 2;
  fx += (-dx / dist) * cohForce * speed * 0.3;
  fy += (-dy / dist) * cohForce * speed * 0.3;

  // Pulse (radial breathing)
  const pulseWave = Math.sin(t * 0.004 * (0.5 + derived.pulse * 2));
  fx += (dx / dist) * pulseWave * derived.pulse * speed * 0.4;
  fy += (dy / dist) * pulseWave * derived.pulse * speed * 0.4;

  // Mandala force: at high order + low energy, snap toward symmetric positions
  if (o > 0.7 && e < 0.3) {
    const mandalaStrength = (o - 0.7) * (0.3 - e) * 10; // 0 to ~1
    const angle = Math.atan2(dy, dx);
    const symmetry = 8;
    const sector = (Math.PI * 2) / symmetry;
    const snapped = Math.round(angle / sector) * sector;
    const ringDist = 40 + (agents.indexOf(a) % 5) * 45;
    const tx = cx + Math.cos(snapped) * ringDist;
    const ty = cy + Math.sin(snapped) * ringDist;
    fx += (tx - a.x) * mandalaStrength * 0.008;
    fy += (ty - a.y) * mandalaStrength * 0.008;
  }

  // Neighbor awareness — compassion region (mid energy, mid-high order)
  if (o > 0.4 && e < 0.6) {
    for (let j = 0; j < agents.length; j++) {
      const b = agents[j];
      if (b === a || !b.alive) continue;
      const bx = b.x - a.x, by = b.y - a.y;
      const bd = Math.sqrt(bx * bx + by * by);
      if (bd < 80 && bd > 15) {
        // gentle attraction (face toward each other)
        fx += (bx / bd) * 0.05 * o;
        fy += (by / bd) * 0.05 * o;
      }
      if (bd < 15) {
        // separation
        fx -= (bx / bd) * 0.2;
        fy -= (by / bd) * 0.2;
      }
    }
  }

  // Wander
  a.phase += 0.01 + e * 0.02;
  fx += Math.sin(a.phase) * 0.15;
  fy += Math.cos(a.phase * 1.3) * 0.15;

  // Apply — damping inversely related to mass (heavy things keep momentum)
  const damping = 0.90 + (a.mass / 5) * 0.06; // 0.90 to 0.96
  a.vx = a.vx * damping + fx * (1 - damping);
  a.vy = a.vy * damping + fy * (1 - damping);
  a.x += a.vx;
  a.y += a.vy;

  // Wrap
  const m = 60;
  if (a.x < -m) a.x += W + m * 2;
  if (a.x > W + m) a.x -= W + m * 2;
  if (a.y < -m) a.y += H + m * 2;
  if (a.y > H + m) a.y -= H + m * 2;
}
```

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "v3: 40 agents with mass, lifespan, individual life cycles, field-driven behavior"
```

---

### Task 3: Breath, Karma, Macro

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add breath cycle**

```javascript
// === BREATH ===
const BREATH = { BIRTH: 0, LIVING: 1, DYING: 2, SILENCE: 3 };
let breathState = BREATH.BIRTH;
let breathTimer = 0;
let breathAlpha = 0;
let birthDur, livingDur, dyingDur, silenceDur;

function easeInOut(t) { return t < 0.5 ? 2*t*t : -1+(4-2*t)*t; }

function beginLife() {
  const ls = 0.3 + abundance * 1.4;
  const ss = 0.3 + (1 - abundance) * 2;
  birthDur = (2000 + entropy() * 3000) * ls;
  livingDur = (3000 + entropy() * 30000) * ls;  // up to 33s * scale — wider range
  dyingDur = (2000 + entropy() * 4000) * ls;
  silenceDur = (500 + entropy() * 8000) * ss;   // wider: 0.5s to 8.5s * scale
  breathState = BREATH.BIRTH;
  breathTimer = 0;
  chooseNextTarget();
}

function updateBreath(dt) {
  breathTimer += dt;
  switch (breathState) {
    case BREATH.BIRTH:
      breathAlpha = easeInOut(Math.min(breathTimer / birthDur, 1));
      if (breathTimer >= birthDur) { breathState = BREATH.LIVING; breathTimer = 0; }
      break;
    case BREATH.LIVING:
      breathAlpha = 1;
      if (breathTimer >= livingDur) {
        breathState = BREATH.DYING; breathTimer = 0;
        // push toward low energy
        fieldTarget.energy = entropy() * 0.1;
        fieldTarget.order = field.order + (entropy() - 0.5) * 0.3;
        fieldTarget.order = Math.max(0, Math.min(1, fieldTarget.order));
      }
      break;
    case BREATH.DYING:
      breathAlpha = 1 - easeInOut(Math.min(breathTimer / dyingDur, 1));
      if (breathTimer >= dyingDur) {
        stampTrace();
        harvestKarma();
        checkTrueEmptiness();
        breathState = BREATH.SILENCE; breathTimer = 0;
      }
      break;
    case BREATH.SILENCE:
      breathAlpha = 0;
      if (breathTimer >= silenceDur) {
        if (isTrueEmptiness) { traceCanvas.style.opacity = '1'; isTrueEmptiness = false; }
        beginLife();
      }
      break;
  }
}
```

- [ ] **Step 2: Add karma — target biased by current field position**

```javascript
// === KARMA ===
function harvestKarma() {
  // karma is implicit — the current field position IS the karma
  // chooseNextTarget() already reads field.energy and field.order
}
// chooseNextTarget already implements karma bias (70% nearby, 30% jump)
```

- [ ] **Step 3: Add macro rhythm**

```javascript
// === MACRO ===
let abundance = 0.5;
let abundanceTarget = entropy();
let macroTimer = 0;
let macroInterval = 60000 + entropy() * 120000;

function updateMacro(dt) {
  macroTimer += dt;
  if (macroTimer > macroInterval) {
    abundanceTarget = entropy();
    macroTimer = 0;
    macroInterval = 60000 + entropy() * 120000;
  }
  abundance += (abundanceTarget - abundance) * 0.00005 * dt;
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v3: breath cycle, implicit karma, macro rhythm"
```

---

### Task 4: Traces, True Emptiness, Buddha-nature

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add traces**

```javascript
// === TRACES ===
function stampTrace() {
  traceCtx.globalAlpha = 0.2;
  traceCtx.drawImage(mainCanvas, 0, 0);
  traceCtx.globalAlpha = 1.0;
}
function fadeTrace() {
  traceCtx.globalCompositeOperation = 'destination-out';
  traceCtx.fillStyle = 'rgba(0,0,0,0.008)';
  traceCtx.fillRect(0, 0, W, H);
  traceCtx.globalCompositeOperation = 'source-over';
}
```

- [ ] **Step 2: Add true emptiness**

```javascript
// === EMPTINESS ===
let isTrueEmptiness = false;
function checkTrueEmptiness() {
  isTrueEmptiness = entropy() < 0.15;
  if (isTrueEmptiness) {
    silenceDur = 15000 + entropy() * 90000;
    traceCanvas.style.opacity = '0';
  }
}
```

- [ ] **Step 3: Add buddha-nature**

```javascript
// === BUDDHA-NATURE ===
let bnVisible = true, bnTimer = 0, bnVanishDur = 0;
const bnX = innerWidth * (0.15 + Math.random() * 0.7);
const bnY = innerHeight * (0.15 + Math.random() * 0.7);

function updateBN(dt) {
  bnTimer += dt;
  if (!bnVisible) {
    bnVanishDur -= dt;
    if (bnVanishDur <= 0) bnVisible = true;
  } else if (bnTimer > 30000) {
    bnTimer = 0;
    if (entropy() < 0.05) { bnVisible = false; bnVanishDur = 30000 + entropy() * 90000; }
  }
}

function drawBN() {
  if (!bnVisible || (isTrueEmptiness && breathState === BREATH.SILENCE)) return;
  const a = 0.06 + Math.sin(performance.now() * 0.001) * 0.03;
  ctx.fillStyle = `rgba(200,200,210,${a})`;
  ctx.fillRect(bnX, bnY, 1, 1);
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v3: traces, true emptiness, buddha-nature"
```

---

### Task 5: Color + Render + Main Loop

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Add color system with natural gold region**

```javascript
// === COLOR ===
function fieldToHue() {
  const w = derived.warmth;
  // warmth 0→220 (blue), 1→15 (red)
  // but if warmth>0.8, energy 0.5-0.7, order>0.7 → shift toward gold (hue 45)
  let h = 220 - w * 205;
  if (w > 0.8 && field.energy > 0.5 && field.energy < 0.7 && field.order > 0.7) {
    const goldBlend = (w - 0.8) * 5 * (1 - Math.abs(field.energy - 0.6) * 5) * (field.order - 0.7) * 3.3;
    h = h + (45 - h) * Math.min(goldBlend, 1);
  }
  return h;
}
function fieldToSat() { return 15 + field.energy * 45; }
function fieldToLight() { return 35 + field.energy * 25; }
```

- [ ] **Step 2: Add render functions**

```javascript
// === RENDER ===
function renderAgents() {
  const hue = fieldToHue();
  const sat = fieldToSat();
  const light = fieldToLight();

  for (const a of agents) {
    if (!a.alive || a.alpha < 0.01) continue;
    const finalAlpha = a.alpha * breathAlpha;
    if (finalAlpha < 0.005) continue;

    const r = (a.mass * 0.8 + a.mass * field.energy * 0.5);
    const aHue = hue + Math.sin(a.phase) * 8;

    // Core
    ctx.beginPath();
    ctx.arc(a.x, a.y, r, 0, Math.PI * 2);
    ctx.fillStyle = `hsla(${aHue}, ${sat}%, ${light}%, ${(0.4 + field.energy * 0.4) * finalAlpha})`;
    ctx.fill();

    // Trail
    const v = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
    if (v > 0.5) {
      const trailLen = Math.min(v * 4, 25);
      ctx.beginPath();
      ctx.moveTo(a.x, a.y);
      ctx.lineTo(a.x - (a.vx / v) * trailLen, a.y - (a.vy / v) * trailLen);
      ctx.strokeStyle = `hsla(${aHue}, ${sat * 0.7}%, ${light}%, ${Math.min(v * 0.1, 0.3) * finalAlpha})`;
      ctx.lineWidth = r * 0.4;
      ctx.lineCap = 'round';
      ctx.stroke();
    }
  }

  // Connections — visibility driven by order
  if (field.order > 0.3) {
    const connAlpha = (field.order - 0.3) * 0.4;
    for (let i = 0; i < agents.length; i++) {
      if (agents[i].alpha < 0.3) continue;
      for (let j = i + 1; j < agents.length; j++) {
        if (agents[j].alpha < 0.3) continue;
        const dx = agents[i].x - agents[j].x;
        const dy = agents[i].y - agents[j].y;
        const d = Math.sqrt(dx * dx + dy * dy);
        if (d < 100) {
          ctx.beginPath();
          ctx.moveTo(agents[i].x, agents[i].y);
          ctx.lineTo(agents[j].x, agents[j].y);
          ctx.strokeStyle = `hsla(${hue}, ${sat * 0.5}%, ${light}%, ${connAlpha * (1 - d/100) * breathAlpha * Math.min(agents[i].alpha, agents[j].alpha)})`;
          ctx.lineWidth = 0.5;
          ctx.stroke();
        }
      }
    }
  }
}
```

- [ ] **Step 3: Add main loop**

```javascript
// === LOOP ===
// Initialize
beginLife();

let lastTime = performance.now();
function loop(t) {
  const dt = Math.min(t - lastTime, 50); // cap dt to avoid jumps
  lastTime = t;

  updateMacro(dt);
  lerpField(dt);
  updateBreath(dt);
  updateBN(dt);

  // True emptiness: absolute void
  if (breathState === BREATH.SILENCE && isTrueEmptiness) {
    ctx.clearRect(0, 0, W, H);
    requestAnimationFrame(loop);
    return;
  }

  fadeTrace();
  ctx.clearRect(0, 0, W, H);

  if (breathAlpha > 0.001) {
    for (const a of agents) {
      updateAgentLife(a, dt);
      updateAgentPhysics(a, t, dt);
    }
    renderAgents();
  }

  drawBN();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

// Fullscreen on click
document.addEventListener('click', () => {
  if (!document.fullscreenElement) document.documentElement.requestFullscreen().catch(() => {});
});
</script>
</body>
</html>
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v3: color with natural gold, render, main loop — complete rewrite"
```

---

### Task 6: Delete old plan and verify

**Files:**
- Modify: `index.html` (if any issues found during visual verification)

- [ ] **Step 1: Open in browser and verify**

Run: `open index.html`

Check:
- ~40 particles visible, each with distinct sizes (mass)
- Particles die individually (fade out one at a time, not all together)
- New particles born during the life cycle
- Field transitions visible — sometimes gradual, sometimes sudden
- Connection lines appear during high-order phases
- During high order + low energy, particles drift toward symmetric arrangement
- Breath cycle works: fade in, live, fade out, silence, rebirth
- True emptiness occasionally: complete black, no trace
- Buddha-nature pixel present (very subtle)
- Colors shift naturally with field state

- [ ] **Step 2: Commit any fixes**

```bash
git add index.html
git commit -m "v3: visual tuning after browser verification"
```
