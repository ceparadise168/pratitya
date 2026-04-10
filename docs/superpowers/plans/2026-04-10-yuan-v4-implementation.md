# 緣 v4 — 生成式法像 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Rewrite index.html as a generative thangka-inspired artwork — concentric three-layer structure, mineral color palette, pattern elements (arcs, spirals, rays, rings) with trackable 眾生 individuals, polar coordinate system.

**Architecture:** Three-canvas stack (trace/core/main). Core canvas pre-computed with iconometric grid. Main canvas renders 30-50 background patterns + 5-10 眾生 patterns driven by 2D continuous field (energy × order). Breath cycle, macro rhythm, traces, true emptiness, buddha-nature carried forward.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, Canvas 2D API, no dependencies.

**Approach:** Complete rewrite. The visual language, spatial structure, and rendering are entirely different from v3.

---

## File Structure

Single file: `index.html`

Three canvases: `#trace` (z:0), `#core` (z:1), `#main` (z:2)

---

### Task 1: Three-Canvas Shell + Entropy + Field + Macro

Create the HTML structure and carry forward field/entropy/macro from v3.

- [ ] **Step 1: Write the complete HTML shell**

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
#core  { z-index: 1; }
#main  { z-index: 2; }
</style>
</head>
<body>
<canvas id="trace"></canvas>
<canvas id="core"></canvas>
<canvas id="main"></canvas>
<script>
// === CANVAS ===
const traceCanvas = document.getElementById('trace');
const coreCanvas = document.getElementById('core');
const mainCanvas = document.getElementById('main');
const traceCtx = traceCanvas.getContext('2d');
const coreCtx = coreCanvas.getContext('2d');
const ctx = mainCanvas.getContext('2d');
let W, H, CX, CY, R;

function resize() {
  W = traceCanvas.width = coreCanvas.width = mainCanvas.width = innerWidth;
  H = traceCanvas.height = coreCanvas.height = mainCanvas.height = innerHeight;
  CX = W / 2;
  CY = H / 2;
  R = Math.min(W, H) * 0.45; // max radius for the mandala
  drawCoreGrid();
}
resize();
addEventListener('resize', resize);
```

- [ ] **Step 2: Add entropy, field, macro** (identical to v3)

```javascript
// === ENTROPY ===
function entropy() {
  const t = performance.now();
  const s = Math.sin(t * 12345.6789 + t * t * 0.001) * 0.5 + 0.5;
  return (Math.random() + s) % 1;
}

// === FIELD ===
const field = { energy: 0.3, order: 0.5 };
const fieldTarget = { energy: 0.3, order: 0.5 };
const derived = { warmth: 0.5, rotation: 0, flowX: 0, flowY: 0, pulse: 0 };

function deriveParams() {
  const e = field.energy, o = field.order;
  derived.warmth = e * 0.6 + (1 - o) * 0.4;
  derived.rotation = (entropy() > 0.5 ? 1 : -1) * e * (1 - Math.abs(o - 0.5) * 2) * 0.8;
  const flowStrength = e * (1 - o) * 1.5;
  derived.flowX = (entropy() - 0.5) * 2 * flowStrength;
  derived.flowY = (entropy() - 0.5) * 2 * flowStrength;
  derived.pulse = (1 - Math.abs(e - 0.5) * 2) * 0.6;
}

function lerpField(dt) {
  const de = fieldTarget.energy - field.energy;
  const dor = fieldTarget.order - field.order;
  const dist = Math.sqrt(de * de + dor * dor);
  const rate = (0.0003 + dist * dist * 0.005) * dt;
  field.energy += de * rate;
  field.order += dor * rate;
}

