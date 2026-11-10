<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<title>Sky Blaster</title>
<style>
  :root {
    --sky-top: #7ec8f2;
    --sky-bottom: #bfe9ff;
    --sun: #ffe066;
    --hud-bg: rgba(255,255,255,0.85);
    --accent: #ff6b6b;
    --accent2: #4ecdc4;
    --text: #2b2d42;
  }
  * { box-sizing: border-box; }
  html, body {
    margin: 0;
    height: 100%;
    background: linear-gradient(180deg, #2b2d42, #1a1a2e);
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'Comic Sans MS', 'Trebuchet MS', sans-serif;
    overflow: hidden;
  }
  html, body {
    touch-action: none;
    -webkit-user-select: none;
    user-select: none;
    -webkit-touch-callout: none;
    -webkit-tap-highlight-color: transparent;
  }
  #wrap {
    position: relative;
    width: min(calc(100vw - 24px), calc((100vh - 24px) * 1100 / 650), 1100px);
    border-radius: 20px;
    overflow: hidden;
    box-shadow: 0 20px 60px rgba(0,0,0,0.5), 0 0 0 8px #2b2d42, 0 0 0 12px #ffd166;
  }
  canvas {
    display: block;
    width: 100%;
    height: auto;
    background: linear-gradient(180deg, var(--sky-top), var(--sky-bottom));
    touch-action: none;
  }
  #hud {
    position: absolute;
    top: 14px;
    left: 14px;
    right: 14px;
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 6px;
    pointer-events: none;
    font-weight: bold;
    color: var(--text);
  }
  .pill {
    background: var(--hud-bg);
    padding: 8px 16px;
    border-radius: 999px;
    box-shadow: 0 3px 0 rgba(0,0,0,0.15);
    font-size: 18px;
  }
  #overlay {
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background: rgba(20,20,40,0.55);
    color: white;
    text-align: center;
    gap: 14px;
  }
  #overlay h1 {
    font-size: 42px;
    margin: 0;
    text-shadow: 3px 3px 0 var(--accent);
    letter-spacing: 1px;
  }
  #overlay p { margin: 0; font-size: 16px; max-width: 520px; line-height: 1.5; }
  button {
    font-family: inherit;
    font-size: 18px;
    font-weight: bold;
    padding: 12px 28px;
    border: none;
    border-radius: 999px;
    background: var(--accent);
    color: white;
    cursor: pointer;
    box-shadow: 0 5px 0 #b33939;
    transition: transform 0.05s;
  }
  button:active { transform: translateY(4px); box-shadow: 0 1px 0 #b33939; }
  .hidden { display: none !important; }
  #touchControls {
    position: fixed;
    left: 0; right: 0; bottom: 0;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    padding: 0 max(18px, env(safe-area-inset-right, 0px)) calc(18px + env(safe-area-inset-bottom, 0px)) max(18px, env(safe-area-inset-left, 0px));
    pointer-events: none;
    z-index: 5;
  }
  #stick {
    position: relative;
    width: 150px; height: 150px;
    border-radius: 50%;
    background: rgba(255,255,255,0.28);
    border: 3px solid rgba(255,255,255,0.7);
    box-shadow: 0 4px 14px rgba(0,0,0,0.25);
    pointer-events: auto;
    touch-action: none;
  }
  #knob {
    position: absolute;
    left: 50%; top: 50%;
    width: 64px; height: 64px;
    margin: -32px 0 0 -32px;
    border-radius: 50%;
    background: rgba(255,255,255,0.92);
    box-shadow: 0 3px 0 rgba(0,0,0,0.2);
    pointer-events: none;
  }
  #fireBtn {
    pointer-events: auto;
    touch-action: none;
    width: 120px; height: 120px;
    border-radius: 50%;
    padding: 0;
    font-size: 22px;
    letter-spacing: 1px;
    background: var(--accent);
    color: #fff;
    box-shadow: 0 6px 0 #b33939, 0 8px 16px rgba(0,0,0,0.3);
    opacity: 0.92;
  }
  #fireBtn.down { transform: translateY(5px); box-shadow: 0 1px 0 #b33939; }
  #overlay:not(.hidden) ~ #touchControls { visibility: hidden; }
  .subtext { font-size: 12px; font-weight: normal; opacity: 0.75; margin-left: 4px; }
  #playerSelect { display: flex; gap: 10px; flex-wrap: wrap; justify-content: center; }
  #playerSelect button { font-size: 15px; padding: 10px 18px; }
  @media (max-width: 900px), (max-height: 620px) {
    #overlay { gap: 8px; padding: 10px; }
    #overlay h1 { font-size: 30px; }
    #overlay p { font-size: 14px; line-height: 1.35; }
    #overlay button, #playerSelect button { font-size: 15px; padding: 9px 16px; }
    .pill { font-size: 14px; padding: 5px 11px; }
    #hud { top: 8px; left: 8px; right: 8px; }
    #stick { width: 130px; height: 130px; }
    #fireBtn { width: 104px; height: 104px; }
  }
  @supports (height: 100dvh) {
    #wrap { width: min(calc(100vw - 24px), calc((100dvh - 24px) * 1100 / 650), 1100px); }
  }
</style>


