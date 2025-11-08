<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Duel — 2 Player</title>
<style>
  :root{
    --bg:#0f1724;
    --panel:#0b1220;
    --accent:#7dd3fc;
    --accent-2: #a78bfa;
    --muted:#94a3b8;
    --hp-bg:#243244;
    --hp-lose:#ef4444;
    --hp-win:#34d399;
    font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
  }
  html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071028);color:#e6eef8;}
  .wrap{
    display:flex;
    flex-direction:column;
    min-height:100vh;
    align-items:center;
    justify-content:center;
    gap:18px;
    padding:24px;
  }
  .ui{
    width:min(1000px,96vw);
    display:flex;
    justify-content:space-between;
    align-items:center;
    gap:16px;
    background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
    border:1px solid rgba(255,255,255,0.03);
    padding:12px;
    border-radius:14px;
    box-shadow: 0 6px 30px rgba(2,6,23,0.6);
  }
  .player-panel{
    width:48%;
    display:flex;
    align-items:center;
    gap:12px;
  }
  .hp{
    flex:1;
    background:var(--hp-bg);
    border-radius:10px;
    height:14px;
    overflow:hidden;
    position:relative;
  }
  .hp .bar{
    height:100%;
    width:100%;
    background:linear-gradient(90deg,var(--hp-win),var(--accent));
    transform-origin:left;
    transition:width .22s linear, background .22s linear;
  }
  .hp .label{
    position:absolute;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    font-size:11px;color:var(--muted);
    font-weight:600;
    letter-spacing:0.5px;
  }
  .names{font-size:13px;color:var(--muted);font-weight:600;}
  .center-controls{display:flex;flex-direction:column;align-items:center;gap:6px;}
  button{
    background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:var(--muted);cursor:pointer;
  }
  button:hover{border-color:rgba(255,255,255,0.12);color:var(--accent);}
  canvas{
    display:block;
    width:min(1000px,96vw);
    height:420px;
    border-radius:12px;
    background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));
    box-shadow: 0 20px 50px rgba(2,6,23,0.6);
  }
  .instructions{
    max-width:1000px;
    color:var(--muted);
    font-size:13px;
    text-align:center;
    margin-top:2px;
  }
  .overlay{
    position:fixed;inset:0;display:flex;align-items:center;justify-content:center;
    background:linear-gradient(0deg, rgba(2,6,23,0.55), rgba(2,6,23,0.55));
    z-index:20;
  }
  .card{
    background:var(--panel);padding:20px;border-radius:14px;border:1px solid rgba(255,255,255,0.04);
    text-align:center;max-width:560px;color:#dbeafe;
  }
  .title{font-size:20px;margin-bottom:8px;}
  .small{font-size:13px;color:var(--muted);margin-bottom:12px;}
  .footer{font-size:12px;color:var(--muted);margin-top:10px;}
  @media (max-width:680px){
    canvas{height:340px}
    .ui{flex-direction:column;align-items:stretch}
    .player-panel{width:100%}
  }
</style>
</head>
<body>
<div class="wrap">
  <div class="ui" role="region" aria-label="game UI">
    <div class="player-panel">
      <div class="names">Player 1 — WASD</div>
      <div style="width:12px"></div>
      <div class="hp" id="hp1" aria-hidden="false">
        <div class="bar" id="bar1"></div>
        <div class="label" id="label1">100%</div>
      </div>
    </div>

    <div class="center-controls">
      <div style="font-weight:700; color:var(--muted); letter-spacing:1px">Stickman Duel</div>
      <div style="display:flex;gap:8px">
        <button id="restart">Restart</button>
        <button id="toggleSound">Toggle Sound</button>
        <button id="fullscreenBtn">Fullscreen</button>
      </div>
    </div>

    <div class="player-panel" style="justify-content:flex-end;">
      <div class="hp" id="hp2">
        <div class="bar" id="bar2" style="background:linear-gradient(90deg,var(--accent-2),#f472b6)"></div>
        <div class="label" id="label2">100%</div>
      </div>
      <div style="width:12px"></div>
      <div class="names" style="text-align:right">Player 2 — Arrows</div>
    </div>
  </div>

  <canvas id="game" width="1200" height="520"></canvas>

  <div class="instructions">
    Controls — Player 1: W A D (move/jump) + S (punch) · Player 2: ↑ ← → (move/jump) + ↓ (punch) · First to deplete opponent wins.
  </div>
</div>

<!-- Overlay for start / win -->
<div id="overlay" class="overlay" style="display:flex">
  <div class="card">
    <div class="title">Stickman Duel</div>
    <div class="small">Two-player local — simple, modern, and responsive.</div>
    <div style="margin-bottom:12px">
      <strong>Player 1</strong> — WASD &middot; <strong>Player 2</strong> — Arrow keys
    </div>
    <div style="display:flex;gap:8px;justify-content:center">
      <button id="startBtn">Start Match</button>
      <button id="demoBtn">Demo (AI vs AI)</button>
    </div>
    <div class="footer">Hit Restart to reset anytime. Sound is synth-based and safe for web (no external files).</div>
  </div>
</div>

<script>
/* =========================
   Game constants & helpers
   ========================= */
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d', { alpha: false });
let cw = canvas.width, ch = canvas.height;

