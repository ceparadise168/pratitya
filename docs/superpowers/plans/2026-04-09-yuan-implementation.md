# 緣 (Yuán) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a generative visual art piece where a living field with emotional aspects (佛相) drives agent behavior through breathing life cycles, producing unrepeatable visual experiences that embody Buddhist impermanence.

**Architecture:** Single HTML file with Canvas 2D. The system has three layers: a Field (hidden parameters), Agents (visible particles driven by the field), and a Trace canvas (fading ghost of past lives). An Aspect system drives the field through emotional states (忿怒/慈悲/歡喜/禪定/寂靜), a Breath cycle manages life/death transitions, and a Macro rhythm creates larger temporal patterns. Thangka visual grammar (aureole, mandala, flame, gold) emerges under specific field conditions.

**Tech Stack:** Vanilla HTML/CSS/JavaScript, Canvas 2D API, no dependencies.

**Note:** This is an art project, not a software project. There are no unit tests. Each task's verification is visual — open the file in a browser and observe the described behavior. The art requires extensive tuning; parameter values in the plan are starting points to be refined by feel.

---

## File Structure

All code lives in a single file:

- **Create:** `index.html` — the complete artwork

The file is organized into clearly commented sections within a single `<script>` tag:

```
index.html
├── <style>        — fullscreen black canvas, no UI
├── <canvas id="main">   — primary render target
├── <canvas id="trace">  — ghost/trace layer (behind main)
├── <script>
│   ├── // === CANVAS SETUP ===
│   ├── // === FIELD ===           — hidden parameters
│   ├── // === ASPECT 佛相 ===     — emotional state machine
│   ├── // === BREATH 呼吸 ===     — life cycle (birth/live/die/silence)
│   ├── // === MACRO 劫 ===        — rain/drought seasons
│   ├── // === KARMA 因緣 ===      — residual conditions
│   ├── // === AGENTS 眾生 ===     — particle system
│   ├── // === COLOR 色 ===        — field-to-color mapping
│   ├── // === THANGKA 唐卡 ===    — aureole, mandala, flame, gold
│   ├── // === TRACES 痕跡 ===     — ghost layer management
│   ├── // === BUDDHA-NATURE 佛性 === — persistent witness pixel
│   └── // === MAIN LOOP ===       — orchestration
```

---

### Task 1: Canvas Scaffolding + Static Agents

**Files:**
- Create: `index.html`

Set up the dual-canvas structure and render static agents as proof of life.

- [ ] **Step 1: Create the HTML shell with dual canvas**

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
canvas {
  position: fixed; top: 0; left: 0;
  width: 100vw; height: 100vh;
}
#trace { z-index: 0; }
#main  { z-index: 1; }
</style>
</head>
<body>
<canvas id="trace"></canvas>
<canvas id="main"></canvas>
<script>
// === CANVAS SETUP ===
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

// === AGENTS 眾生 ===
const NUM_AGENTS = 200;
const agents = [];
for (let i = 0; i < NUM_AGENTS; i++) {
  agents.push({
    x: Math.random() * innerWidth,
    y: Math.random() * innerHeight,
    vx: 0,
    vy: 0,
    size: 1 + Math.random() * 2,
    phase: Math.random() * Math.PI * 2,
  });
}