</head>
<body>
<div id="wrap">
  <canvas id="game" width="1100" height="650"></canvas>
  <div id="hud">
    <div class="pill">⭐ <span id="score">0</span> <span id="playerIndicator" class="subtext"></span></div>
    <div class="pill">❤️ <span id="lives">3</span></div>
    <div class="pill">🚩 <span id="round">1</span>/5 <span id="roundTimer" class="subtext">2:30</span></div>
    <div class="pill">🔫 <span id="ammo">30</span>/30s <span id="timer" class="subtext"></span></div>
    <div class="pill">🏆 <span id="highscore">0</span></div>
  </div>
  <div id="overlay">
    <h1>SKY BLASTER</h1>
    <p>Arrow keys / WASD to fly, X (or Space) to shoot forward.<br>On a touchscreen: left stick to fly, big FIRE button to shoot.<br>30 bullets, refilled every 30 seconds. Kill a smiley (1 hit) to get 1 bullet back, kill a UFO (3 hits) to get 3 back!<br>Survive 5 rounds of 2:30 each to beat the game and become 👑 KING! Every round is a new setting (sky, sunset, moon, space, nebula) and the UFOs shoot a bit more.</p>
    <p>Choose how many players:</p>
    <div id="pickerArea"></div>
  </div>
  <div id="touchControls" class="hidden">
    <div id="stick"><div id="knob"></div></div>
    <button id="fireBtn" type="button">FIRE</button>
  </div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const W = canvas.width, H = canvas.height;

const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const ammoEl = document.getElementById('ammo');
const timerEl = document.getElementById('timer');
const highScoreEl = document.getElementById('highscore');
const playerIndicatorEl = document.getElementById('playerIndicator');
const overlay = document.getElementById('overlay');
const roundEl = document.getElementById('round');
const roundTimerEl = document.getElementById('roundTimer');

let keys = {};
window.addEventListener('keydown', e => {
  keys[e.key.toLowerCase()] = true;
  if (e.key === ' ') e.preventDefault();
});
window.addEventListener('keyup', e => keys[e.key.toLowerCase()] = false);

// ---- Touch controls (iPad / phone): left stick to fly, big button to shoot ----
let firing = false;
const joy = { active: false, x: 0, y: 0, id: null };
(function setupTouch() {
  const isTouch = ('ontouchstart' in window) || (navigator.maxTouchPoints > 0);
  const controls = document.getElementById('touchControls');
  if (isTouch) controls.classList.remove('hidden');
  const stick = document.getElementById('stick');
  const knob = document.getElementById('knob');
  const fireBtn = document.getElementById('fireBtn');

  function moveStick(e) {
    const r = stick.getBoundingClientRect();
    const maxr = r.width * 0.32;
    let vx = e.clientX - (r.left + r.width / 2);
    let vy = e.clientY - (r.top + r.height / 2);
    const d = Math.hypot(vx, vy);
    if (d > maxr) { vx = vx / d * maxr; vy = vy / d * maxr; }
    joy.x = vx / maxr;
    joy.y = vy / maxr;
    knob.style.transform = 'translate(' + vx + 'px,' + vy + 'px)';
  }
  function endStick() {
    joy.active = false; joy.x = 0; joy.y = 0; joy.id = null;
    knob.style.transform = '';
  }
  stick.addEventListener('pointerdown', e => {
    e.preventDefault();
    if (joy.active) return;
    joy.active = true;
    joy.id = e.pointerId;
    try { stick.setPointerCapture(e.pointerId); } catch (_) {}
    moveStick(e);
  });
  stick.addEventListener('pointermove', e => {
    if (joy.active && e.pointerId === joy.id) { e.preventDefault(); moveStick(e); }
  });
  ['pointerup', 'pointercancel', 'lostpointercapture'].forEach(t =>
    stick.addEventListener(t, e => { if (e.pointerId === joy.id) endStick(); }));

  fireBtn.addEventListener('pointerdown', e => {
    e.preventDefault();
    firing = true;
    fireBtn.classList.add('down');
    try { fireBtn.setPointerCapture(e.pointerId); } catch (_) {}
  });
  ['pointerup', 'pointercancel', 'lostpointercapture'].forEach(t =>
    fireBtn.addEventListener(t, () => { firing = false; fireBtn.classList.remove('down'); }));

  if (isTouch) document.addEventListener('contextmenu', e => e.preventDefault());
})();

// ---- Game state ----
let running = false;
let score = 0;
let lives = 3;
let frame = 0;
let clouds = [];
let stars = [];
let bullets = [];
let enemies = [];
let enemyBullets = [];
let particles = [];
let powerUps = [];
let shootCooldown = 0;
let invuln = 0;
let difficultyTimer = 0;
let spawnInterval = 70;
let ammoRemaining = 30;
let ammoWindowStart = 0;
const MAX_AMMO_PER_WINDOW = 30;
const AMMO_WINDOW_MS = 30000;

// ---- Rounds & king ----
const TOTAL_ROUNDS = 5;
const ROUND_MS = 150000; // 2.5 minutes per round
let round = 1;
let roundElapsedMs = 0;
let lastTick = 0;
let king = null;      // player number (1-based) who beat the game first this session
let playerWins = [];  // true/false per player: did they beat all rounds?

