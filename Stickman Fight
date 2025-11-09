<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Duel — 2 Player / Bot / Spring</title>
<style>
:root{
  --bg:#0f1724; --panel:#0b1220; --accent:#7dd3fc; --accent-2:#a78bfa;
  --muted:#94a3b8; --hp-bg:#243244; --hp-lose:#ef4444; --hp-win:#34d399;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
}
html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071028);color:#e6eef8;}
.wrap{display:flex;flex-direction:column;min-height:100vh;align-items:center;justify-content:center;gap:18px;padding:24px;}
.ui{width:min(1000px,96vw);display:flex;justify-content:space-between;align-items:center;gap:16px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);padding:12px;border-radius:14px;box-shadow:0 6px 30px rgba(2,6,23,0.6);}
.player-panel{width:48%;display:flex;align-items:center;gap:12px;}
.hp{flex:1;background:var(--hp-bg);border-radius:10px;height:14px;overflow:hidden;position:relative;}
.hp .bar{height:100%;width:100%;background:linear-gradient(90deg,var(--hp-win),var(--accent));transform-origin:left;transition:width .22s linear, background .22s linear;}
.hp .label{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);font-size:11px;color:var(--muted);font-weight:600;letter-spacing:0.5px;}
.names{font-size:13px;color:var(--muted);font-weight:600;}
.center-controls{display:flex;flex-direction:column;align-items:center;gap:6px;}
button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:var(--muted);cursor:pointer;}
button:hover{border-color:rgba(255,255,255,0.12);color:var(--accent);}
canvas{display:block;width:min(1000px,96vw);height:420px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));box-shadow:0 20px 50px rgba(2,6,23,0.6);}
.instructions{max-width:1000px;color:var(--muted);font-size:13px;text-align:center;margin-top:2px;}
.overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:linear-gradient(0deg, rgba(2,6,23,0.55), rgba(2,6,23,0.55));z-index:20;}
.card{background:var(--panel);padding:20px;border-radius:14px;border:1px solid rgba(255,255,255,0.04);text-align:center;max-width:560px;color:#dbeafe;}
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
  <div class="ui">
    <div class="player-panel">
      <div class="names">Player 1 — WASD</div>
      <div style="width:12px"></div>
      <div class="hp" id="hp1">
        <div class="bar" id="bar1"></div>
        <div class="label" id="label1">100%</div>
      </div>
    </div>
    <div class="center-controls">
      <div style="font-weight:700;color:var(--muted);letter-spacing:1px;">Stickman Duel</div>
      <div style="display:flex;gap:8px">
        <button id="restart">Restart</button>
        <button id="toggleSound">Toggle Sound</button>
      </div>
    </div>
    <div class="player-panel" style="justify-content:flex-end;">
      <div class="hp" id="hp2">
        <div class="bar" id="bar2" style="background:linear-gradient(90deg,var(--accent-2),#f472b6)"></div>
        <div class="label" id="label2">100%</div>
      </div>
      <div style="width:12px"></div>
      <div class="names" style="text-align:right;">Player 2 — Arrows</div>
    </div>
  </div>

  <canvas id="game" width="1200" height="520"></canvas>

  <div class="instructions">
    Controls — Player 1: W A D + S (punch) · Player 2: ↑ ← → + ↓ (punch) · First to deplete opponent wins.
  </div>
</div>

<div id="overlay" class="overlay">
  <div class="card">
    <div class="title">Stickman Duel</div>
    <div class="small">Two-player local or Bot — simple, modern, and responsive.</div>
    <div style="margin-bottom:12px"><strong>Player 1</strong> — WASD &middot; <strong>Player 2</strong> — Arrow keys</div>
    <div style="display:flex;gap:8px;justify-content:center">
      <button id="startBtn">Start 2-Player</button>
      <button id="easyBotBtn">Easy Bot</button>
      <button id="hardBotBtn">Hard Bot</button>
    </div>
    <div class="footer">Hit Restart to reset anytime. Sound is synth-based and safe for web.</div>
  </div>
</div>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
let cw = canvas.width, ch = canvas.height;
const groundY = ch - 80, leftBound = 80, rightBound = cw-80;

let running=false, botMode=false, botDifficulty='easy';
let keys={}, lastTime=performance.now();

// Players
class Player {
  constructor(x,color,facing=1){
    this.x=x; this.y=0; this.vx=0; this.vy=0;
    this.width=36; this.height=70;
    this.onGround=false; this.health=100; this.color=color; this.facing=facing;
    this.punching=false; this.punchTimer=0; this.stun=0; this.combo=0; this.comboTimer=0;
  }
  rect(){ return {x:this.x-this.width/2, y:this.y-this.height, w:this.width, h:this.height}; }
  punchHitbox(){ const range=38, px=this.x+this.facing*(this.width/2+range/2), py=this.y-this.height/2;
  return {x:px-range/2, y:py-16, w:range, h:32}; }
}
const p1=new Player(360,'#7dd3fc',1);
const p2=new Player(840,'#f472b6',-1);

// Spring
const spring = {x:cw/2, y:groundY, width:60, height:20, active:false, timer:0, maxLift:20};

// Health bars
const bar1=document.getElementById('bar1'), bar2=document.getElementById('bar2'),
      label1=document.getElementById('label1'), label2=document.getElementById('label2');
const overlay=document.getElementById('overlay');

// Input
window.addEventListener('keydown', e=>keys[e.code]=true);
window.addEventListener('keyup', e=>keys[e.code]=false);

// Audio
let AudioEnabled = true;
const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
function resumeAudio(){ if(audioCtx.state==='suspended') audioCtx.resume(); }

// Buttons
function setupButtons(){
  document.getElementById('startBtn').addEventListener('click', ()=>startGame(false));
  document.getElementById('easyBotBtn').addEventListener('click', ()=>startGame(true,'easy'));
  document.getElementById('hardBotBtn').addEventListener('click', ()=>startGame(true,'hard'));
  document.getElementById('restart').addEventListener('click', ()=>resetGame());
  document.getElementById('toggleSound').addEventListener('click', ()=>{
    AudioEnabled = !AudioEnabled;
    document.getElementById('toggleSound').textContent = AudioEnabled?'Sound: On':'Sound: Off';
    resumeAudio();
  });
}

// Game control
function startGame(bot=false,difficulty='easy'){
  botMode=bot; botDifficulty=difficulty;
  overlay.style.display='none'; resumeAudio();
  resetPlayers(); lastTime=performance.now(); running=true;
  requestAnimationFrame(update);
}
function resetPlayers(){
  [p1,p2].forEach(p=>{
    p.x=p===p1?360:840; p.y=groundY; p.vx=p.vy=0; p.health=100; p.stun=0; p.punching=false; p.combo=0; p.comboTimer=0;
  });
}
function resetGame(){ running=false; overlay.style.display='flex'; resetPlayers(); updateHealthBars(); }
function updateHealthBars(){
  bar1.style.width=p1.health+'%'; label1.textContent=Math.round(p1.health)+'%';
  bar2.style.width=p2.health+'%'; label2.textContent=Math.round(p2.health)+'%';
}

// Physics & combat
function applyPhysics(p,dt){
  const gravity=1400;
  p.vy+=gravity*dt; p.x+=p.vx*dt; p.y+=p.vy*dt;
  if(p.y>groundY){ p.y=groundY; p.vy=0; p.onGround=true; } else p.onGround=false;
  if(p.x<leftBound){ p.x=leftBound; p.vx=0; }
  if(p.x>rightBound){ p.x=rightBound; p.vx=0; }
  if(p.onGround) p.vx*=0.88;
}

function rectIntersect(a,b){ return !(a.x+a.w<b.x || a.x>b.x+b.w || a.y+a.h<b.y || a.y>b.y+b.h); }
function resolvePunch(attacker,defender){
  if(attacker.punching && attacker.punchTimer>0.06){
    if(rectIntersect(attacker.punchHitbox(), defender.rect()) && defender.stun<=0){
      defender.health=Math.max(0, defender.health-8-Math.floor(Math.random()*6));
      defender.vx=attacker.facing*220; defender.vy=-140; defender.stun=0.26;
      attacker.combo++; attacker.comboTimer=1.0; defender.combo=0; defender.comboTimer=0;
    }
  }
}

// Bot AI
function botAI(controlled,target){
  const dx=target.x-controlled.x, dist=Math.abs(dx);
  if(botDifficulty==='easy'){
    controlled.vx+=Math.sign(dx)*200*0.016;
    if(dist<80 && Math.random()<0.01 && !controlled.punching && controlled.stun<=0){
      controlled.punching=true; controlled.punchTimer=0;
    }
    if(dist<60 && Math.random()<0.01 && controlled.onGround){ controlled.vy=-350; controlled.onGround=false; }
  } else {
    if(dist>100) controlled.vx+=Math.sign(dx)*350*0.016; else controlled.vx*=0.8;
    if(dist<90 && Math.random()<0.03 && !controlled.punching && controlled.stun<=0){ controlled.punching=true; controlled.punchTimer=0; }
    if(target.punching && Math.abs(target.x-controlled.x)<60 && controlled.onGround){ controlled.vy=-400; controlled.onGround=false; }
    if(Math.random()<0.005 && controlled.onGround){ controlled.vy=-420; controlled.onGround=false; }
  }
}

// Update loop
function update(now){
  const dt = Math.min(0.032,(now-lastTime)/1000); lastTime=now;
  if(!running) return;

  // Player input
  if(!botMode){
    if(keys['KeyA']){ p1.vx-=1200*dt; p1.facing=-1; }
    if(keys['KeyD']){ p1.vx+=1200*dt; p1.facing=1; }
    if(keys['KeyW'] && p1.onGround){ p1.vy=-450; p1.onGround=false; }
    if(keys['KeyS'] && !p1.punching && p1.stun<=0){ p1.punching=true; p1.punchTimer=0; }

    if(keys['ArrowLeft']){ p2.vx-=1200*dt; p2.facing=-1; }
    if(keys['ArrowRight']){ p2.vx+=1200*dt; p2.facing=1; }
    if(keys['ArrowUp'] && p2.onGround){ p2.vy=-450; p2.onGround=false; }
    if(keys['ArrowDown'] && !p2.punching && p2.stun<=0){ p2.punching=true; p2.punchTimer=0; }
  } else {
    botAI(p2,p1);
    botAI(p1,p2);
  }

  // Update players
  [p1,p2].forEach(p=>{
    if(p.stun>0){ p.stun=Math.max(0,p.stun-dt); p.vx*=0.96; }
    if(p.punching){ p.punchTimer+=dt; if(p.punchTimer>0.25){ p.punching=false; p.punchTimer=0; } }
    if(p.comboTimer>0){ p.comboTimer=Math.max(0,p.comboTimer-dt); if(p.comboTimer===0) p.combo=0; }

    applyPhysics(p,dt);

    // Spring
    const springRect={x:spring.x-spring.width/2, y:spring.y-spring.height, w:spring.width, h:spring.height};
    if(rectIntersect(springRect,p.rect()) && p.vy>=0){ p.vy=-900; spring.active=true; spring.timer=0.15; }
  });

  render();

  // Win
  if(p1.health<=0 || p2.health<=0){
    running=false;
    setTimeout(()=>alert((p1.health<=0?'Player 2':'Player 1')+' Wins!'),200);
    return;
  }

  requestAnimationFrame(update);
}

// Rendering
function render(){
  ctx.clearRect(0,0,cw,ch);
  ctx.fillStyle='#071128'; ctx.fillRect(0,groundY,cw,ch-groundY);

  // Spring
  const springYDraw = spring.active ? spring.y+spring.maxLift : spring.y;
  ctx.fillStyle='lime'; ctx.fillRect(spring.x-spring.width/2, springYDraw-spring.height, spring.width, spring.height);

  // Players
  [p1,p2].forEach(p=>{
    ctx.fillStyle=p.color;
    ctx.fillRect(p.x-p.width/2,p.y-p.height,p.width,p.height);
    if(p.combo>1){
      ctx.fillStyle='yellow'; ctx.font='20px Inter'; ctx.textAlign='center';
      ctx.fillText('+'+p.combo,p.x,p.y-p.height-35);
    }
  });

  updateHealthBars();
}

// Initialize buttons
setupButtons();
</script>
</body>
</html>