// Resize canvas for fullscreen with proper scaling
function resizeCanvasToDisplay() {
  const aspect = 1200 / 520;
  let width = window.innerWidth;
  let height = window.innerHeight;

  if(width / height > aspect){
    width = height * aspect;
  } else {
    height = width / aspect;
  }

  canvas.width = 1200;
  canvas.height = 520;
  canvas.style.width = width + 'px';
  canvas.style.height = height + 'px';
}
window.addEventListener('resize', resizeCanvasToDisplay);
resizeCanvasToDisplay();

/* =========================
   Audio (WebAudio synth sounds)
   ========================= */
const AudioEnabled = { val: true };
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

function playTone(freq, type='sine', length=0.12, gain=0.12, decay=0.02){
  if(!AudioEnabled.val) return;
  const now = audioCtx.currentTime;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type = type;
  o.frequency.setValueAtTime(freq, now);
  g.gain.setValueAtTime(gain, now);
  g.gain.exponentialRampToValueAtTime(0.0001, now + length + decay);
  o.connect(g); g.connect(audioCtx.destination);
  o.start(now); o.stop(now + length + decay);
}

function playPunchSound() {
  playTone(600,'square',0.08,0.09,0.02);
  setTimeout(()=>playTone(420,'sawtooth',0.12,0.06,0.03), 10);
}
function playHitSound() {
  playTone(160,'sine',0.18,0.15,0.03);
  setTimeout(()=>playTone(220,'triangle',0.1,0.08,0.02), 30);
}
function playWinSound() {
  playTone(880,'sine',0.28,0.14,0.04);
  setTimeout(()=>playTone(1100,'sine',0.22,0.12,0.03), 200);
}
function backgroundDrone(){
  if(!AudioEnabled.val) return;
  const o = audioCtx.createOscillator();
  const g = audioCtx.createGain();
  o.type='sine'; o.frequency.value = 55;
  g.gain.value = 0.02;
  o.connect(g); g.connect(audioCtx.destination);
  o.start();
  return {osc:o,gain:g};
}
let bgNode = null;

/* =========================
   Player object
   ========================= */
class Player {
  constructor(x, color, facing=1){
    this.x = x;
    this.y = 0;
    this.vx = 0;
    this.vy = 0;
    this.width = 36;
    this.height = 70;
    this.onGround = false;
    this.health = 100;
    this.color = color;
    this.facing = facing;
    this.punching = false;
    this.punchTimer = 0;
    this.stun = 0;
  }
  rect(){ return {x:this.x - this.width/2, y:this.y - this.height, w:this.width, h:this.height}; }
  punchHitbox(){
    const range = 38;
    const px = this.x + this.facing * (this.width/2 + range/2);
    const py = this.y - this.height/2;
    return {x: px - range/2, y: py - 16, w: range, h: 32};
  }
}

/* =========================
   World & players
   ========================= */
const groundY = ch - 80;
const leftBound = 80;
const rightBound = cw - 80;
const p1 = new Player(360, '#7dd3fc', 1);
const p2 = new Player(840, '#f472b6', -1);

let keys = {};
let lastTime = performance.now();
let running = false;
let demoMode = false;

/* =========================
   Input
   ========================= */
window.addEventListener('keydown', (e)=>{ keys[e.code] = true; });
window.addEventListener('keyup', (e)=>{ keys[e.code] = false; });

/* =========================
   UI elements
   ========================= */
const bar1 = document.getElementById('bar1');
const bar2 = document.getElementById('bar2');
const label1 = document.getElementById('label1');
const label2 = document.getElementById('label2');
const overlay = document.getElementById('overlay');

document.getElementById('startBtn').addEventListener('click', ()=>{
  overlay.style.display='none';
  startMatch();
});
document.getElementById('demoBtn').addEventListener('click', ()=>{
  overlay.style.display='none';
  demoMode = true;
  startMatch();
});
document.getElementById('restart').addEventListener('click', ()=>{ resetMatch(); });
document.getElementById('toggleSound').addEventListener('click', ()=>{
  AudioEnabled.val = !AudioEnabled.val;
  if(!AudioEnabled.val && bgNode){ bgNode.osc.stop(); bgNode = null; }
  if(AudioEnabled.val && running && !bgNode) bgNode = backgroundDrone();
  document.getElementById('toggleSound').textContent = AudioEnabled.val ? 'Sound: On' : 'Sound: Off';
});

/* =========================
   Fullscreen functionality
   ========================= */
const fsBtn = document.getElementById('fullscreenBtn');
fsBtn.addEventListener('click', () => {
  if (!document.fullscreenElement) {
    canvas.requestFullscreen().catch(err => {
      console.error(`Error attempting to enable fullscreen: ${err.message}`);
    });
  } else {
    document.exitFullscreen();
  }
});
document.addEventListener('fullscreenchange', () => {
  fsBtn.textContent = document.fullscreenElement ? 'Exit Fullscreen' : 'Fullscreen';
});

/* =========================
   Game mechanics, loop, AI, rendering, and start/reset functions
   ========================= */
// ... (All original game logic code goes here, unchanged from your original)
// Keep your physics, collision, rendering, punch logic, AI, and requestAnimationFrame loop

</script>
</body>
</html>