// Bullets refunded when a shot hits an enemy
const AMMO_REFUND = { blob: 1, ufo: 3, bat: 0 };

let numPlayers = 1;
let currentPlayerIndex = 0;
let playerScores = [];
let highScore = 0;
try { highScore = Number(localStorage.getItem('skyBlasterHighScore') || 0); } catch (_) {}
highScoreEl.textContent = highScore;

const player = {
  x: 90, y: H/2, w: 54, h: 34, speed: 4.6, color: '#4ecdc4'
};

function resetGame() {
  score = 0; lives = 3; frame = 0;
  bullets = []; enemies = []; enemyBullets = []; particles = []; powerUps = [];
  player.x = 90; player.y = H/2;
  shootCooldown = 0; invuln = 90; spawnInterval = 70;
  ammoRemaining = MAX_AMMO_PER_WINDOW;
  ammoWindowStart = performance.now();
  scoreEl.textContent = score;
  livesEl.textContent = lives;
  ammoEl.textContent = MAX_AMMO_PER_WINDOW;
  timerEl.textContent = '';
  const tag = numPlayers > 1 ? `P${currentPlayerIndex+1}/${numPlayers}` : '';
  playerIndicatorEl.textContent = (tag + (king ? ` 👑P${king}` : '')).trim();
  round = 1;
  roundElapsedMs = 0;
  lastTick = performance.now();
  difficultyTimer = 0;
  bannerUntil = performance.now() + 2600;
  updateRoundHud();
}

function beginRound() {
  bullets = []; enemies = []; enemyBullets = []; powerUps = [];
  shootCooldown = 0; invuln = 90;
  ammoRemaining = MAX_AMMO_PER_WINDOW;
  ammoWindowStart = performance.now();
  roundElapsedMs = 0;
  lastTick = performance.now();
  bannerUntil = performance.now() + 2600;
  updateRoundHud();
}

for (let i=0;i<40;i++) {
  clouds.push({x: Math.random()*W, y: Math.random()*H*0.7, r: 20+Math.random()*30, speed: 0.2+Math.random()*0.4});
}

function rand(a,b){ return a + Math.random()*(b-a); }

// UFOs shoot fairly rarely in round 1, a little more each round
function ufoShootDelay(first) {
  const factor = 1 - (round - 1) * 0.12;
  return (first ? rand(110,220) : rand(200,320)) * factor;
}

function updateRoundHud() {
  roundEl.textContent = round;
  const totalSecs = Math.ceil(Math.max(0, ROUND_MS - roundElapsedMs) / 1000);
  const mm = Math.floor(totalSecs / 60);
  const ss = String(totalSecs % 60).padStart(2, '0');
  roundTimerEl.textContent = `${mm}:${ss} · ${currentTheme().name}`;
}

// ---- Enemy types ----
function spawnEnemy() {
  const roll = Math.random();
  let type = 'blob';
  if (roll > 0.7) type = 'ufo';
  if (roll > 0.9) type = 'bat';
  const y = rand(40, H-40);
  const baseSpeed = 1.6 + Math.min(3, score/500);
  enemies.push({
    x: W + 40, y, type,
    w: type==='ufo'?54:44, h: type==='ufo'?30:38,
    speed: baseSpeed + rand(-0.3,0.6),
    amp: rand(20,50), freq: rand(0.02,0.05), t: rand(0,10),
    hp: type==='ufo'?3:1,
    shootTimer: ufoShootDelay(true),
    color: type==='ufo' ? '#c084fc' : (type==='bat' ? '#a597d6' : '#ff9f43')
  });
}

function spawnPowerUp(x,y) {
  if (Math.random() < 0.18) {
    powerUps.push({x, y, r: 12, type: Math.random()<0.5?'life':'rapid', vy: 0.6});
  }
}

// ---- Drawing helpers ----
function drawPlayer() {
  ctx.save();
  ctx.translate(player.x, player.y);
  if (isDarkTheme()) { ctx.shadowColor = 'rgba(255,255,255,0.45)'; ctx.shadowBlur = 8; }
  if (invuln > 0 && frame % 10 < 5) ctx.globalAlpha = 0.4;

  // body
  ctx.fillStyle = player.color;
  ctx.beginPath();
  ctx.ellipse(0, 0, 28, 15, 0, 0, Math.PI*2);
  ctx.fill();

  // nose
  ctx.fillStyle = '#ffe066';
  ctx.beginPath();
  ctx.moveTo(24, 0);
  ctx.lineTo(38, -4);
  ctx.lineTo(38, 4);
  ctx.closePath();
  ctx.fill();

  // fin
  ctx.fillStyle = '#ff6b6b';
  ctx.beginPath();
  ctx.moveTo(-10,-6);
  ctx.lineTo(-26,-18);
  ctx.lineTo(-6,-10);
  ctx.closePath();
  ctx.fill();
  ctx.beginPath();
  ctx.moveTo(-10,6);
  ctx.lineTo(-26,18);
  ctx.lineTo(-6,10);
  ctx.closePath();
  ctx.fill();

  // cockpit
  ctx.fillStyle = '#2b2d42';
  ctx.beginPath();
  ctx.ellipse(6,-2,7,6,0,0,Math.PI*2);
  ctx.fill();
  ctx.fillStyle = '#a3f7ff';
  ctx.beginPath();
  ctx.ellipse(7,-3,3,2.5,0,0,Math.PI*2);
  ctx.fill();

  // flame
  const flameLen = 10 + Math.sin(frame*0.5)*4;
  ctx.fillStyle = '#ffb703';
  ctx.beginPath();
  ctx.moveTo(-26,-5);
  ctx.lineTo(-26-flameLen,0);
  ctx.lineTo(-26,5);
  ctx.closePath();
  ctx.fill();

  ctx.restore();
}

