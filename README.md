

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>🐍 Snake</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@700;900&display=swap');

  :root {
    --bg:        #090d0f;
    --panel:     #0e1417;
    --border:    #1a2a22;
    --green1:    #00ff88;
    --green2:    #00cc66;
    --green3:    #007744;
    --food:      #ff3c5a;
    --food2:     #ff7090;
    --text:      #c8ffd4;
    --muted:     #3a6650;
    --score-col: #00ff88;
    --glow:      0 0 12px #00ff8866;
    --glow-lg:   0 0 40px #00ff8833, 0 0 80px #00cc6611;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Share Tech Mono', monospace;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    user-select: none;
  }

  /* Scanline overlay */
  body::before {
    content: '';
    position: fixed; inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,0,0,.15) 2px,
      rgba(0,0,0,.15) 4px
    );
    pointer-events: none;
    z-index: 999;
  }

  /* Ambient glow blobs */
  body::after {
    content: '';
    position: fixed; inset: 0;
    background:
      radial-gradient(ellipse 60% 40% at 20% 80%, #00ff8808 0%, transparent 70%),
      radial-gradient(ellipse 50% 30% at 80% 20%, #00cc6608 0%, transparent 70%);
    pointer-events: none;
    z-index: 0;
  }

  .wrapper {
    position: relative; z-index: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 16px;
  }

  /* ── Header ── */
  header {
    text-align: center;
    letter-spacing: .15em;
  }
  header h1 {
    font-family: 'Orbitron', sans-serif;
    font-size: clamp(1.4rem, 4vw, 2rem);
    font-weight: 900;
    color: var(--green1);
    text-shadow: var(--glow);
    text-transform: uppercase;
  }
  header p {
    font-size: .7rem;
    color: var(--muted);
    margin-top: 4px;
    letter-spacing: .3em;
    text-transform: uppercase;
  }

  /* ── HUD bar ── */
  .hud {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 602px;
    padding: 6px 14px;
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 6px 6px 0 0;
    font-size: .8rem;
  }
  .hud-item { display: flex; flex-direction: column; gap: 1px; }
  .hud-label { color: var(--muted); font-size: .6rem; letter-spacing: .2em; text-transform: uppercase; }
  .hud-value {
    font-family: 'Orbitron', sans-serif;
    font-size: 1rem;
    color: var(--score-col);
    text-shadow: var(--glow);
    transition: color .2s;
  }
  .hud-value.bump {
    animation: bump .25s ease;
  }
  @keyframes bump {
    0%   { transform: scale(1);   color: #fff; }
    50%  { transform: scale(1.3); color: #fff; }
    100% { transform: scale(1);   color: var(--score-col); }
  }

  /* ── Canvas shell ── */
  .canvas-shell {
    position: relative;
    border: 1px solid var(--border);
    border-top: none;
    border-radius: 0 0 8px 8px;
    overflow: hidden;
    box-shadow: var(--glow-lg);
  }

  canvas {
    display: block;
    background: var(--panel);
    image-rendering: pixelated;
  }

  /* ── Overlay (game-over / start) ── */
  .overlay {
    position: absolute; inset: 0;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    gap: 12px;
    background: rgba(9,13,15,.88);
    backdrop-filter: blur(3px);
    opacity: 0;
    pointer-events: none;
    transition: opacity .3s;
  }
  .overlay.show { opacity: 1; pointer-events: all; }

  .overlay-title {
    font-family: 'Orbitron', sans-serif;
    font-size: clamp(1.8rem, 5vw, 2.6rem);
    font-weight: 900;
    letter-spacing: .1em;
    text-shadow: 0 0 24px currentColor;
  }
  .overlay-title.red  { color: #ff3c5a; }
  .overlay-title.green{ color: var(--green1); }

  .overlay-score {
    font-size: 1rem;
    color: var(--text);
    letter-spacing: .15em;
  }
  .overlay-score span {
    color: var(--green1);
    font-size: 1.4rem;
  }

  .overlay-hint {
    font-size: .72rem;
    color: var(--muted);
    letter-spacing: .2em;
    text-transform: uppercase;
    margin-top: 8px;
  }

  .btn-row { display: flex; gap: 12px; margin-top: 4px; }
  .btn {
    font-family: 'Share Tech Mono', monospace;
    font-size: .8rem;
    letter-spacing: .2em;
    text-transform: uppercase;
    padding: 10px 28px;
    border-radius: 4px;
    border: 1.5px solid;
    cursor: pointer;
    transition: background .15s, box-shadow .15s, transform .1s;
    background: transparent;
  }
  .btn:active { transform: scale(.96); }
  .btn-play {
    color: var(--green1);
    border-color: var(--green1);
  }
  .btn-play:hover { background: var(--green3); box-shadow: var(--glow); }
  .btn-quit {
    color: #ff3c5a;
    border-color: #ff3c5a;
  }
  .btn-quit:hover { background: #3a0010; box-shadow: 0 0 12px #ff3c5a66; }

  /* ── Footer hint ── */
  .keys {
    font-size: .62rem;
    color: var(--muted);
    letter-spacing: .2em;
    text-transform: uppercase;
    text-align: center;
  }
  .keys kbd {
    display: inline-block;
    padding: 2px 6px;
    border: 1px solid var(--muted);
    border-radius: 3px;
    font-family: inherit;
    font-size: .6rem;
    margin: 0 2px;
  }

  /* ── Speed pip bar ── */
  .pips { display: flex; gap: 4px; align-items: center; }
  .pip {
    width: 8px; height: 8px;
    border-radius: 50%;
    background: var(--border);
    transition: background .3s, box-shadow .3s;
  }
  .pip.lit {
    background: var(--green1);
    box-shadow: 0 0 6px var(--green1);
  }

  /* Mobile d-pad */
  .dpad {
    display: none;
    grid-template-columns: repeat(3, 44px);
    grid-template-rows: repeat(3, 44px);
    gap: 4px;
    margin-top: 4px;
  }
  @media (max-width: 650px) { .dpad { display: grid; } }
  .dpad-btn {
    background: var(--panel);
    border: 1px solid var(--border);
    border-radius: 6px;
    color: var(--green1);
    font-size: 1.2rem;
    display: flex; align-items: center; justify-content: center;
    cursor: pointer;
    transition: background .1s;
    -webkit-tap-highlight-color: transparent;
  }
  .dpad-btn:active { background: var(--green3); }
  .dpad-center { opacity: 0; pointer-events: none; }

</style>
</head>
<body>

<div class="wrapper">

  <header>
    <h1>⬛ Snake</h1>
    <p>Retro Terminal Edition</p>
  </header>

  <!-- HUD -->
  <div class="hud">
    <div class="hud-item">
      <span class="hud-label">Score</span>
      <span class="hud-value" id="scoreDisplay">0000</span>
    </div>
    <div class="hud-item" style="align-items:center">
      <span class="hud-label">Speed</span>
      <div class="pips" id="pips"></div>
    </div>
    <div class="hud-item" style="align-items:flex-end">
      <span class="hud-label">Best</span>
      <span class="hud-value" id="bestDisplay">0000</span>
    </div>
  </div>

  <!-- Canvas -->
  <div class="canvas-shell">
    <canvas id="gameCanvas" width="600" height="400"></canvas>

    <!-- Overlay -->
    <div class="overlay show" id="overlay">
      <div class="overlay-title green" id="overlayTitle">SNAKE</div>
      <div class="overlay-score" id="overlayScore" style="display:none">
        Score <span id="finalScore">0</span>
      </div>
      <div class="btn-row">
        <button class="btn btn-play" id="startBtn">▶ Play</button>
        <button class="btn btn-quit" id="quitBtn" style="display:none">✕ Quit</button>
      </div>
      <div class="overlay-hint" id="overlayHint">Arrow keys or D-pad to move</div>
    </div>
  </div>

  <!-- Mobile D-Pad -->
  <div class="dpad" id="dpad">
    <div></div>
    <button class="dpad-btn" id="dUp">▲</button>
    <div></div>
    <button class="dpad-btn" id="dLeft">◀</button>
    <div class="dpad-center"></div>
    <button class="dpad-btn" id="dRight">▶</button>
    <div></div>
    <button class="dpad-btn" id="dDown">▼</button>
    <div></div>
  </div>

  <div class="keys">
    <kbd>↑</kbd><kbd>↓</kbd><kbd>←</kbd><kbd>→</kbd> Move &nbsp;|&nbsp;
    <kbd>R</kbd> Restart &nbsp;|&nbsp; <kbd>Q</kbd> Quit
  </div>

</div>

<script>
// ─────────────────────────────────────────────
//  CONFIG
// ─────────────────────────────────────────────
const CELL    = 20;
const COLS    = 30;   // 600 / 20
const ROWS    = 20;   // 400 / 20
const BASE_MS = 130;  // starting tick interval (ms) — lower = faster
const MIN_MS  = 55;   // fastest allowed
const SPEED_MILESTONES = [50, 100, 200, 350, 500]; // score thresholds

const DIR = {
  UP:    { x:  0, y: -1 },
  DOWN:  { x:  0, y:  1 },
  LEFT:  { x: -1, y:  0 },
  RIGHT: { x:  1, y:  0 },
};
const OPPOSITE = { UP:'DOWN', DOWN:'UP', LEFT:'RIGHT', RIGHT:'LEFT' };

// Colour tokens
const C = {
  bg:        '#090d0f',
  grid:      '#111a14',
  head:      '#00ff88',
  body:      '#00aa55',
  bodyDark:  '#007733',
  eye:       '#001a0a',
  food:      '#ff3c5a',
  foodGlow:  '#ff3c5a',
  particle:  ['#ff3c5a','#ff7090','#ffaa00','#ffffff'],
};

// ─────────────────────────────────────────────
//  CANVAS SETUP
// ─────────────────────────────────────────────
const canvas = document.getElementById('gameCanvas');
const ctx    = canvas.getContext('2d');

// ─────────────────────────────────────────────
//  GAME STATE
// ─────────────────────────────────────────────
let state, tickTimer, animFrame, particles, bestScore;

function freshState() {
  const sc = Math.floor(COLS / 2), sr = Math.floor(ROWS / 2);
  return {
    body:    [{x: sc, y: sr}, {x: sc-1, y: sr}, {x: sc-2, y: sr}],
    dir:     'RIGHT',
    nextDir: 'RIGHT',
    food:    spawnFood([{x:sc,y:sr},{x:sc-1,y:sr},{x:sc-2,y:sr}]),
    score:   0,
    alive:   true,
    tick:    0,
  };
}

function spawnFood(body) {
  let p;
  do { p = {x: Math.floor(Math.random()*COLS), y: Math.floor(Math.random()*ROWS)}; }
  while (body.some(s => s.x===p.x && s.y===p.y));
  return p;
}

// ─────────────────────────────────────────────
//  PARTICLES
// ─────────────────────────────────────────────
function spawnParticles(cx, cy) {
  const count = 14;
  for (let i = 0; i < count; i++) {
    const angle = (Math.PI * 2 * i) / count + Math.random()*.4;
    const speed = 1.5 + Math.random() * 3;
    particles.push({
      x: cx, y: cy,
      vx: Math.cos(angle) * speed,
      vy: Math.sin(angle) * speed,
      life: 1,
      decay: .035 + Math.random() * .03,
      r: 2 + Math.random() * 3,
      color: C.particle[Math.floor(Math.random()*C.particle.length)],
    });
  }
}

function updateParticles() {
  particles = particles.filter(p => p.life > 0);
  for (const p of particles) {
    p.x += p.vx; p.y += p.vy;
    p.vy += .12; // gravity
    p.life -= p.decay;
    p.r *= .97;
  }
}

function drawParticles() {
  for (const p of particles) {
    ctx.save();
    ctx.globalAlpha = p.life * p.life;
    ctx.fillStyle = p.color;
    ctx.shadowColor = p.color;
    ctx.shadowBlur = 8;
    ctx.beginPath();
    ctx.arc(p.x, p.y, p.r, 0, Math.PI*2);
    ctx.fill();
    ctx.restore();
  }
}

// ─────────────────────────────────────────────
//  DRAWING
// ─────────────────────────────────────────────
function drawGrid() {
  ctx.strokeStyle = C.grid;
  ctx.lineWidth = .5;
  for (let x = 0; x <= canvas.width; x += CELL) {
    ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x, canvas.height); ctx.stroke();
  }
  for (let y = 0; y <= canvas.height; y += CELL) {
    ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(canvas.width, y); ctx.stroke();
  }
}

function drawSnake(body) {
  const len = body.length;
  for (let i = len - 1; i >= 0; i--) {
    const s = body[i];
    const t = 1 - i / len; // 1 at head, 0 at tail

    // Gradient green body
    const g = Math.round(0x33 + (0xaa - 0x33) * t).toString(16).padStart(2,'0');
    ctx.fillStyle = `#00${g}44`;

    const px = s.x * CELL + 1;
    const py = s.y * CELL + 1;
    const sz = CELL - 2;

    if (i === 0) {
      // Head — bright with glow
      ctx.shadowColor = C.head;
      ctx.shadowBlur  = 14;
      ctx.fillStyle   = C.head;
      roundRect(ctx, px, py, sz, sz, 5);
      ctx.fill();
      ctx.shadowBlur = 0;

      // Eyes
      drawEyes(s);
    } else {
      ctx.shadowBlur = 0;
      roundRect(ctx, px + 1, py + 1, sz - 2, sz - 2, 4);
      ctx.fill();
    }
  }
}

function drawEyes(head) {
  const px = head.x * CELL;
  const py = head.y * CELL;
  const dir = state.dir;
  const offsets = {
    RIGHT: [{x:14,y:5}, {x:14,y:13}],
    LEFT:  [{x:4, y:5}, {x:4, y:13}],
    UP:    [{x:5, y:5}, {x:13,y:5}],
    DOWN:  [{x:5, y:14},{x:13,y:14}],
  };
  for (const o of (offsets[dir] || offsets.RIGHT)) {
    ctx.fillStyle = C.eye;
    ctx.beginPath();
    ctx.arc(px + o.x, py + o.y, 2.2, 0, Math.PI*2);
    ctx.fill();
    ctx.fillStyle = '#ffffff';
    ctx.beginPath();
    ctx.arc(px + o.x - .5, py + o.y - .5, .8, 0, Math.PI*2);
    ctx.fill();
  }
}

// Food pulse animation
let foodPulse = 0;
function drawFood(food) {
  foodPulse += .08;
  const scale = 1 + Math.sin(foodPulse) * .12;
  const cx = food.x * CELL + CELL / 2;
  const cy = food.y * CELL + CELL / 2;
  const r  = (CELL / 2 - 2) * scale;

  ctx.save();
  ctx.shadowColor = C.foodGlow;
  ctx.shadowBlur  = 16 + Math.sin(foodPulse) * 6;

  const grad = ctx.createRadialGradient(cx - 2, cy - 2, 1, cx, cy, r);
  grad.addColorStop(0, '#ff8099');
  grad.addColorStop(1, C.food);
  ctx.fillStyle = grad;

  ctx.beginPath();
  ctx.arc(cx, cy, r, 0, Math.PI*2);
  ctx.fill();
  ctx.restore();
}

function roundRect(ctx, x, y, w, h, r) {
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.lineTo(x + w - r, y);
  ctx.quadraticCurveTo(x + w, y, x + w, y + r);
  ctx.lineTo(x + w, y + h - r);
  ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
  ctx.lineTo(x + r, y + h);
  ctx.quadraticCurveTo(x, y + h, x, y + h - r);
  ctx.lineTo(x, y + r);
  ctx.quadraticCurveTo(x, y, x + r, y);
  ctx.closePath();
}

// ─────────────────────────────────────────────
//  RENDER LOOP  (animation only — no game logic)
// ─────────────────────────────────────────────
function renderLoop() {
  animFrame = requestAnimationFrame(renderLoop);
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = C.bg;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  drawGrid();

  if (!state) return;

  drawFood(state.food);
  drawSnake(state.body);
  updateParticles();
  drawParticles();
}

// ─────────────────────────────────────────────
//  GAME TICK (logic)
// ─────────────────────────────────────────────
function tick() {
  if (!state || !state.alive) return;

  state.dir = state.nextDir;
  const head  = state.body[0];
  const d     = DIR[state.dir];
  const next  = {x: head.x + d.x, y: head.y + d.y};

  // Wall collision
  if (next.x < 0 || next.x >= COLS || next.y < 0 || next.y >= ROWS) {
    return gameOver();
  }
  // Self collision
  if (state.body.some(s => s.x === next.x && s.y === next.y)) {
    return gameOver();
  }

  state.body.unshift(next);

  if (next.x === state.food.x && next.y === state.food.y) {
    // Ate food
    state.score += 10;
    updateHUD();
    spawnParticles(
      state.food.x * CELL + CELL/2,
      state.food.y * CELL + CELL/2
    );
    state.food = spawnFood(state.body);
    scheduleNextTick(); // recalculate speed
  } else {
    state.body.pop();
  }

  state.tick++;
}

function gameOver() {
  state.alive = false;
  clearTimeout(tickTimer);

  // Save best
  bestScore = Math.max(bestScore || 0, state.score);
  localStorage.setItem('snakeBest', bestScore);
  document.getElementById('bestDisplay').textContent = String(bestScore).padStart(4,'0');

  // Show overlay
  const ov = document.getElementById('overlay');
  document.getElementById('overlayTitle').textContent  = 'GAME OVER';
  document.getElementById('overlayTitle').className    = 'overlay-title red';
  document.getElementById('overlayScore').style.display = '';
  document.getElementById('finalScore').textContent    = state.score;
  document.getElementById('startBtn').textContent      = '▶ Restart';
  document.getElementById('quitBtn').style.display     = '';
  document.getElementById('overlayHint').textContent   = 'Press R to restart · Q to quit';
  ov.classList.add('show');
}

function intervalForScore(score) {
  let ms = BASE_MS;
  for (const milestone of SPEED_MILESTONES) {
    if (score >= milestone) ms -= 14;
  }
  return Math.max(ms, MIN_MS);
}

function scheduleNextTick() {
  clearTimeout(tickTimer);
  const ms = intervalForScore(state.score);
  tickTimer = setTimeout(() => { tick(); scheduleNextTick(); }, ms);
}

// ─────────────────────────────────────────────
//  HUD
// ─────────────────────────────────────────────
const MAX_PIPS = 5;
function buildPips() {
  const container = document.getElementById('pips');
  container.innerHTML = '';
  for (let i = 0; i < MAX_PIPS; i++) {
    const d = document.createElement('div');
    d.className = 'pip';
    d.id = 'pip' + i;
    container.appendChild(d);
  }
}

function updateHUD() {
  const sc  = state.score;
  const el  = document.getElementById('scoreDisplay');
  el.textContent = String(sc).padStart(4,'0');
  el.classList.remove('bump');
  void el.offsetWidth;
  el.classList.add('bump');

  // Light up pips
  const lit = SPEED_MILESTONES.filter(m => sc >= m).length;
  for (let i = 0; i < MAX_PIPS; i++) {
    document.getElementById('pip' + i).classList.toggle('lit', i < lit);
  }
}

// ─────────────────────────────────────────────
//  START / RESTART
// ─────────────────────────────────────────────
function startGame() {
  clearTimeout(tickTimer);
  particles = [];
  state = freshState();
  foodPulse = 0;
  updateHUD();

  document.getElementById('overlay').classList.remove('show');
  scheduleNextTick();
}

// ─────────────────────────────────────────────
//  INPUT — KEYBOARD
// ─────────────────────────────────────────────
document.addEventListener('keydown', e => {
  const map = {
    ArrowUp:'UP', ArrowDown:'DOWN', ArrowLeft:'LEFT', ArrowRight:'RIGHT',
    w:'UP', s:'DOWN', a:'LEFT', d:'RIGHT',
    W:'UP', S:'DOWN', A:'LEFT', D:'RIGHT',
  };
  if (map[e.key]) {
    e.preventDefault();
    if (state && state.alive && OPPOSITE[state.dir] !== map[e.key]) {
      state.nextDir = map[e.key];
    }
  }
  if (e.key === 'r' || e.key === 'R') startGame();
  if ((e.key === 'q' || e.key === 'Q') && state && !state.alive) {
    window.close();
  }
});

// ─────────────────────────────────────────────
//  INPUT — TOUCH SWIPE
// ─────────────────────────────────────────────
let touchStart = null;
canvas.addEventListener('touchstart', e => {
  touchStart = {x: e.touches[0].clientX, y: e.touches[0].clientY};
}, {passive:true});
canvas.addEventListener('touchend', e => {
  if (!touchStart || !state || !state.alive) return;
  const dx = e.changedTouches[0].clientX - touchStart.x;
  const dy = e.changedTouches[0].clientY - touchStart.y;
  if (Math.abs(dx) < 10 && Math.abs(dy) < 10) return;
  let dir;
  if (Math.abs(dx) > Math.abs(dy)) dir = dx > 0 ? 'RIGHT' : 'LEFT';
  else dir = dy > 0 ? 'DOWN' : 'UP';
  if (OPPOSITE[state.dir] !== dir) state.nextDir = dir;
  touchStart = null;
}, {passive:true});

// ─────────────────────────────────────────────
//  INPUT — D-PAD BUTTONS
// ─────────────────────────────────────────────
function dpadDir(dir) {
  if (state && state.alive && OPPOSITE[state.dir] !== dir) state.nextDir = dir;
}
document.getElementById('dUp').addEventListener('click',    () => dpadDir('UP'));
document.getElementById('dDown').addEventListener('click',  () => dpadDir('DOWN'));
document.getElementById('dLeft').addEventListener('click',  () => dpadDir('LEFT'));
document.getElementById('dRight').addEventListener('click', () => dpadDir('RIGHT'));

// ─────────────────────────────────────────────
//  BUTTONS
// ─────────────────────────────────────────────
document.getElementById('startBtn').addEventListener('click', startGame);
document.getElementById('quitBtn').addEventListener('click',  () => {
  document.getElementById('overlay').classList.add('show');
  document.getElementById('overlayTitle').textContent = 'SNAKE';
  document.getElementById('overlayTitle').className   = 'overlay-title green';
  document.getElementById('overlayScore').style.display = 'none';
  document.getElementById('startBtn').textContent     = '▶ Play';
  document.getElementById('quitBtn').style.display    = 'none';
  document.getElementById('overlayHint').textContent  = 'Arrow keys or D-pad to move';
  clearTimeout(tickTimer);
  state = null;
});

// ─────────────────────────────────────────────
//  INIT
// ─────────────────────────────────────────────
buildPips();
bestScore = parseInt(localStorage.getItem('snakeBest') || '0');
document.getElementById('bestDisplay').textContent = String(bestScore).padStart(4,'0');
particles = [];
renderLoop();
</script>
</body>
</html>