function chooseNextTarget() {
  const isJump = entropy() < 0.3;
  const range = isJump ? 0.8 : 0.3;
  let ne = field.energy + (entropy() - 0.5) * 2 * range;
  let no = field.order + (entropy() - 0.5) * 2 * range;
  ne = Math.max(0, Math.min(1, ne));
  no = Math.max(0, Math.min(1, no));
  if (Math.abs(ne - fieldTarget.energy) < 0.05 && Math.abs(no - fieldTarget.order) < 0.05) {
    ne = entropy(); no = entropy();
  }
  fieldTarget.energy = ne;
  fieldTarget.order = no;
  deriveParams();
}

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

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "v4: three-canvas shell + entropy + field + macro"
```

---

### Task 2: Core Grid + Color Palette

Pre-computed iconometric grid on the core canvas. Mineral color palette.

- [ ] **Step 1: Add color palette**

```javascript
// === PALETTE ===
const PALETTE = {
  deepBlue:  { h: 220, s: 75, l: 25 },
  vermilion: { h: 5,   s: 80, l: 45 },
  emerald:   { h: 155, s: 65, l: 30 },
  gold:      { h: 45,  s: 85, l: 55 },
  purple:    { h: 280, s: 60, l: 25 },
  warmWhite: { h: 40,  s: 30, l: 85 },
};

// Color pairs — strict pairing, no gradients
const PAIRS = [
  ['gold', 'deepBlue'],
  ['vermilion', 'emerald'],
  ['warmWhite', 'purple'],
  ['gold', 'vermilion'], // rare — peak splendor
];

function hsl(color, alpha) {
  const c = PALETTE[color];
  return `hsla(${c.h}, ${c.s}%, ${c.l}%, ${alpha})`;
}

// Field → palette selection
function fieldPrimaryColor() {
  if (field.energy > 0.6) return derived.warmth > 0.6 ? 'vermilion' : 'gold';
  if (field.order > 0.6) return 'gold';
  if (field.energy < 0.3) return field.order > 0.5 ? 'deepBlue' : 'purple';
  return 'deepBlue';
}

function fieldSecondaryColor() {
  const primary = fieldPrimaryColor();
  const pair = PAIRS.find(p => p.includes(primary));
  return pair ? pair.find(c => c !== primary) : 'deepBlue';
}