function drawEnemy(e) {
  ctx.save();
  ctx.translate(e.x, e.y);
  if (isDarkTheme()) { ctx.shadowColor = 'rgba(255,255,255,0.55)'; ctx.shadowBlur = 10; }
  if (e.type === 'ufo') {
    ctx.fillStyle = e.color;
    ctx.beginPath();
    ctx.ellipse(0,0,26,10,0,0,Math.PI*2);
    ctx.fill();
    ctx.fillStyle = '#e0aaff';
    ctx.beginPath();
    ctx.ellipse(0,-8,12,10,0,Math.PI,0);
    ctx.fill();
    ctx.fillStyle = '#fff';
    for (let i=-1;i<=1;i++){
      ctx.beginPath();
      ctx.arc(i*14, 4, 3, 0, Math.PI*2);
      ctx.fill();
    }
  } else if (e.type === 'bat') {
    ctx.fillStyle = e.color;
    const wing = Math.sin(frame*0.3)*10;
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.quadraticCurveTo(-20, -10+wing, -30, 4+wing);
    ctx.quadraticCurveTo(-14,4,0,0);
    ctx.fill();
    ctx.beginPath();
    ctx.moveTo(0,0);
    ctx.quadraticCurveTo(20, -10-wing, 30, 4-wing);
    ctx.quadraticCurveTo(14,4,0,0);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(0,0,10,9,0,0,Math.PI*2);
    ctx.fill();
    ctx.fillStyle='#ff6b6b';
    ctx.beginPath(); ctx.arc(-4,-1,2,0,Math.PI*2); ctx.fill();
    ctx.beginPath(); ctx.arc(4,-1,2,0,Math.PI*2); ctx.fill();
  } else {
    ctx.fillStyle = e.color;
    ctx.beginPath();
    ctx.arc(0,0,20,0,Math.PI*2);
    ctx.fill();
    ctx.fillStyle = '#fff';
    ctx.beginPath(); ctx.arc(-6,-4,5,0,Math.PI*2); ctx.arc(6,-4,5,0,Math.PI*2); ctx.fill();
    ctx.fillStyle = '#2b2d42';
    ctx.beginPath(); ctx.arc(-6,-4,2.4,0,Math.PI*2); ctx.arc(6,-4,2.4,0,Math.PI*2); ctx.fill();
    ctx.strokeStyle = '#2b2d42';
    ctx.lineWidth = 2;
    ctx.beginPath(); ctx.arc(0,4,7,0,Math.PI); ctx.stroke();
  }
  ctx.restore();
}

// ---- Level settings: one scene per round ----
const THEMES = [
  { name: 'Sunny Sky',    kind: 'sky' },
  { name: 'Sunset Skies', kind: 'sunset' },
  { name: 'The Moon',     kind: 'moon' },
  { name: 'Outer Space',  kind: 'space' },
  { name: 'Nebula Core',  kind: 'nebula' }
];
function currentTheme() { return THEMES[Math.min(round, TOTAL_ROUNDS) - 1]; }
function isDarkTheme() { return ['moon','space','nebula'].includes(currentTheme().kind); }

let bannerUntil = 0;
let planetX = W * 0.72, planet2X = W * 0.2;
const moonCraters = [];
for (let i = 0; i < 16; i++) {
  moonCraters.push({ x: i * (W / 14) + Math.random() * 40, w: 30 + Math.random() * 60, h: 5 + Math.random() * 7 });
}
const nebulaBlobs = [
  { x: 200, y: 160, r: 280, c: '190,60,210', vx: 0.12 },
  { x: 640, y: 480, r: 320, c: '40,170,210', vx: 0.09 },
  { x: 930, y: 190, r: 250, c: '240,90,140', vx: 0.15 },
  { x: 430, y: 570, r: 230, c: '90,70,230',  vx: 0.10 }
];
for (let i = 0; i < 170; i++) {
  stars.push({ x: Math.random() * W, y: Math.random() * H, r: 0.6 + Math.random() * 1.6,
               layer: 1 + Math.floor(Math.random() * 3), tw: Math.random() * 6.28 });
}

function fillGradient(stops) {
  const g = ctx.createLinearGradient(0, 0, 0, H);
  stops.forEach(([o, c]) => g.addColorStop(o, c));
  ctx.fillStyle = g;
  ctx.fillRect(0, 0, W, H);
}

function drawClouds(color) {
  ctx.fillStyle = color;
  clouds.forEach(c => {
    ctx.beginPath();
    ctx.ellipse(c.x, c.y, c.r, c.r*0.6, 0, 0, Math.PI*2);
    ctx.ellipse(c.x + c.r*0.7, c.y + 4, c.r*0.7, c.r*0.45, 0, 0, Math.PI*2);
    ctx.ellipse(c.x - c.r*0.7, c.y + 6, c.r*0.6, c.r*0.4, 0, 0, Math.PI*2);
    ctx.fill();
    if (running) c.x -= c.speed;
    if (c.x < -60) c.x = W + 60;
  });
}