// === MAIN LOOP ===
function loop(t) {
  ctx.clearRect(0, 0, W, H);

  for (const a of agents) {
    ctx.beginPath();
    ctx.arc(a.x, a.y, a.size, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(180, 180, 200, 0.6)';
    ctx.fill();
  }

  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
</script>
</body>
</html>
```

- [ ] **Step 2: Verify in browser**

Run: `open index.html`
Expected: Fullscreen black background with ~200 static white-ish dots scattered randomly. No UI, no cursor. Two canvases stacked (trace behind main).

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "scaffold: dual canvas with static agents"
```

---

### Task 2: Field System

**Files:**
- Modify: `index.html`

Add the hidden field with 6 parameters and smooth interpolation toward target values.

- [ ] **Step 1: Add the Field object and lerp logic between CANVAS SETUP and AGENTS**

```javascript
// === FIELD ===
const field = {
  energy: 0.5,
  warmth: 0.5,
  rotation: 0,
  flowX: 0,
  flowY: 0,
  cohesion: 0.5,
  pulse: 0,
};

const fieldTarget = {
  energy: 0.5,
  warmth: 0.5,
  rotation: 0,
  flowX: 0,
  flowY: 0,
  cohesion: 0.5,
  pulse: 0,
};

function lerpField(dt) {
  const keys = Object.keys(field);
  for (const k of keys) {
    field[k] += (fieldTarget[k] - field[k]) * 0.0005 * dt;
  }
}

function randomizeFieldTarget() {
  fieldTarget.energy = Math.random();
  fieldTarget.warmth = Math.random();
  fieldTarget.rotation = (Math.random() - 0.5) * 2;
  fieldTarget.flowX = (Math.random() - 0.5) * 2;
  fieldTarget.flowY = (Math.random() - 0.5) * 2;
  fieldTarget.cohesion = Math.random();
  fieldTarget.pulse = Math.random();
}
```

- [ ] **Step 2: Call lerpField in the main loop**

Update the loop to track delta time and call `lerpField(dt)` each frame. Add a temporary debug overlay (top-left, small, dim text) showing field values so you can see them changing. This debug overlay will be removed later.

```javascript
let lastTime = performance.now();

function loop(t) {
  const dt = t - lastTime;
  lastTime = t;

  lerpField(dt);

  ctx.clearRect(0, 0, W, H);

  // agents draw (existing)...

  // Temporary debug
  ctx.fillStyle = 'rgba(255,255,255,0.15)';
  ctx.font = '11px monospace';
  const keys = Object.keys(field);
  for (let i = 0; i < keys.length; i++) {
    ctx.fillText(`${keys[i]}: ${field[keys[i]].toFixed(3)}`, 10, 20 + i * 14);
  }

  requestAnimationFrame(loop);
}
```

- [ ] **Step 3: Test field interpolation**

Add a temporary `setInterval(() => randomizeFieldTarget(), 5000)` call before the loop starts. Open the file. Expected: debug overlay shows field values slowly drifting toward new targets every 5 seconds. Remove the setInterval after verifying (the Aspect system will drive this later).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add field system with smooth interpolation"
```

---

### Task 3: Agent Behavior Driven by Field

**Files:**
- Modify: `index.html`

Make agents respond to field parameters: flow pushes them, rotation swirls them, cohesion draws them to/from center, pulse creates radial breathing, and energy scales speed.

- [ ] **Step 1: Add updateAgents function before MAIN LOOP**

```javascript
function updateAgents(t, dt) {
  const cx = W / 2, cy = H / 2;
  const speed = 0.3 + field.energy * 2.5;
  const pulseWave = Math.sin(t * 0.003 * (0.5 + field.pulse * 2));

  for (const a of agents) {
    let fx = 0, fy = 0;

    // Flow
    fx += field.flowX * speed * 0.5;
    fy += field.flowY * speed * 0.5;

    // Rotation around center
    const dx = a.x - cx, dy = a.y - cy;
    const dist = Math.sqrt(dx * dx + dy * dy) + 1;
    fx += (-dy / dist) * field.rotation * speed * 0.8;
    fy += (dx / dist) * field.rotation * speed * 0.8;

    // Cohesion (toward/away from center)
    const cohesionForce = (field.cohesion - 0.5) * 2;
    fx += (-dx / dist) * cohesionForce * speed * 0.3;
    fy += (-dy / dist) * cohesionForce * speed * 0.3;

    // Pulse (radial breathing)
    fx += (dx / dist) * pulseWave * field.pulse * speed * 0.4;
    fy += (dy / dist) * pulseWave * field.pulse * speed * 0.4;

    // Flocking: separation + alignment (sample every 3rd for perf)
    let sepX = 0, sepY = 0, alignX = 0, alignY = 0, nearCount = 0;
    for (let j = 0; j < agents.length; j += 3) {
      const b = agents[j];
      if (b === a) continue;
      const bx = b.x - a.x, by = b.y - a.y;
      const bd = Math.sqrt(bx * bx + by * by);
      if (bd < 50) {
        nearCount++;
        if (bd < 20) { sepX -= bx / bd; sepY -= by / bd; }
        alignX += b.vx; alignY += b.vy;
      }
    }
    if (nearCount > 0) {
      fx += sepX * 0.3;
      fy += sepY * 0.3;
      fx += (alignX / nearCount - a.vx) * 0.05;
      fy += (alignY / nearCount - a.vy) * 0.05;
    }

    // Individual wander
    a.phase += 0.01 + field.energy * 0.02;
    fx += Math.sin(a.phase) * 0.2;
    fy += Math.cos(a.phase * 1.3) * 0.2;

    // Apply forces
    a.vx = a.vx * 0.95 + fx * 0.05;
    a.vy = a.vy * 0.95 + fy * 0.05;
    a.x += a.vx;
    a.y += a.vy;

    // Wrap edges
    const margin = 50;
    if (a.x < -margin) a.x += W + margin * 2;
    if (a.x > W + margin) a.x -= W + margin * 2;
    if (a.y < -margin) a.y += H + margin * 2;
    if (a.y > H + margin) a.y -= H + margin * 2;
  }
}
```

- [ ] **Step 2: Call updateAgents in the main loop before drawing**

```javascript
updateAgents(t, dt);
```

- [ ] **Step 3: Re-add temporary randomizeFieldTarget interval and verify**

Add `setInterval(() => randomizeFieldTarget(), 6000)` temporarily. Open the file. Expected: agents move in response to field — sometimes flowing in a direction, sometimes swirling, sometimes clustering toward center, sometimes dispersing. Movement should feel organic, not mechanical.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: agents respond to field forces (flow, rotation, cohesion, pulse, flocking)"
```

---

### Task 4: Color System

**Files:**
- Modify: `index.html`

Derive agent colors from field state. Warmth controls hue (cold→blue, warm→red), energy controls brightness, and cohesion controls connection line visibility.

- [ ] **Step 1: Add color derivation section and update agent drawing**

Add before MAIN LOOP:

```javascript
// === COLOR 色 ===
function fieldToHue() {
  // warmth 0 → hue 220 (cold blue), warmth 1 → hue 15 (warm red)
  return 220 - field.warmth * 205;
}

function fieldToSaturation() {
  // energy drives saturation: low energy → desaturated, high → vivid
  return 15 + field.energy * 45;
}

function fieldToLightness() {
  return 35 + field.energy * 25;
}
```

Update the agent drawing in the loop to replace the static gray:

```javascript
ctx.clearRect(0, 0, W, H);

const hue = fieldToHue();
const sat = fieldToSaturation();
const light = fieldToLightness();

for (const a of agents) {
  const r = a.size * (0.8 + field.energy * 0.6);

  // Agent dot
  ctx.beginPath();
  ctx.arc(a.x, a.y, r, 0, Math.PI * 2);
  ctx.fillStyle = `hsla(${hue + a.phase * 3}, ${sat}%, ${light}%, ${0.4 + field.energy * 0.4})`;
  ctx.fill();

  // Motion trail
  const v = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
  if (v > 0.8) {
    ctx.beginPath();
    ctx.moveTo(a.x, a.y);
    ctx.lineTo(a.x - a.vx * 3, a.y - a.vy * 3);
    ctx.strokeStyle = `hsla(${hue}, ${sat}%, ${light}%, ${Math.min(v * 0.08, 0.25)})`;
    ctx.lineWidth = r * 0.5;
    ctx.lineCap = 'round';
    ctx.stroke();
  }
}

// Connection lines when cohesion > 0.4
if (field.cohesion > 0.4) {
  const connAlpha = (field.cohesion - 0.4) * 0.25;
  for (let i = 0; i < agents.length; i += 2) {
    for (let j = i + 1; j < Math.min(i + 6, agents.length); j += 2) {
      const dx = agents[i].x - agents[j].x;
      const dy = agents[i].y - agents[j].y;
      const d = Math.sqrt(dx * dx + dy * dy);
      if (d < 60) {
        ctx.beginPath();
        ctx.moveTo(agents[i].x, agents[i].y);
        ctx.lineTo(agents[j].x, agents[j].y);
        ctx.strokeStyle = `hsla(${hue}, ${sat * 0.5}%, ${light}%, ${connAlpha * (1 - d / 60)})`;
        ctx.lineWidth = 0.4;
        ctx.stroke();
      }
    }
  }
}
```

- [ ] **Step 2: Verify**

With temporary randomize still active, open file. Expected: colors shift between cold blues and warm reds as warmth changes. Brighter and more vivid at high energy, dim and desaturated at low energy. Connection lines appear when cohesion is high.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: color system derives hue/saturation/lightness from field state"
```

---

### Task 5: Aspect System (佛相)

**Files:**
- Modify: `index.html`

Replace the temporary random field changes with the Aspect system — five emotional states that each set specific field target ranges.

- [ ] **Step 1: Add Aspect system between FIELD and AGENTS**

Remove the temporary `setInterval(() => randomizeFieldTarget(), ...)` call.

```javascript
// === ASPECT 佛相 ===
const ASPECTS = {
  wrath: {
    energy: [0.8, 1.0], warmth: [0.7, 1.0], rotation: [0.5, 1.0],
    flowX: [-0.5, 0.5], flowY: [-0.5, 0.5], cohesion: [0.3, 0.6], pulse: [0.6, 1.0],
  },
  compassion: {
    energy: [0.15, 0.35], warmth: [0.55, 0.75], rotation: [-0.2, 0.2],
    flowX: [-0.3, 0.3], flowY: [-0.3, 0.3], cohesion: [0.5, 0.7], pulse: [0.1, 0.3],
  },
  joy: {
    energy: [0.5, 0.8], warmth: [0.4, 0.7], rotation: [-0.4, 0.4],
    flowX: [-0.3, 0.5], flowY: [-0.8, -0.2], cohesion: [0.3, 0.6], pulse: [0.3, 0.7],
  },
  meditation: {
    energy: [0.05, 0.2], warmth: [0.1, 0.3], rotation: [-0.1, 0.1],
    flowX: [-0.05, 0.05], flowY: [-0.05, 0.05], cohesion: [0.7, 0.95], pulse: [0.0, 0.15],
  },
  silence: {
    energy: [0.0, 0.05], warmth: [0.3, 0.5], rotation: [0, 0],
    flowX: [0, 0], flowY: [0, 0], cohesion: [0.4, 0.6], pulse: [0.0, 0.05],
  },
};

let currentAspect = 'compassion';

function randRange(range) {
  return range[0] + Math.random() * (range[1] - range[0]);
}

function applyAspect(name) {
  currentAspect = name;
  const a = ASPECTS[name];
  for (const k of Object.keys(a)) {
    fieldTarget[k] = randRange(a[k]);
  }
}

// Start with a random non-silence aspect
applyAspect(['compassion', 'joy', 'meditation'][Math.floor(Math.random() * 3)]);
```

- [ ] **Step 2: Add temporary aspect cycling for testing**

Temporarily add after the aspect system:

```javascript
// TEMP: cycle through aspects for testing
const aspectNames = Object.keys(ASPECTS);
let aspectIndex = 0;
setInterval(() => {
  aspectIndex = (aspectIndex + 1) % aspectNames.length;
  applyAspect(aspectNames[aspectIndex]);
  console.log('Aspect:', aspectNames[aspectIndex]);
}, 8000);
```

- [ ] **Step 3: Verify**

Open file, open console. Expected: every 8 seconds, aspect changes. Wrath should feel intense (fast, red, saturated). Compassion feels soft and warm. Joy is lively with upward movement. Meditation is slow, centered, cold blue. Silence is nearly still. Console logs the current aspect.

- [ ] **Step 4: Remove temporary cycling, commit**

Remove the temporary `setInterval` aspect cycling code.

```bash
git add index.html
git commit -m "feat: aspect system (佛相) with five emotional states driving field"
```

---

### Task 6: Breath Cycle (呼吸)

**Files:**
- Modify: `index.html`

Add the life cycle state machine: Birth → Living → Dying → Silence → Rebirth. Controls a `breathAlpha` that fades agents in/out.

- [ ] **Step 1: Add Breath system after ASPECT**

```javascript
// === BREATH 呼吸 ===
const BREATH = { BIRTH: 0, LIVING: 1, DYING: 2, SILENCE: 3 };
let breathState = BREATH.BIRTH;
let breathTimer = 0;
let breathAlpha = 0; // global alpha multiplier for all visuals

// Durations are set per-life; these are defaults overridden by beginLife()
let birthDur = 3000;
let livingDur = 8000;
let dyingDur = 3000;
let silenceDur = 2000;

function easeInOut(t) {
  return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
}

function beginLife() {
  // Variable durations — impermanence of time itself
  birthDur = 2000 + Math.random() * 3000;
  livingDur = 3000 + Math.random() * 20000; // 3s to 23s — some brief, some long
  dyingDur = 2000 + Math.random() * 4000;
  silenceDur = 1000 + Math.random() * 5000;
  breathState = BREATH.BIRTH;
  breathTimer = 0;

  // Choose next aspect (karma system will refine this later)
  const names = Object.keys(ASPECTS).filter(n => n !== 'silence');
  applyAspect(names[Math.floor(Math.random() * names.length)]);
}

function updateBreath(dt) {
  breathTimer += dt;

  switch (breathState) {
    case BREATH.BIRTH:
      breathAlpha = easeInOut(Math.min(breathTimer / birthDur, 1));
      if (breathTimer >= birthDur) {
        breathState = BREATH.LIVING;
        breathTimer = 0;
      }
      break;

    case BREATH.LIVING:
      breathAlpha = 1;
      if (breathTimer >= livingDur) {
        breathState = BREATH.DYING;
        breathTimer = 0;
      }
      break;

    case BREATH.DYING:
      breathAlpha = 1 - easeInOut(Math.min(breathTimer / dyingDur, 1));
      // Push field toward silence during dying
      applyAspect('silence');
      if (breathTimer >= dyingDur) {
        breathState = BREATH.SILENCE;
        breathTimer = 0;
      }
      break;

    case BREATH.SILENCE:
      breathAlpha = 0;
      if (breathTimer >= silenceDur) {
        beginLife();
      }
      break;
  }
}

// Start first life
beginLife();
```

- [ ] **Step 2: Integrate breathAlpha into the main loop**

Update the main loop: call `updateBreath(dt)` after `lerpField(dt)`. Multiply all agent rendering alpha values by `breathAlpha`. During SILENCE state, skip agent drawing entirely.

In the agent drawing section, wrap the drawing code:

```javascript
if (breathAlpha > 0.001) {
  updateAgents(t, dt);

  // ... existing agent + connection drawing code ...
  // Replace all alpha values: multiply by breathAlpha
  // e.g. ctx.fillStyle = `hsla(${hue + a.phase * 3}, ${sat}%, ${light}%, ${(0.4 + field.energy * 0.4) * breathAlpha})`;
  // e.g. connection connAlpha * breathAlpha
}
```

- [ ] **Step 3: Verify**

Open file. Expected: agents fade in (birth), live for a while, fade out (dying), go completely dark (silence), then a new life begins with potentially different colors and movement patterns. Each cycle has different durations.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: breath cycle (birth → living → dying → silence → rebirth)"
```

---

### Task 7: Karma System (因緣)

**Files:**
- Modify: `index.html`

Replace random aspect selection in `beginLife()` with karma-driven selection. The dying life's field state becomes residual conditions that weight the next aspect choice.

- [ ] **Step 1: Add Karma system after BREATH**

```javascript
// === KARMA 因緣 ===
let karma = {
  energy: 0.5,
  warmth: 0.5,
  rotation: 0,
  cohesion: 0.5,
};

function harvestKarma() {
  // Snapshot current field as residual conditions
  karma.energy = field.energy;
  karma.warmth = field.warmth;
  karma.rotation = field.rotation;
  karma.cohesion = field.cohesion;
}

function karmaChooseAspect() {
  // Weight each aspect by affinity to karma
  const weights = {
    wrath: 0.3 + karma.energy * 3 + karma.warmth * 1.5,
    compassion: 0.5 + (1 - karma.energy) * 1.5 + karma.warmth * 1,
    joy: 0.5 + karma.energy * 1.5 + (1 - Math.abs(karma.rotation)) * 0.5,
    meditation: 0.3 + (1 - karma.energy) * 2 + karma.cohesion * 2,
    // silence is not chosen here — it's the breath cycle's dying phase
  };

  // Add randomness so it's never fully deterministic
  const entries = Object.entries(weights);
  const jittered = entries.map(([name, w]) => [name, w * (0.3 + Math.random() * 1.4)]);
  const total = jittered.reduce((sum, [, w]) => sum + w, 0);
  let r = Math.random() * total;
  for (const [name, w] of jittered) {
    r -= w;
    if (r <= 0) return name;
  }
  return jittered[0][0];
}
```

- [ ] **Step 2: Wire karma into breath cycle**

In `updateBreath`, at the DYING → SILENCE transition, call `harvestKarma()`.

In `beginLife()`, replace the random aspect selection with:

```javascript
const nextAspect = karmaChooseAspect();
applyAspect(nextAspect);
```

- [ ] **Step 3: Verify**

Open file. Watch several life cycles. Expected: after a high-energy life (wrath-like), the next life is more likely to be wrath or joy (high energy karma). After a calm life (meditation-like), the next is more likely compassion or meditation. But there's always randomness — occasionally a calm life is followed by wrath (karma is tendency, not fate).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: karma system — residual conditions influence next life's aspect"
```

---

### Task 8: Macro Rhythm — Rain and Drought Seasons (劫)

**Files:**
- Modify: `index.html`

Add a slow-drifting abundance parameter that creates longer-scale patterns: rain seasons (rapid succession of lives) and drought seasons (long silences, brief lives).

- [ ] **Step 1: Add Macro system after KARMA**

```javascript
// === MACRO 劫 ===
let abundance = 0.5; // 0 = drought, 1 = rain season
let abundanceTarget = Math.random();
let abundanceShiftTimer = 0;
const ABUNDANCE_SHIFT_INTERVAL = 60000 + Math.random() * 120000; // 1-3 minutes

function updateMacro(dt) {
  abundanceShiftTimer += dt;
  if (abundanceShiftTimer > ABUNDANCE_SHIFT_INTERVAL) {
    abundanceTarget = Math.random();
    abundanceShiftTimer = 0;
  }
  abundance += (abundanceTarget - abundance) * 0.00005 * dt;
}
```

- [ ] **Step 2: Wire abundance into breath timing**

In `beginLife()`, scale durations by abundance:

```javascript
function beginLife() {
  // Abundance scales life: rain = longer lives, shorter silence; drought = opposite
  const lifeScale = 0.3 + abundance * 1.4;  // 0.3x to 1.7x
  const silenceScale = 0.3 + (1 - abundance) * 2; // inverse

  birthDur = (2000 + Math.random() * 3000) * lifeScale;
  livingDur = (3000 + Math.random() * 20000) * lifeScale;
  dyingDur = (2000 + Math.random() * 4000) * lifeScale;
  silenceDur = (1000 + Math.random() * 5000) * silenceScale;

  breathState = BREATH.BIRTH;
  breathTimer = 0;

  const nextAspect = karmaChooseAspect();
  applyAspect(nextAspect);
}
```

- [ ] **Step 3: Scale agent count by abundance**

In `updateAgents`, skip some agents during drought to make visuals sparser:

```javascript
const activeCount = Math.floor(NUM_AGENTS * (0.3 + abundance * 0.7));
```

Only update and draw the first `activeCount` agents.

- [ ] **Step 4: Call updateMacro(dt) in the main loop, verify**

Open file and watch for 2+ minutes. Expected: some periods feel busier (shorter silences, longer lives, more agents), others feel sparse (long silences, brief dim lives, fewer agents). The shift is gradual — you notice in retrospect.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: macro rhythm (劫) — rain/drought seasons scale life durations and agent density"
```

---

### Task 9: Traces Layer (痕跡)

**Files:**
- Modify: `index.html`

When a life dies, stamp its final frame onto the trace canvas at low opacity. The trace canvas slowly fades, so ghosts of past lives linger for ~2-3 generations.

- [ ] **Step 1: Add Traces system before MAIN LOOP**

```javascript
// === TRACES 痕跡 ===
function stampTrace() {
  // Burn current main canvas onto trace at low opacity
  traceCtx.globalAlpha = 0.15;
  traceCtx.drawImage(mainCanvas, 0, 0);
  traceCtx.globalAlpha = 1.0;
}

function fadeTrace() {
  // Very slow fade — clears over ~3 life cycles
  traceCtx.fillStyle = 'rgba(0, 0, 0, 0.003)';
  traceCtx.fillRect(0, 0, W, H);
}
```

- [ ] **Step 2: Wire into breath cycle**

In `updateBreath`, at the transition from DYING to SILENCE (when `breathTimer >= dyingDur`), call `stampTrace()` before changing state.

Call `fadeTrace()` every frame in the main loop (before agent drawing).

- [ ] **Step 3: Verify**

Open file. Expected: when a life dies, a faint ghost of its final form remains visible on screen. The next life dances on top of this ghost. After 2-3 more lives, the original ghost has faded completely. The layering creates a sense of accumulated memory.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: traces layer — dying lives leave fading ghosts"
```

---

### Task 10: True Emptiness (真空)

**Files:**
- Modify: `index.html`

Some silence phases are true emptiness — completely black, no trace rendering, no buddha-nature (added later). Rare and powerful.

- [ ] **Step 1: Add true emptiness flag to breath system**

```javascript
let isTrueEmptiness = false;

// In the DYING → SILENCE transition in updateBreath, after stampTrace():
isTrueEmptiness = Math.random() < 0.15; // ~15% of silences are true emptiness
if (isTrueEmptiness) {
  silenceDur = 15000 + Math.random() * 90000; // 15s to 105s — uncomfortable
  // Clear trace canvas entirely — true emptiness erases memory
  traceCtx.clearRect(0, 0, W, H);
}
```

- [ ] **Step 2: Skip all rendering during true emptiness**

In the main loop, when `breathState === BREATH.SILENCE && isTrueEmptiness`, skip `fadeTrace()` and render nothing at all — not even clearing the main canvas (it's already black from the dying phase fading to zero).

```javascript
if (breathState === BREATH.SILENCE && isTrueEmptiness) {
  // Absolute nothing. Pure black.
  ctx.clearRect(0, 0, W, H);
  updateBreath(dt);
  updateMacro(dt);
  requestAnimationFrame(loop);
  return;
}
```

- [ ] **Step 3: Verify**

This is hard to test due to 15% chance. Temporarily set probability to 1.0 and silenceDur to `3000 + Math.random() * 5000`. Open file. Expected: after each life, screen goes completely black. No trace ghosts, no subtle elements. True void. Restore probability to 0.15 and original duration after testing.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: true emptiness — rare silence phases with absolute void"
```

---

### Task 11: Buddha-nature (佛性)

**Files:**
- Modify: `index.html`

A nearly invisible single-pixel light that persists through all states — except during true emptiness, and very rarely disappears even outside of it.

- [ ] **Step 1: Add Buddha-nature system before MAIN LOOP**

```javascript
// === BUDDHA-NATURE 佛性 ===
let buddhaNatureVisible = true;
let buddhaNatureX = 0;
let buddhaNatureY = 0;
let buddhaNatureTimer = 0;
const BUDDHA_NATURE_VANISH_CHECK = 30000; // check every 30s

function initBuddhaNature() {
  // Position: somewhere in the frame, not center (too obvious)
  buddhaNatureX = W * (0.15 + Math.random() * 0.7);
  buddhaNatureY = H * (0.15 + Math.random() * 0.7);
}
initBuddhaNature();
addEventListener('resize', () => {
  buddhaNatureX = Math.min(buddhaNatureX, W - 10);
  buddhaNatureY = Math.min(buddhaNatureY, H - 10);
});

function updateBuddhaNature(dt) {
  buddhaNatureTimer += dt;
  if (buddhaNatureTimer > BUDDHA_NATURE_VANISH_CHECK) {
    buddhaNatureTimer = 0;
    // ~5% chance to vanish for one cycle, then reappear
    if (buddhaNatureVisible && Math.random() < 0.05) {
      buddhaNatureVisible = false;
    } else {
      buddhaNatureVisible = true;
    }
  }
}

function drawBuddhaNature() {
  if (!buddhaNatureVisible) return;
  if (isTrueEmptiness && breathState === BREATH.SILENCE) return;

  // Extremely subtle — 1px, very low alpha, slow breathing
  const t = performance.now();
  const alpha = 0.06 + Math.sin(t * 0.001) * 0.03; // 0.03 to 0.09
  ctx.fillStyle = `rgba(200, 200, 210, ${alpha})`;
  ctx.fillRect(buddhaNatureX, buddhaNatureY, 1, 1);
}
```

- [ ] **Step 2: Call in main loop**

Add `updateBuddhaNature(dt)` and `drawBuddhaNature()` in the main loop — draw it last, on top of everything.

- [ ] **Step 3: Verify**

Open file and look very carefully. The pixel is nearly invisible — you may need to look for 10-20 seconds to spot it. It should persist through births and deaths. During true emptiness it disappears. This is correct and intentional — it rewards patient observation.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: buddha-nature — persistent near-invisible witness pixel"
```

---

### Task 12: Thangka — Aureole (背光)

**Files:**
- Modify: `index.html`

When field energy crosses a high threshold, agents briefly form a radiating symmetrical pattern from a focal point, then it shatters.

- [ ] **Step 1: Add Thangka section with aureole logic**

```javascript
// === THANGKA 唐卡 ===
let aureoleActive = false;
let aureoleTimer = 0;
let aureoleCx = 0, aureoleCy = 0;
const AUREOLE_DURATION = 4000;
const AUREOLE_ENERGY_THRESHOLD = 0.85;
let aureoleCooldown = 0;

function checkAureole(dt) {
  aureoleCooldown -= dt;
  if (!aureoleActive && field.energy > AUREOLE_ENERGY_THRESHOLD && aureoleCooldown <= 0) {
    aureoleActive = true;
    aureoleTimer = 0;
    // Focal point: center of agent mass
    let sumX = 0, sumY = 0;
    for (const a of agents) { sumX += a.x; sumY += a.y; }
    aureoleCx = sumX / agents.length;
    aureoleCy = sumY / agents.length;
    aureoleCooldown = 20000 + Math.random() * 30000; // don't repeat too soon
  }
  if (aureoleActive) {
    aureoleTimer += dt;
    if (aureoleTimer > AUREOLE_DURATION) {
      aureoleActive = false;
    }
  }
}

function applyAureoleForce() {
  if (!aureoleActive) return;
  const progress = aureoleTimer / AUREOLE_DURATION;
  // First half: pull into radial symmetry. Second half: release (shatter).
  if (progress > 0.5) return;

  const strength = easeInOut(progress * 2) * 0.15; // peak at progress=0.5
  for (const a of agents) {
    const dx = a.x - aureoleCx, dy = a.y - aureoleCy;
    const dist = Math.sqrt(dx * dx + dy * dy) + 1;
    const angle = Math.atan2(dy, dx);
    // Push outward along radial lines (create spoke pattern)
    const targetDist = 50 + (Math.abs(Math.sin(angle * 6)) * 150); // 6-fold symmetry
    const forceMag = (targetDist - dist) * strength * 0.01;
    a.vx += Math.cos(angle) * forceMag;
    a.vy += Math.sin(angle) * forceMag;
  }
}

function drawAureoleGlow() {
  if (!aureoleActive) return;
  const progress = aureoleTimer / AUREOLE_DURATION;
  const alpha = Math.sin(progress * Math.PI) * 0.08 * breathAlpha; // fade in and out
  if (alpha < 0.001) return;

  const hue = fieldToHue();
  const grad = ctx.createRadialGradient(aureoleCx, aureoleCy, 0, aureoleCx, aureoleCy, 200);
  grad.addColorStop(0, `hsla(${hue}, 40%, 60%, ${alpha})`);
  grad.addColorStop(1, `hsla(${hue}, 40%, 60%, 0)`);
  ctx.fillStyle = grad;
  ctx.beginPath();
  ctx.arc(aureoleCx, aureoleCy, 200, 0, Math.PI * 2);
  ctx.fill();
}
```

- [ ] **Step 2: Wire into main loop**

Call `checkAureole(dt)` after field update. Call `applyAureoleForce()` inside `updateAgents` at the end (after all other forces, before applying velocity). Call `drawAureoleGlow()` after agent drawing.

- [ ] **Step 3: Verify**

Temporarily set `AUREOLE_ENERGY_THRESHOLD = 0.3` and `aureoleCooldown = 5000` for easier triggering. Open file. Expected: when energy is moderate-high, agents briefly arrange into a radial spoke pattern with a subtle center glow, then scatter. Restore thresholds after testing.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: thangka aureole — radiating symmetry at energy peaks"
```

---

### Task 13: Thangka — Mandala (曼陀羅)

**Files:**
- Modify: `index.html`

During meditation aspect with very high cohesion, agents briefly arrange into concentric symmetric rings, then dissolve.

- [ ] **Step 1: Add mandala logic in the THANGKA section**

```javascript
let mandalaActive = false;
let mandalaTimer = 0;
const MANDALA_DURATION = 6000;
let mandalaCooldown = 0;

function checkMandala(dt) {
  mandalaCooldown -= dt;
  if (!mandalaActive && currentAspect === 'meditation' && field.cohesion > 0.8 && field.energy < 0.2 && mandalaCooldown <= 0) {
    mandalaActive = true;
    mandalaTimer = 0;
    mandalaCooldown = 30000 + Math.random() * 60000;
  }
  if (mandalaActive) {
    mandalaTimer += dt;
    if (mandalaTimer > MANDALA_DURATION || currentAspect !== 'meditation') {
      mandalaActive = false;
    }
  }
}

function applyMandalaForce() {
  if (!mandalaActive) return;
  const progress = mandalaTimer / MANDALA_DURATION;
  // Strength: build up in first 40%, hold, dissolve in last 30%
  let strength;
  if (progress < 0.4) strength = easeInOut(progress / 0.4) * 0.2;
  else if (progress < 0.7) strength = 0.2;
  else strength = 0.2 * (1 - easeInOut((progress - 0.7) / 0.3));

  const cx = W / 2, cy = H / 2;
  const foldSymmetry = 8; // 8-fold symmetry like a mandala

  for (let i = 0; i < agents.length; i++) {
    const a = agents[i];
    const dx = a.x - cx, dy = a.y - cy;
    const dist = Math.sqrt(dx * dx + dy * dy) + 1;
    const angle = Math.atan2(dy, dx);

    // Snap angle to nearest symmetry spoke
    const sectorAngle = (Math.PI * 2) / foldSymmetry;
    const snappedAngle = Math.round(angle / sectorAngle) * sectorAngle;

    // Target: concentric ring based on agent index
    const ringIndex = Math.floor(i / (agents.length / 5)); // 5 rings
    const targetDist = 40 + ringIndex * 50;

    const targetX = cx + Math.cos(snappedAngle) * targetDist;
    const targetY = cy + Math.sin(snappedAngle) * targetDist;

    a.vx += (targetX - a.x) * strength * 0.01;
    a.vy += (targetY - a.y) * strength * 0.01;
  }
}
```

- [ ] **Step 2: Wire into main loop**

Call `checkMandala(dt)` after `checkAureole(dt)`. Call `applyMandalaForce()` inside `updateAgents` alongside `applyAureoleForce()`.

- [ ] **Step 3: Verify**

Temporarily force meditation aspect and set cohesion threshold to 0.5 and energy threshold to 0.5. Open file. Expected: agents slowly arrange into concentric rings with 8-fold symmetry, hold briefly, then dissolve back into freeform movement. Restore thresholds.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: thangka mandala — symmetric rings during deep meditation"
```

---

### Task 14: Thangka — Flame Motif (火焰紋)

**Files:**
- Modify: `index.html`

During wrath aspect, agent trails become stylized curling flame shapes — upward-biased, forking, with calligraphic energy.

- [ ] **Step 1: Add flame trail rendering in THANGKA section**

```javascript
function drawFlameTrails() {
  if (currentAspect !== 'wrath' || field.energy < 0.7) return;

  const intensity = (field.energy - 0.7) / 0.3; // 0 to 1
  const hue = fieldToHue();

  for (let i = 0; i < agents.length; i += 2) { // every other agent for perf
    const a = agents[i];
    const v = Math.sqrt(a.vx * a.vx + a.vy * a.vy);
    if (v < 1.5) continue;

    const len = v * 6 * intensity;
    const baseAngle = Math.atan2(a.vy, a.vx) + Math.PI; // trail behind

    ctx.save();
    ctx.translate(a.x, a.y);

    // Main flame stroke
    ctx.beginPath();
    ctx.moveTo(0, 0);
    const segments = 6;
    let cx1 = 0, cy1 = 0;
    for (let s = 1; s <= segments; s++) {
      const t = s / segments;
      const curl = Math.sin(t * Math.PI * 2 + performance.now() * 0.005 + a.phase) * 15 * t;
      const upBias = -t * 20 * intensity; // flames rise
      const px = Math.cos(baseAngle) * len * t + curl;
      const py = Math.sin(baseAngle) * len * t + upBias;
      ctx.lineTo(px, py);
    }
    ctx.strokeStyle = `hsla(${hue + 10}, ${50 + intensity * 30}%, ${50 + intensity * 15}%, ${intensity * breathAlpha * 0.4})`;
    ctx.lineWidth = 1.5 + intensity;
    ctx.lineCap = 'round';
    ctx.stroke();

    // Fork: smaller tendril splitting off
    if (v > 2.5) {
      ctx.beginPath();
      const forkStart = 0.4;
      const forkAngle = baseAngle + (Math.random() > 0.5 ? 0.5 : -0.5);
      const fx = Math.cos(baseAngle) * len * forkStart;
      const fy = Math.sin(baseAngle) * len * forkStart;
      ctx.moveTo(fx, fy);
      for (let s = 1; s <= 3; s++) {
        const t = s / 3;
        const px = fx + Math.cos(forkAngle) * len * 0.4 * t;
        const py = fy + Math.sin(forkAngle) * len * 0.4 * t - t * 12;
        ctx.lineTo(px, py);
      }
      ctx.strokeStyle = `hsla(${hue + 20}, ${40 + intensity * 20}%, ${55 + intensity * 10}%, ${intensity * breathAlpha * 0.25})`;
      ctx.lineWidth = 1;
      ctx.stroke();
    }

    ctx.restore();
  }
}
```

- [ ] **Step 2: Wire into main loop**

Call `drawFlameTrails()` after agent drawing, before aureole glow.

- [ ] **Step 3: Verify**

Temporarily force wrath aspect. Open file. Expected: agents leave curling, upward-biased flame trails. Trails should feel calligraphic and rhythmic, not like realistic fire. The effect intensifies with energy. Restore normal aspect flow.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: thangka flame motif — stylized curling trails during wrath"
```

---

### Task 15: Thangka — Gold Event (金)

**Files:**
- Modify: `index.html`

Extremely rare moments where conditions align and agent colors shift briefly to gold. An event, not a style.

- [ ] **Step 1: Add gold event logic in THANGKA section**

```javascript
let goldActive = false;
let goldTimer = 0;
const GOLD_DURATION = 3000;
let goldCooldown = 60000; // can't happen in first minute
let timeSinceStart = 0;

function checkGold(dt) {
  timeSinceStart += dt;
  goldCooldown -= dt;

  if (!goldActive && goldCooldown <= 0) {
    // Requires: specific karma alignment — moderate energy, high warmth, high cohesion
    const alignment =
      (1 - Math.abs(field.energy - 0.6)) *   // energy near 0.6
      field.warmth *                           // high warmth
      field.cohesion *                         // high cohesion
      (1 - Math.abs(field.rotation));          // low rotation
    // Threshold: alignment must be > 0.3, plus luck
    if (alignment > 0.3 && Math.random() < 0.002) { // ~0.2% per frame when aligned
      goldActive = true;
      goldTimer = 0;
      goldCooldown = 180000 + Math.random() * 180000; // 3-6 minutes between golds
    }
  }

  if (goldActive) {
    goldTimer += dt;
    if (goldTimer > GOLD_DURATION) {
      goldActive = false;
    }
  }
}

function goldAlpha() {
  if (!goldActive) return 0;
  const progress = goldTimer / GOLD_DURATION;
  return Math.sin(progress * Math.PI); // fade in, fade out
}
```

- [ ] **Step 2: Blend gold into color system**

In the agent drawing section, when `goldActive`, blend the agent color toward gold:

```javascript
// Inside agent drawing loop, after computing hue/sat/light:
const gAlpha = goldAlpha();
let drawHue = hue + a.phase * 3;
let drawSat = sat;
let drawLight = light;
if (gAlpha > 0) {
  // Gold: hue 45, sat 70, light 60
  drawHue = drawHue + (45 - drawHue) * gAlpha;
  drawSat = drawSat + (70 - drawSat) * gAlpha;
  drawLight = drawLight + (60 - drawLight) * gAlpha;
}
// Use drawHue, drawSat, drawLight in fillStyle
```

- [ ] **Step 3: Verify**

Temporarily set gold probability to 0.5 and cooldown to 5000. Open file. Expected: periodically, all agents shift to warm gold tones for ~3 seconds, then fade back. The gold should feel like a momentary transcendence. Restore original probability and cooldown.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: thangka gold — rare transcendent color event"
```

---

### Task 16: Remove Debug Overlay + Final Polish

**Files:**
- Modify: `index.html`

Remove the debug overlay, tune parameters, ensure long-running stability.

- [ ] **Step 1: Remove debug overlay**

Delete the temporary debug text rendering code (the `ctx.fillText` loop showing field values).

- [ ] **Step 2: Add memory leak prevention for long runs**

Ensure no arrays grow unboundedly. The trace canvas fade handles its own cleanup. Add a check that `performance.now()` is used (not `Date.now()`) for animation timing to avoid drift.

- [ ] **Step 3: Add fullscreen on click (optional, unobtrusive)**

```javascript
// Silent fullscreen toggle — no UI, just click anywhere
document.addEventListener('click', () => {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen().catch(() => {});
  }
});
```

- [ ] **Step 4: Final visual tuning pass**

Open the file and watch for 5+ minutes. Tune these values by feel:
- `lerpField` speed (0.0005) — how fast field responds to aspect changes
- Agent count (200) — balance between visual richness and performance
- Breath durations — do births feel too sudden? Do silences drag?
- Connection line threshold and alpha — too prominent? Too subtle?
- Trail length — do motion trails enhance or clutter?
- Trace stamp opacity (0.15) and fade rate (0.003) — are ghosts visible enough? Too visible?

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "polish: remove debug overlay, add fullscreen, tune visual parameters"
```

---

### Task 17: Cleanup Demo Files

**Files:**
- Delete: `demo-a-dissolve.html`, `demo-b-breath.html`, `demo-c-coexist.html`, `demo-color-a-ink.html`, `demo-color-b-cosmos.html`, `demo-color-c-seasons.html`, `demo-field-agents.html`, `demo-karma-engine.html`

- [ ] **Step 1: Remove all demo files**

```bash
git rm demo-*.html
git commit -m "chore: remove brainstorming demo files"
```