// Color saturation scales with time — first cycles are muted
let colorMaturity = 0; // 0 to 1, grows over first few breath cycles
function updateColorMaturity(dt) {
  if (colorMaturity < 1) colorMaturity = Math.min(1, colorMaturity + dt * 0.00003);
}
```

- [ ] **Step 2: Add core grid drawing**

```javascript
// === CORE GRID ===
function drawCoreGrid() {
  coreCtx.clearRect(0, 0, W, H);
  coreCtx.strokeStyle = hsl('gold', 0.1);
  coreCtx.lineWidth = 0.5;

  // Concentric circles (golden ratio spacing)
  const phi = 1.618;
  let cr = R * 0.05;
  while (cr < R) {
    coreCtx.beginPath();
    coreCtx.arc(CX, CY, cr, 0, Math.PI * 2);
    coreCtx.stroke();
    cr *= phi * 0.6; // ~0.97 ratio — tighter than pure golden
    if (cr < R * 0.05 * 1.2) cr = R * 0.05 * phi; // skip too-small gaps
  }

  // Radial lines (16-fold)
  const spokes = 16;
  for (let i = 0; i < spokes; i++) {
    const angle = (i / spokes) * Math.PI * 2;
    coreCtx.beginPath();
    coreCtx.moveTo(CX, CY);
    coreCtx.lineTo(CX + Math.cos(angle) * R * 0.8, CY + Math.sin(angle) * R * 0.8);
    coreCtx.stroke();
  }

  // Cross axes (slightly brighter)
  coreCtx.strokeStyle = hsl('gold', 0.14);
  coreCtx.lineWidth = 0.7;
  coreCtx.beginPath();
  coreCtx.moveTo(CX - R * 0.8, CY); coreCtx.lineTo(CX + R * 0.8, CY);
  coreCtx.moveTo(CX, CY - R * 0.8); coreCtx.lineTo(CX, CY + R * 0.8);
  coreCtx.stroke();

  // Proportion arcs — hints of the iconometric body structure
  coreCtx.strokeStyle = hsl('gold', 0.07);
  coreCtx.lineWidth = 0.4;
  const arcRadii = [R * 0.15, R * 0.25, R * 0.4, R * 0.6];
  for (const ar of arcRadii) {
    // Quarter arcs at cardinal points
    for (let q = 0; q < 4; q++) {
      const startAngle = (q / 4) * Math.PI * 2 - Math.PI / 8;
      coreCtx.beginPath();
      coreCtx.arc(CX, CY, ar, startAngle, startAngle + Math.PI / 4);
      coreCtx.stroke();
    }
  }
}
```

- [ ] **Step 3: Add core grid breathing**

The core grid's alpha oscillates very slowly. Instead of re-drawing each frame, modulate the canvas opacity via CSS:

```javascript
let coreBreathPhase = 0;
function updateCoreBreath(dt) {
  coreBreathPhase += dt * 0.0001; // full cycle ~60s
  const alpha = 0.08 + Math.sin(coreBreathPhase) * 0.035; // 0.045 to 0.115
  coreCanvas.style.opacity = alpha / 0.1; // normalize around 1.0
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v4: core grid (iconometric geometry) + mineral color palette"
```

---

### Task 3: Pattern System — Background Patterns

The heart of v4. Background patterns in polar coordinates.

- [ ] **Step 1: Add pattern data structures and spawning**

```javascript
// === PATTERNS ===
const bgPatterns = [];
const beingPatterns = [];

function spawnBgPattern() {
  const types = ['arc', 'spiral', 'ray', 'ring'];
  // Weight by field state
  const weights = [
    1 + field.order * 2,          // arc: favored by order
    1 + field.energy * 2,         // spiral: favored by energy
    0.5 + field.energy * (1 - field.order), // ray: favored by energy+chaos
    0.5 + field.order,            // ring: favored by order
  ];
  const total = weights.reduce((a, b) => a + b, 0);
  let r = entropy() * total;
  let type = types[0];
  for (let i = 0; i < weights.length; i++) {
    r -= weights[i];
    if (r <= 0) { type = types[i]; break; }
  }

  const dist = R * (0.1 + entropy() * 0.7); // spawn within activity band
  const theta = entropy() * Math.PI * 2;
  const maxLife = 3000 + entropy() * 15000; // 3-18s

  return {
    type,
    r: dist,
    theta,
    age: 0,
    maxLife,
    alpha: 0,
    // Type-specific params
    arcSpan: 0.2 + entropy() * 0.6,     // for arc: angular span
    spiralTurns: 1 + entropy() * 2,      // for spiral: number of turns
    rayLength: R * (0.1 + entropy() * 0.3), // for ray: length
    ringSpan: 0.3 + entropy() * 1.2,     // for ring: angular span
    lineWidth: 0.5 + entropy() * 1.5,
    colorKey: entropy() > 0.5 ? fieldPrimaryColor() : fieldSecondaryColor(),
    rotSpeed: (entropy() - 0.5) * 0.0003, // slow drift
    isBeing: false,
  };
}
```

- [ ] **Step 2: Add pattern lifecycle update**

```javascript
function updatePatternLife(p, dt) {
  p.age += dt;
  p.theta += p.rotSpeed * dt;
  // Fade in/out
  const fadeTime = 1500;
  if (p.age < fadeTime) {
    p.alpha = p.age / fadeTime;
  } else if (p.age > p.maxLife - fadeTime) {
    p.alpha = Math.max(0, (p.maxLife - p.age) / fadeTime);
  } else {
    p.alpha = 1;
  }
  return p.age < p.maxLife;
}

function managePatterns(dt) {
  // Update existing
  for (let i = bgPatterns.length - 1; i >= 0; i--) {
    if (!updatePatternLife(bgPatterns[i], dt)) bgPatterns.splice(i, 1);
  }
  // Spawn to maintain density based on field
  const targetCount = Math.floor(15 + field.energy * 25 + field.order * 10);
  while (bgPatterns.length < targetCount) bgPatterns.push(spawnBgPattern());
}
```

- [ ] **Step 3: Add pattern rendering**

```javascript
function renderPattern(p) {
  if (p.alpha < 0.01) return;
  const x = CX + Math.cos(p.theta) * p.r;
  const y = CY + Math.sin(p.theta) * p.r;
  const fadeFromCenter = Math.min(1, p.r / (R * 0.2)); // fade near center
  const fadeAtEdge = Math.max(0, 1 - (p.r / R - 0.7) / 0.3); // fade at boundary
  const a = p.alpha * breathAlpha * colorMaturity * fadeFromCenter * Math.min(1, fadeAtEdge);
  if (a < 0.005) return;

  ctx.strokeStyle = hsl(p.colorKey, a * 0.7);
  ctx.lineWidth = p.lineWidth;
  ctx.lineCap = 'round';

  switch (p.type) {
    case 'arc': {
      const startAngle = p.theta - p.arcSpan / 2;
      const endAngle = p.theta + p.arcSpan / 2;
      ctx.beginPath();
      ctx.arc(CX, CY, p.r, startAngle, endAngle);
      ctx.stroke();
      // Mirror petal
      ctx.beginPath();
      ctx.arc(CX, CY, p.r * 0.92, startAngle + 0.05, endAngle - 0.05);
      ctx.stroke();
      break;
    }
    case 'spiral': {
      ctx.beginPath();
      const steps = 30;
      for (let s = 0; s <= steps; s++) {
        const t = s / steps;
        const sr = p.r - t * p.r * 0.3;
        const sa = p.theta + t * p.spiralTurns * Math.PI * 0.5;
        const sx = CX + Math.cos(sa) * sr;
        const sy = CY + Math.sin(sa) * sr;
        s === 0 ? ctx.moveTo(sx, sy) : ctx.quadraticCurveTo(
          CX + Math.cos(sa - 0.1) * (sr + 5), CY + Math.sin(sa - 0.1) * (sr + 5),
          sx, sy
        );
      }
      ctx.stroke();
      break;
    }
    case 'ray': {
      ctx.beginPath();
      ctx.moveTo(x, y);
      const endR = p.r + p.rayLength;
      ctx.lineTo(CX + Math.cos(p.theta) * endR, CY + Math.sin(p.theta) * endR);
      ctx.stroke();
      break;
    }
    case 'ring': {
      ctx.beginPath();
      ctx.arc(CX, CY, p.r, p.theta, p.theta + p.ringSpan);
      ctx.stroke();
      break;
    }
  }
}

function renderAllBgPatterns() {
  for (const p of bgPatterns) renderPattern(p);
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v4: background pattern system — arcs, spirals, rays, rings in polar coords"
```

---

### Task 4: 眾生紋樣 (Being Patterns)

Trackable individuals with bezierCurveTo, unique signatures, and death events.

- [ ] **Step 1: Add being pattern spawning**

```javascript
// === BEINGS ===
function spawnBeing(inheritR, inheritTheta) {
  const dist = inheritR || R * (0.15 + entropy() * 0.5);
  const theta = inheritTheta || entropy() * Math.PI * 2;
  return {
    type: ['arc', 'spiral'][Math.floor(entropy() * 2)],
    r: dist,
    theta,
    age: 0,
    maxLife: 15000 + entropy() * 45000, // 15-60s — longer than background
    alpha: 0,
    // Unique signature
    arcSpan: 0.3 + entropy() * 0.5,
    spiralTurns: 1.5 + entropy() * 1.5,
    lineWidth: 1 + entropy() * 1.5, // slightly thicker than background
    colorKey: fieldPrimaryColor(),
    rotSpeed: (entropy() - 0.5) * 0.0002,
    isBeing: true,
    // Signature variations — what makes this one unique
    curveBias: (entropy() - 0.5) * 20,   // unique curve personality
    sizeScale: 0.8 + entropy() * 0.4,     // unique size
    phaseOffset: entropy() * Math.PI * 2, // unique animation phase
    dying: false,
    deathTimer: 0,
    fragments: [], // for death animation
  };
}
```

- [ ] **Step 2: Add being rendering with bezierCurveTo**

```javascript
function renderBeing(b) {
  if (b.alpha < 0.01 && !b.dying) return;
  const fadeFromCenter = Math.min(1, b.r / (R * 0.15));
  const a = b.alpha * breathAlpha * colorMaturity * fadeFromCenter;
  if (a < 0.005 && !b.dying) return;

  ctx.strokeStyle = hsl(b.colorKey, a * 0.85);
  ctx.lineWidth = b.lineWidth * b.sizeScale;
  ctx.lineCap = 'round';

  if (b.dying) {
    // Death animation: fragments scatter
    for (const f of b.fragments) {
      f.r += f.vr * 0.5;
      f.theta += f.vt;
      f.alpha *= 0.97;
      if (f.alpha < 0.01) continue;
      const fx = CX + Math.cos(f.theta) * f.r;
      const fy = CY + Math.sin(f.theta) * f.r;
      ctx.strokeStyle = hsl(b.colorKey, f.alpha * breathAlpha);
      ctx.lineWidth = f.width;
      ctx.beginPath();
      ctx.moveTo(fx, fy);
      ctx.lineTo(fx + Math.cos(f.angle) * f.len, fy + Math.sin(f.angle) * f.len);
      ctx.stroke();
    }
    return;
  }

  // Living: draw with cubic bezier for higher quality
  const x = CX + Math.cos(b.theta) * b.r;
  const y = CY + Math.sin(b.theta) * b.r;

  if (b.type === 'arc') {
    const span = b.arcSpan * b.sizeScale;
    const steps = 8;
    ctx.beginPath();
    for (let s = 0; s <= steps; s++) {
      const t = s / steps;
      const angle = b.theta - span / 2 + t * span;
      const wobble = Math.sin(t * Math.PI * 3 + b.phaseOffset + b.age * 0.001) * b.curveBias * 0.1;
      const pr = b.r + wobble;
      const px = CX + Math.cos(angle) * pr;
      const py = CY + Math.sin(angle) * pr;
      if (s === 0) { ctx.moveTo(px, py); continue; }
      // Cubic bezier for smooth S-curves
      const prevAngle = b.theta - span / 2 + ((s - 1) / steps) * span;
      const cpR = b.r + b.curveBias * 0.3;
      const cp1x = CX + Math.cos(prevAngle + span / steps * 0.3) * cpR;
      const cp1y = CY + Math.sin(prevAngle + span / steps * 0.3) * cpR;
      const cp2x = CX + Math.cos(angle - span / steps * 0.3) * cpR;
      const cp2y = CY + Math.sin(angle - span / steps * 0.3) * cpR;
      ctx.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, px, py);
    }
    ctx.stroke();
  } else {
    // Spiral with cubic bezier
    ctx.beginPath();
    const steps = 20;
    for (let s = 0; s <= steps; s++) {
      const t = s / steps;
      const sr = b.r * (1 - t * 0.3) * b.sizeScale;
      const sa = b.theta + t * b.spiralTurns * Math.PI * 0.5;
      const wobble = Math.sin(t * Math.PI * 2 + b.phaseOffset + b.age * 0.0008) * b.curveBias * 0.15;
      const sx = CX + Math.cos(sa) * (sr + wobble);
      const sy = CY + Math.sin(sa) * (sr + wobble);
      if (s === 0) { ctx.moveTo(sx, sy); continue; }
      const prevT = (s - 1) / steps;
      const prevR = b.r * (1 - prevT * 0.3) * b.sizeScale;
      const prevA = b.theta + prevT * b.spiralTurns * Math.PI * 0.5;
      const cpR1 = (prevR + sr) / 2 + b.curveBias * 0.2;
      const cpA1 = (prevA + sa) / 2;
      ctx.quadraticCurveTo(
        CX + Math.cos(cpA1) * cpR1, CY + Math.sin(cpA1) * cpR1, sx, sy
      );
    }
    ctx.stroke();
  }
}
```

- [ ] **Step 3: Add being lifecycle and death animation**

```javascript
function updateBeing(b, dt) {
  if (b.dying) {
    b.deathTimer += dt;
    return b.deathTimer < 3000; // death animation lasts 3s
  }
  b.age += dt;
  b.theta += b.rotSpeed * dt;
  // Fade in/out
  if (b.age < 2500) {
    b.alpha = b.age / 2500; // slower fade in than background — more deliberate
  } else if (b.age > b.maxLife - 2000) {
    b.alpha = Math.max(0, (b.maxLife - b.age) / 2000);
  } else {
    b.alpha = 1;
  }
  // Trigger death
  if (b.age >= b.maxLife && !b.dying) {
    b.dying = true;
    b.deathTimer = 0;
    // Create fragments
    const numFrags = 5 + Math.floor(entropy() * 8);
    for (let i = 0; i < numFrags; i++) {
      b.fragments.push({
        r: b.r + (entropy() - 0.5) * 20,
        theta: b.theta + (entropy() - 0.5) * b.arcSpan,
        vr: (entropy() - 0.5) * 2,
        vt: (entropy() - 0.5) * 0.01,
        angle: entropy() * Math.PI * 2,
        len: 3 + entropy() * 10,
        width: 0.3 + entropy() * 0.8,
        alpha: 0.6 + entropy() * 0.4,
      });
    }
    // Stamp to trace — only beings leave marks
    stampBeingTrace(b);
  }
  return true;
}

function stampBeingTrace(b) {
  // Draw the being's last form onto trace canvas
  const savedCtx = ctx;
  // Temporarily render to trace
  const prevStroke = traceCtx.strokeStyle;
  traceCtx.globalAlpha = 0.2;
  // Simple stamp: draw an arc at the being's position
  traceCtx.strokeStyle = hsl(b.colorKey, 0.3);
  traceCtx.lineWidth = b.lineWidth * 0.5;
  traceCtx.beginPath();
  traceCtx.arc(CX, CY, b.r, b.theta - b.arcSpan / 2, b.theta + b.arcSpan / 2);
  traceCtx.stroke();
  traceCtx.globalAlpha = 1.0;
}

function manageBeings(dt) {
  for (let i = beingPatterns.length - 1; i >= 0; i--) {
    if (!updateBeing(beingPatterns[i], dt)) {
      const dead = beingPatterns[i];
      // Spawn next being influenced by the dead one's position
      beingPatterns[i] = spawnBeing(
        dead.r + (entropy() - 0.5) * R * 0.2,
        dead.theta + (entropy() - 0.5) * 0.5
      );
    }
  }
  // Maintain 5-8 beings
  const targetBeings = 5 + Math.floor(field.energy * 3);
  while (beingPatterns.length < targetBeings) beingPatterns.push(spawnBeing());
  while (beingPatterns.length > targetBeings + 2) beingPatterns.pop();
}

function renderAllBeings() {
  for (const b of beingPatterns) renderBeing(b);
}
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "v4: 眾生紋樣 — trackable beings with bezier curves and death fragments"
```

---

### Task 5: Breath + Traces + Emptiness + Buddha-nature + Loop

Assemble the full lifecycle and main loop.

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
  livingDur = (5000 + entropy() * 30000) * ls;
  dyingDur = (3000 + entropy() * 5000) * ls;
  silenceDur = (500 + entropy() * 8000) * ss;
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
        fieldTarget.energy = entropy() * 0.1;
        fieldTarget.order = field.order + (entropy() - 0.5) * 0.3;
        fieldTarget.order = Math.max(0, Math.min(1, fieldTarget.order));
      }
      break;
    case BREATH.DYING:
      breathAlpha = 1 - easeInOut(Math.min(breathTimer / dyingDur, 1));
      if (breathTimer >= dyingDur) {
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

- [ ] **Step 2: Add traces, emptiness, buddha-nature**

```javascript
// === TRACES ===
let traceFadeTimer = 0;
function fadeTrace(dt) {
  traceFadeTimer += dt;
  if (traceFadeTimer < 400) return; // ~2.5 times per second
  traceFadeTimer = 0;
  traceCtx.globalCompositeOperation = 'destination-out';
  traceCtx.fillStyle = 'rgba(0,0,0,0.01)';
  traceCtx.fillRect(0, 0, W, H);
  traceCtx.globalCompositeOperation = 'source-over';
}

// === EMPTINESS ===
let isTrueEmptiness = false;
function checkTrueEmptiness() {
  isTrueEmptiness = entropy() < 0.15;
  if (isTrueEmptiness) {
    silenceDur = 15000 + entropy() * 90000;
    traceCanvas.style.opacity = '0';
  }
}

// === BUDDHA-NATURE ===
let bnVisible = true, bnTimer = 0, bnVanishDur = 0;
const bnX = innerWidth * (0.3 + Math.random() * 0.4);
const bnY = innerHeight * (0.3 + Math.random() * 0.4);
const BN_SIZE = 3;

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
  // Buddha-nature: first to appear after void, last to vanish
  let bnAlpha;
  if (breathState === BREATH.BIRTH && breathTimer < 1000) {
    bnAlpha = breathTimer / 1000; // appears in first second of birth
  } else if (breathState === BREATH.DYING && breathTimer > dyingDur - 1000) {
    bnAlpha = (dyingDur - breathTimer) / 1000; // last second of dying
  } else if (breathState === BREATH.SILENCE) {
    bnAlpha = 0;
  } else {
    bnAlpha = 1;
  }
  const pulseAlpha = 0.05 + Math.sin(performance.now() * 0.0008) * 0.035;
  const a = pulseAlpha * bnAlpha;
  if (a < 0.005) return;

  ctx.fillStyle = hsl('gold', a);
  ctx.beginPath();
  ctx.arc(bnX, bnY, BN_SIZE, 0, Math.PI * 2);
  ctx.fill();
}
```

- [ ] **Step 3: Add main loop**

```javascript
// === LOOP ===
beginLife();