function drawStars(speed) {
  stars.forEach(st => {
    if (running) {
      st.x -= st.layer * speed;
      if (st.x < -2) { st.x = W + 2; st.y = Math.random() * H; }
    }
    ctx.globalAlpha = 0.6 + 0.4 * Math.sin(frame * 0.05 + st.tw);
    ctx.fillStyle = '#fff';
    ctx.beginPath();
    ctx.arc(st.x, st.y, st.r, 0, Math.PI*2);
    ctx.fill();
  });
  ctx.globalAlpha = 1;
}

function drawSun(x, y, r, core, glowRGB) {
  const g = ctx.createRadialGradient(x, y, r*0.4, x, y, r*3);
  g.addColorStop(0, `rgba(${glowRGB},0.55)`);
  g.addColorStop(1, `rgba(${glowRGB},0)`);
  ctx.fillStyle = g;
  ctx.beginPath(); ctx.arc(x, y, r*3, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = core;
  ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI*2); ctx.fill();
}

function drawEarth(x, y, r) {
  const glow = ctx.createRadialGradient(x, y, r*0.9, x, y, r*1.6);
  glow.addColorStop(0, 'rgba(120,190,255,0.45)');
  glow.addColorStop(1, 'rgba(120,190,255,0)');
  ctx.fillStyle = glow;
  ctx.beginPath(); ctx.arc(x, y, r*1.6, 0, Math.PI*2); ctx.fill();

  ctx.save();
  ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI*2); ctx.clip();
  const body = ctx.createRadialGradient(x - r*0.4, y - r*0.4, r*0.1, x, y, r);
  body.addColorStop(0, '#6ec3ff');
  body.addColorStop(1, '#1b4f9c');
  ctx.fillStyle = body;
  ctx.fillRect(x - r, y - r, r*2, r*2);
  ctx.fillStyle = '#4caf6a';
  ctx.beginPath(); ctx.ellipse(x - r*0.3, y - r*0.15, r*0.38, r*0.26, -0.5, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.ellipse(x + r*0.35, y + r*0.35, r*0.3, r*0.2, 0.4, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.ellipse(x + r*0.1, y - r*0.6, r*0.25, r*0.12, 0, 0, Math.PI*2); ctx.fill();
  ctx.fillStyle = 'rgba(255,255,255,0.7)';
  ctx.beginPath(); ctx.ellipse(x + r*0.2, y - r*0.05, r*0.45, r*0.07, -0.2, 0, Math.PI*2); ctx.fill();
  ctx.beginPath(); ctx.ellipse(x - r*0.4, y + r*0.5, r*0.35, r*0.06, 0.3, 0, Math.PI*2); ctx.fill();
  ctx.restore();
}

function drawRingedPlanet(x, y, r) {
  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(-0.35);
  ctx.lineWidth = r * 0.2;
  ctx.strokeStyle = 'rgba(235,205,150,0.75)';
  ctx.beginPath(); ctx.ellipse(0, 0, r*1.9, r*0.5, 0, 0, Math.PI*2); ctx.stroke();
  const body = ctx.createRadialGradient(-r*0.4, -r*0.4, r*0.1, 0, 0, r);
  body.addColorStop(0, '#ffd9a0');
  body.addColorStop(1, '#b3612f');
  ctx.fillStyle = body;
  ctx.beginPath(); ctx.arc(0, 0, r, 0, Math.PI*2); ctx.fill();
  ctx.strokeStyle = 'rgba(235,205,150,0.95)';
  ctx.beginPath(); ctx.ellipse(0, 0, r*1.9, r*0.5, 0, 0, Math.PI); ctx.stroke();
  ctx.restore();
}

function drawSmallPlanet(x, y, r) {
  const body = ctx.createRadialGradient(x - r*0.4, y - r*0.4, r*0.1, x, y, r);
  body.addColorStop(0, '#d98a8a');
  body.addColorStop(1, '#6b2c3a');
  ctx.fillStyle = body;
  ctx.beginPath(); ctx.arc(x, y, r, 0, Math.PI*2); ctx.fill();
}

function drawMoonGround() {
  const top = H - 62;
  const g = ctx.createLinearGradient(0, top, 0, H);
  g.addColorStop(0, '#a7a7b3');
  g.addColorStop(1, '#5d5d6b');
  ctx.fillStyle = g;
  ctx.beginPath();
  ctx.moveTo(0, top + 10);
  for (let x = 0; x <= W; x += 40) ctx.lineTo(x, top + Math.sin(x * 0.02) * 6);
  ctx.lineTo(W, H); ctx.lineTo(0, H); ctx.closePath();
  ctx.fill();
  ctx.fillStyle = 'rgba(40,40,55,0.45)';
  moonCraters.forEach(c => {
    ctx.beginPath();
    ctx.ellipse(c.x, top + 26 + (c.w % 14), c.w, c.h, 0, 0, Math.PI*2);
    ctx.fill();
    if (running) c.x -= 1.6;
    if (c.x < -c.w) c.x = W + c.w;
  });
}

function drawBackground() {
  const kind = currentTheme().kind;

  if (kind === 'sky') {
    fillGradient([[0, '#7ec8f2'], [1, '#bfe9ff']]);
    drawSun(W - 140, 110, 46, '#fff3b0', '255,230,120');
    drawClouds('rgba(255,255,255,0.8)');

  } else if (kind === 'sunset') {
    fillGradient([[0, '#3d2a72'], [0.45, '#c3507c'], [0.75, '#ff8a5c'], [1, '#ffd08a']]);
    drawSun(W * 0.72, H * 0.78, 70, '#fff0c0', '255,170,90');
    drawClouds('rgba(255,170,160,0.65)');

  } else if (kind === 'moon') {
    fillGradient([[0, '#04050d'], [1, '#141a33']]);
    drawStars(0.15);
    drawEarth(W - 170, 130, 62);
    drawMoonGround();

  } else if (kind === 'space') {
    fillGradient([[0, '#02030a'], [1, '#0a0f2a']]);
    drawStars(0.6);
    if (running) {
      planetX -= 0.12; planet2X -= 0.2;
      if (planetX < -220) planetX = W + 220;
      if (planet2X < -80) planet2X = W + 80;
    }
    drawSmallPlanet(planet2X, H * 0.78, 34);
    drawRingedPlanet(planetX, H * 0.3, 52);

  } else {
    fillGradient([[0, '#0a0420'], [1, '#12062e']]);
    ctx.globalCompositeOperation = 'lighter';
    nebulaBlobs.forEach(n => {
      const g = ctx.createRadialGradient(n.x, n.y, 0, n.x, n.y, n.r);
      g.addColorStop(0, `rgba(${n.c},0.42)`);
      g.addColorStop(1, `rgba(${n.c},0)`);
      ctx.fillStyle = g;
      ctx.beginPath(); ctx.arc(n.x, n.y, n.r, 0, Math.PI*2); ctx.fill();
      if (running) { n.x -= n.vx; if (n.x < -n.r) n.x = W + n.r; }
    });
    ctx.globalCompositeOperation = 'source-over';
    drawStars(0.5);
  }
}

function drawBanner() {
  const left = bannerUntil - performance.now();
  if (left <= 0) return;
  ctx.save();
  ctx.globalAlpha = Math.min(1, left / 600);
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.lineJoin = 'round';
  ctx.lineWidth = 8;
  ctx.strokeStyle = 'rgba(20,20,40,0.8)';
  ctx.fillStyle = '#fff';
  ctx.font = 'bold 54px "Comic Sans MS","Trebuchet MS",sans-serif';
  ctx.strokeText(`ROUND ${round}`, W/2, H/2 - 30);
  ctx.fillText(`ROUND ${round}`, W/2, H/2 - 30);
  ctx.font = 'bold 32px "Comic Sans MS","Trebuchet MS",sans-serif';
  ctx.strokeText(currentTheme().name, W/2, H/2 + 26);
  ctx.fillText(currentTheme().name, W/2, H/2 + 26);
  ctx.restore();
}

// ---- Update loop ----
function update() {
  frame++;

  // round clock (real time, so it is 2:30 on any screen refresh rate)
  const tickNow = performance.now();
  roundElapsedMs += Math.min(50, tickNow - lastTick);
  lastTick = tickNow;

  // player movement
  let dx = 0, dy = 0;
  if (keys['arrowup'] || keys['w']) dy -= 1;
  if (keys['arrowdown'] || keys['s']) dy += 1;
  if (keys['arrowleft'] || keys['a']) dx -= 1;
  if (keys['arrowright'] || keys['d']) dx += 1;

  let mx = 0, my = 0;
  const klen = Math.hypot(dx, dy);
  if (klen > 0) { mx = dx / klen; my = dy / klen; }
  if (joy.active && Math.hypot(joy.x, joy.y) > 0.12) { mx = joy.x; my = joy.y; }
  player.x += mx * player.speed;
  player.y += my * player.speed;
  player.x = Math.max(30, Math.min(W-200, player.x));
  player.y = Math.max(24, Math.min(H-24, player.y));

  // ammo: fully recharges to 30 every 30 seconds (fixed clip, not rolling)
  const nowMs = performance.now();
  const elapsed = nowMs - ammoWindowStart;
  if (elapsed >= AMMO_WINDOW_MS) {
    ammoRemaining = MAX_AMMO_PER_WINDOW;
    ammoWindowStart = nowMs;
  }

  // countdown to the next full recharge
  const secsLeft = Math.max(0, (AMMO_WINDOW_MS - elapsed) / 1000);
  timerEl.textContent = `(refill in ${secsLeft.toFixed(1)}s)`;

  // shooting — X (or Space) fires a single shot straight forward
  shootCooldown--;
  const wantsFire = keys['x'] || keys[' '] || firing;
  if (wantsFire && shootCooldown <= 0) {
    if (ammoRemaining > 0) {
      bullets.push({x: player.x+34, y: player.y, vx: 9});
      ammoRemaining--;
      shootCooldown = rapidUntil > frame ? 6 : 12;
    } else {
      shootCooldown = 8; // out of ammo — brief pause before rechecking
    }
  }

  bullets.forEach(b => b.x += b.vx);
  bullets = bullets.filter(b => b.x < W+20);

  enemyBullets.forEach(b => { b.x += b.vx; b.y += b.vy; });
  enemyBullets = enemyBullets.filter(b => b.x > -20 && b.x < W+20);

  // spawn enemies
  difficultyTimer++;
  if (frame % Math.max(28, spawnInterval - Math.floor(difficultyTimer/300)) === 0) {
    spawnEnemy();
  }

  enemies.forEach(e => {
    e.t += e.freq;
    e.x -= e.speed;
    e.y += Math.sin(e.t) * (e.amp*0.02);
    e.shootTimer--;
    if (e.type === 'ufo' && e.shootTimer <= 0 && e.x < W-40) {
      const angle = Math.atan2(player.y-e.y, player.x-e.x);
      enemyBullets.push({x:e.x, y:e.y, vx: Math.cos(angle)*4.2, vy: Math.sin(angle)*4.2});
      e.shootTimer = ufoShootDelay(false);
    }
  });
  enemies = enemies.filter(e => e.x > -60);

  // bullet-enemy collisions
  bullets.forEach(b => {
    if (b.hit) return;
    enemies.forEach(e => {
      if (b.hit || e.dead) return;
      if (Math.hypot(b.x-e.x, b.y-e.y) < (e.type==='ufo'?30:24)) {
        e.hp--;
        b.hit = true;

        if (e.hp <= 0) {
          e.dead = true;

          // ammo refund when you destroy it (smiley = 1, UFO = 3)
          const refund = AMMO_REFUND[e.type] || 0;
          if (refund > 0) {
            ammoRemaining = Math.min(MAX_AMMO_PER_WINDOW, ammoRemaining + refund);
            spawnParticles(e.x, e.y, '#ffd166');
          }

          score += e.type==='ufo'?30:e.type==='bat'?20:10;
          scoreEl.textContent = score;
          spawnParticles(e.x, e.y, e.color);
          spawnPowerUp(e.x, e.y);
        }
      }
    });
  });
  bullets = bullets.filter(b => !b.hit);
  enemies = enemies.filter(e => !e.dead);

  // enemy-player collisions
  if (invuln > 0) invuln--;
  enemies.forEach(e => {
    if (invuln<=0 && Math.hypot(player.x-e.x, player.y-e.y) < 30) {
      e.dead = true;
      hitPlayer();
    }
  });
  enemyBullets.forEach(b => {
    if (invuln<=0 && Math.hypot(player.x-b.x, player.y-b.y) < 16) {
      b.hit = true;
      hitPlayer();
    }
  });
  enemies = enemies.filter(e => !e.dead);
  enemyBullets = enemyBullets.filter(b => !b.hit);

  // power-ups
  powerUps.forEach(p => { p.x -= 2; });
  powerUps = powerUps.filter(p => {
    if (Math.hypot(player.x-p.x, player.y-p.y) < 26) {
      if (p.type==='life') { lives = Math.min(5,lives+1); livesEl.textContent = lives; }
      else { rapidUntil = frame + 300; }
      spawnParticles(p.x,p.y,'#ffe066');
      return false;
    }
    return p.x > -20;
  });

  // particles
  particles.forEach(p => { p.x+=p.vx; p.y+=p.vy; p.life--; p.vy += 0.08; });
  particles = particles.filter(p => p.life > 0);

  // HUD ammo (updated after refunds so it's always current)
  ammoEl.textContent = ammoRemaining;
  ammoEl.style.color = ammoRemaining <= 5 ? '#ff4d6d' : '';

  updateRoundHud();

  if (lives <= 0) endGame();
  else if (roundElapsedMs >= ROUND_MS) roundComplete();
}

let rapidUntil = 0;

function hitPlayer() {
  lives--;
  livesEl.textContent = lives;
  invuln = 90;
  spawnParticles(player.x, player.y, '#ff6b6b');
}

function spawnParticles(x,y,color) {
  for (let i=0;i<12;i++) {
    const a = Math.random()*Math.PI*2;
    const speed = rand(1,4);
    particles.push({x,y, vx: Math.cos(a)*speed, vy: Math.sin(a)*speed, life: 30+Math.random()*15, color});
  }
}

function drawBullets() {
  ctx.fillStyle = '#ffd166';
  bullets.forEach(b => {
    ctx.beginPath();
    ctx.ellipse(b.x,b.y,8,3,0,0,Math.PI*2);
    ctx.fill();
  });
  ctx.fillStyle = '#ff4d6d';
  enemyBullets.forEach(b => {
    ctx.beginPath();
    ctx.arc(b.x,b.y,5,0,Math.PI*2);
    ctx.fill();
  });
}

function drawPowerUps() {
  powerUps.forEach(p => {
    ctx.save();
    ctx.translate(p.x,p.y);
    ctx.fillStyle = p.type==='life' ? '#ff6b6b' : '#4ecdc4';
    ctx.beginPath();
    ctx.arc(0,0,13,0,Math.PI*2);
    ctx.fill();
    ctx.fillStyle = 'white';
    ctx.font = 'bold 14px sans-serif';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(p.type==='life' ? '+' : '⚡', 0, 1);
    ctx.restore();
  });
}

function drawParticles() {
  particles.forEach(p => {
    ctx.globalAlpha = Math.max(0, p.life/40);
    ctx.fillStyle = p.color;
    ctx.beginPath();
    ctx.arc(p.x,p.y,3,0,Math.PI*2);
    ctx.fill();
    ctx.globalAlpha = 1;
  });
}

function draw() {
  drawBackground();
  drawPowerUps();
  enemies.forEach(drawEnemy);
  drawBullets();
  drawParticles();
  drawPlayer();
  drawBanner();
}

function loop() {
  if (!running) return;
  update();
  draw();
  requestAnimationFrame(loop);
}

function endGame() { finishTurn(false); }

function roundComplete() {
  running = false;
  if (round >= TOTAL_ROUNDS) { finishTurn(true); return; }
  const cleared = round;
  round++;
  roundEl.textContent = round;
  drawBackground(); // preview the next setting behind the overlay
  overlay.classList.remove('hidden');
  overlay.innerHTML = `
    <h1>ROUND ${cleared} CLEAR!</h1>
    <p>Score: ${score}</p>
    <p>Next setting: <b>${currentTheme().name}</b><br>Round ${round} of ${TOTAL_ROUNDS}: the UFOs will shoot a little more.</p>
    <button id="nextRoundBtn">Start Round ${round}</button>
  `;
  document.getElementById('nextRoundBtn').addEventListener('click', () => {
    beginRound();
    overlay.classList.add('hidden');
    running = true;
    loop();
  });
}

function finishTurn(won) {
  running = false;
  const playerNum = currentPlayerIndex + 1;
  const lastPlayer = currentPlayerIndex >= numPlayers - 1;
  playerScores.push(score);
  playerWins.push(won);

  let crowned = false;
  if (won && king === null) { king = playerNum; crowned = true; }

  let heading;
  if (won) heading = 'YOU BEAT SKY BLASTER!';
  else heading = lastPlayer ? 'GAME OVER' : `PLAYER ${playerNum} DOWN!`;

  const trophy = won ? '<p>🏆 Award: Sky Blaster Champion!</p>' : '';

  if (!lastPlayer) {
    // more players still waiting for their turn
    currentPlayerIndex++;
    const status = won
      ? (crowned ? `<p>👑 Player ${playerNum} is the KING of the game!</p>`
                 : `<p>👑 Player ${king} got there first and stays king.</p>`)
      : `<p>Reached Round ${round} of ${TOTAL_ROUNDS}</p>`;
    overlay.classList.remove('hidden');
    overlay.innerHTML = `
      <h1>${heading}</h1>
      <p>Score: ${score}</p>
      ${trophy}
      ${status}
      <button id="nextPlayerBtn">Start Player ${currentPlayerIndex+1}</button>
    `;
    document.getElementById('nextPlayerBtn').addEventListener('click', () => {
      resetGame();
      overlay.classList.add('hidden');
      running = true;
      loop();
    });
    return;
  }

  // last player's turn just ended: show final results
  const best = Math.max(...playerScores);
  let newRecord = false;
  if (best > highScore) {
    highScore = best;
    try { localStorage.setItem('skyBlasterHighScore', String(highScore)); } catch (_) {}
    highScoreEl.textContent = highScore;
    newRecord = true;
  }
  const scoreLines = playerScores
    .map((s, i) => `Player ${i+1}: ${s}${s === best ? ' 🏅' : ''}${(i+1) === king ? ' 👑' : ''}${playerWins[i] ? ' ✅' : ''}`)
    .join('<br>');
  const kingLine = king
    ? `<p>👑 Player ${king} is the KING for now! The crown goes away when a new game starts.</p>`
    : '<p>Nobody beat the game, so there is no king this time.</p>';
  overlay.classList.remove('hidden');
  overlay.innerHTML = `
    <h1>${heading}</h1>
    ${trophy}
    <p>${scoreLines}</p>
    ${kingLine}
    ${newRecord ? '<p>🏆 New High Score!</p>' : ''}
    <p>High Score: ${highScore}</p>
    <p>Choose how many players:</p>
    <div id="pickerArea"></div>
  `;
  showPlayerCountPicker();
}

function startGame(n) {
  numPlayers = n;
  currentPlayerIndex = 0;
  playerScores = [];
  playerWins = [];
  king = null; // new game: the old king is dethroned
  resetGame();
  overlay.classList.add('hidden');
  running = true;
  loop();
}

function playerCountMarkup() {
  return `
    <div id="playerSelect">
      <button data-players="1">1 Player</button>
      <button data-players="2">2 Players</button>
      <button data-players="3">3 Players</button>
      <button data-players="4">4 Players</button>
    </div>`;
}

function showPlayerCountPicker() {
  const area = document.getElementById('pickerArea');
  area.innerHTML = playerCountMarkup();
  area.querySelectorAll('#playerSelect button').forEach(btn => {
    btn.addEventListener('click', () => startGame(parseInt(btn.dataset.players, 10)));
  });
}

showPlayerCountPicker();

// idle background animation before start
function idleDraw() {
  drawBackground();
  if (!running) requestAnimationFrame(idleDraw);
}
idleDraw();
</script>
</body>
</html>