let lastTime = performance.now();
function loop(t) {
  const dt = Math.min(t - lastTime, 50);
  lastTime = t;

  updateMacro(dt);
  lerpField(dt);
  updateBreath(dt);
  updateBN(dt);
  updateCoreBreath(dt);
  updateColorMaturity(dt);

  // True emptiness
  if (breathState === BREATH.SILENCE && isTrueEmptiness) {
    ctx.clearRect(0, 0, W, H);
    requestAnimationFrame(loop);
    return;
  }

  fadeTrace(dt);
  ctx.clearRect(0, 0, W, H);

  if (breathAlpha > 0.001) {
    managePatterns(dt);
    manageBeings(dt);
    renderAllBgPatterns();
    renderAllBeings();
  }

  drawBN();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

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
git commit -m "v4: breath, traces, emptiness, buddha-nature, main loop — complete"
```

---

### Task 6: Visual Verification

- [ ] **Step 1: Open in browser**

Run: `open index.html`

Check:
- Core grid visible as faint gold geometry (concentric circles + radial lines)
- Background patterns (arcs, spirals, rays, rings) appearing and disappearing
- 5-8 beings visible — distinguishable from background by line quality
- Being deaths: fragments scatter outward then fade
- Being deaths leave marks on trace canvas
- Color starts muted, becomes richer over time
- Breath cycle: fade in, live, fade out, silence, rebirth
- Occasional true emptiness: complete black
- Buddha-nature golden dot visible (subtle)
- Mineral colors: deep blue, vermilion, emerald, gold visible at high energy
- Concentric structure: more ordered near center, more chaotic toward edge

- [ ] **Step 2: Commit any fixes**

```bash
git add index.html
git commit -m "v4: visual tuning after browser verification"
```
