<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Duel — 2 Player</title>
<style>
:root{
  --bg:#0f1724; --panel:#0b1220; --accent:#7dd3fc; --accent-2: #a78bfa; 
  --muted:#94a3b8; --hp-bg:#243244; --hp-lose:#ef4444; --hp-win:#34d399;
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
}
html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071028);color:#e6eef8;}
.wrap{display:flex;flex-direction:column;min-height:100vh;align-items:center;justify-content:center;gap:18px;padding:24px;}
.ui{width:min(1000px,96vw);display:flex;justify-content:space-between;align-items:center;gap:16px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);padding:12px;border-radius:14px;box-shadow: 0 6px 30px rgba(2,6,23,0.6);}
.player-panel{width:48%;display:flex;align-items:center;gap:12px;}
.hp{flex:1;background:var(--hp-bg);border-radius:10px;height:14px;overflow:hidden;position:relative;}
.hp .bar{height:100%;width:100%;background:linear-gradient(90deg,var(--hp-win),var(--accent));transform-origin:left;transition:width .22s linear, background .22s linear;}
.hp .label{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);font-size:11px;color:var(--muted);font-weight:600;letter-spacing:0.5px;}
.names{font-size:13px;color:var(--muted);font-weight:600;}
.center-controls{display:flex;flex-direction:column;align-items:center;gap:6px;}
button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:var(--muted);cursor:pointer;}
button:hover{border-color:rgba(255,255,255,0.12);color:var(--accent);}
canvas{display:block;width:min(1000px,96vw);height:420px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.01), rgba(255,255,255,0.005));box-shadow: 0 20px 50px rgba(2,6,23,0.6);}
.instructions{max-width:1000px;color:var(--muted);font-size:13px;text-align:center;margin-top:2px;}
.overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:linear-gradient(0deg, rgba(2,6,23,0.55), rgba(2,6,23,0.55));z-index:20;}
.card{background:var(--panel);padding:20px;border-radius:14px;border:1px solid rgba(255,255,255,0.04);text-align:center;max-width:560px;color:#dbeafe;}
.title{font-size:20px;margin-bottom:8px;}
.small{font-size:13px;color:var(--muted);margin-bottom:12px;}
.footer{font-size:12px;color:var(--muted);margin-top:10px;}
@media (max-width:680px){canvas{height:340px}.ui{flex-direction:column;align-items:stretch}.player-panel{width:100%;}}
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
        <button id="toggleSound">Sound: On</button>
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
// Canvas setup
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d', { alpha: false });
let cw = canvas.width, ch = canvas.height;

// Fullscreen & scaling
function resizeCanvas() {
  const aspect = 1200/520;
  let width = window.innerWidth;
  let height = window.innerHeight;
  if(width/height > aspect) width = height*aspect;
  else height = width/aspect;
  canvas.width = 1200;
  canvas.height = 520;
  canvas.style.width = width + 'px';
  canvas.style.height = height + 'px';
}
window.addEventListener('resize', resizeCanvas);
resizeCanvas();

const AudioEnabled = { val:true };
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
function playTone(freq,type='sine',len=0.12,gain=0.12,decay=0.02){
  if(!AudioEnabled.val) return;
  const now=audioCtx.currentTime;
  const o=audioCtx.createOscillator();
  const g=audioCtx.createGain();
  o.type=type;o.frequency.setValueAtTime(freq,now);
  g.gain.setValueAtTime(gain,now); g.gain.exponentialRampToValueAtTime(0.0001, now+len+decay);
  o.connect(g); g.connect(audioCtx.destination); o.start(now); o.stop(now+len+decay);
}
function playPunchSound(){playTone(600,'square',0.08,0.09,0.02); setTimeout(()=>playTone(420,'sawtooth',0.12,0.06,0.03),10);}
function playHitSound(){playTone(160,'sine',0.18,0.15,0.03); setTimeout(()=>playTone(220,'triangle',0.1,0.08,0.02),30);}
function playWinSound(){playTone(880,'sine',0.28,0.14,0.04); setTimeout(()=>playTone(1100,'sine',0.22,0.12,0.03),200);}
function backgroundDrone(){if(!AudioEnabled.val) return; const o=audioCtx.createOscillator(); const g=audioCtx.createGain(); o.type='sine'; o.frequency.value=55; g.gain.value=0.02; o.connect(g); g.connect(audioCtx.destination); o.start(); return {osc:o,gain:g};}
let bgNode=null;

// Player class
class Player{
  constructor(x,color,facing=1){this.x=x; this.y=0; this.vx=0; this.vy=0; this.width=36; this.height=70; this.onGround=false; this.health=100; this.color=color; this.facing=facing; this.punching=false; this.punchTimer=0; this.stun=0;}
  rect(){return {x:this.x-this.width/2,y:this.y-this.height,w:this.width,h:this.height};}
  punchHitbox(){const range=38; const px=this.x+this.facing*(this.width/2+range/2); const py=this.y-this.height/2; return {x:px-range/2,y:py-16,w:range,h:32};}
}

// World
const groundY = 520-80;
const leftBound=80, rightBound=1200-80;
const p1=new Player(360,'#7dd3fc',1), p2=new Player(840,'#f472b6',-1);
let keys={}, lastTime=performance.now(), running=false, demoMode=false;

// Input
window.addEventListener('keydown',e=>keys[e.code]=true);
window.addEventListener('keyup',e=>keys[e.code]=false);

// UI
const bar1=document.getElementById('bar1'), bar2=document.getElementById('bar2'), label1=document.getElementById('label1'), label2=document.getElementById('label2'), overlay=document.getElementById('overlay');
document.getElementById('startBtn').addEventListener('click',()=>{overlay.style.display='none'; startMatch();});
document.getElementById('demoBtn').addEventListener('click',()=>{overlay.style.display='none'; demoMode=true; startMatch();});
document.getElementById('restart').addEventListener('click',()=>resetMatch());
document.getElementById('toggleSound').addEventListener('click',()=>{
  AudioEnabled.val=!AudioEnabled.val;
  if(!AudioEnabled.val && bgNode){bgNode.osc.stop(); bgNode=null;}
  if(AudioEnabled.val && running && !bgNode) bgNode=backgroundDrone();
  document.getElementById('toggleSound').textContent = AudioEnabled.val?'Sound: On':'Sound: Off';
});

// Fullscreen button
const fsBtn = document.getElementById('fullscreenBtn');
fsBtn.addEventListener('click',()=>{
  if(!document.fullscreenElement) canvas.requestFullscreen();
  else document.exitFullscreen();
});
document.addEventListener('fullscreenchange',()=>{ fsBtn.textContent = document.fullscreenElement?'Exit Fullscreen':'Fullscreen';});

// Physics and mechanics
function applyPhysics(dt,p){
  const g=1400; p.vy+=g*dt; p.x+=p.vx*dt; p.y+=p.vy*dt;
  if(p.y>groundY){p.y=groundY;p.vy=0;p.onGround=true;} else p.onGround=false;
  if(p.x<leftBound){p.x=leftBound;p.vx=0;} if(p.x>rightBound){p.x=rightBound;p.vx=0;}
  if(p.onGround && Math.abs(p.vx)<5) p.vx=0; if(p.onGround) p.vx*=0.88;
}
function rectIntersect(a,b){return !(a.x+a.w<b.x||a.x>b.x+b.w||a.y+a.h<b.y||a.y>b.y+b.h);}
function resolvePunch(attacker,defender){
  if(attacker.punching && attacker.punchTimer>0.06){
    const aHit=attacker.punchHitbox(), dRect=defender.rect();
    if(rectIntersect(aHit,dRect) && defender.stun<=0){
      defender.health=Math.max(0,defender.health-8-Math.floor(Math.random()*6));
      defender.vx=attacker.facing*220; defender.vy=-140; defender.stun=0.26; playHitSound(); return true;
    }
  }
  return false;
}

// AI
function simpleAI(c,t){
  const dx=t.x-c.x, dist=Math.abs(dx);
  if(dist>140) c.vx+=Math.sign(dx)*200*0.02;
  else if(Math.random()<0.02) c.punching=true;
  if(dist<60 && Math.random()<0.02 && c.onGround){c.vy=-420; c.onGround=false;}
}

// Game loop
function update(now){
  const dt=Math.min(0.032,(now-lastTime)/1000); lastTime=now;
  if(!running) return;

  if(!demoMode){
    if(keys['KeyA']){p1.vx-=1200*dt;p1.facing=-1;} if(keys['KeyD']){p1.vx+=1200*dt;p1.facing=1;}
    if(keys['KeyW'] && p1.onGround){p1.vy=-450;p1.onGround=false;}
    if(keys['KeyS'] && !p1.punching && p1.stun<=0){p1.punching=true; p1.punchTimer=0; playPunchSound();}
    if(keys['ArrowLeft']){p2.vx-=1200*dt;p2.facing=-1;} if(keys['ArrowRight']){p2.vx+=1200*dt;p2.facing=1;}
    if(keys['ArrowUp'] && p2.onGround){p2.vy=-450;p2.onGround=false;}
    if(keys['ArrowDown'] && !p2.punching && p2.stun<=0){p2.punching=true; p2.punchTimer=0; playPunchSound();}
  } else {
    simpleAI(p1,p2); if(Math.random()<0.004){p1.punching=true;p1.punchTimer=0;playPunchSound();}
    simpleAI(p2,p1); if(Math.random()<0.004){p2.punching=true;p2.punchTimer=0;playPunchSound();}
  }

  updatePlayer(p1,dt,p2); updatePlayer(p2,dt,p1); render();
  if(p1.health<=0 || p2.health<=0){running=false; setTimeout(()=>showWin(p1.health<=0?'Player 2':'Player 1'),260); if(AudioEnabled.val)playWinSound(); if(bgNode){try{bgNode.osc.stop();}catch(e){}; bgNode=null;} return;}
  requestAnimationFrame(update);
}
function updatePlayer(p,dt,op){
  if(p.stun>0){p.stun=Math.max(0,p.stun-dt); p.vx*=0.96;}
  if(p.punching){p.punchTimer+=dt; if(p.punchTimer>0.25){p.punching=false;p.punchTimer=0;}}
  if(p.punching && p.punchTimer>0.05) resolvePunch(p,op);
  const maxSpeed=420; if(p.vx>maxSpeed)p.vx=maxSpeed; if(p.vx<-maxSpeed)p.vx=-maxSpeed;
  applyPhysics(dt,p);
}

// Render
let shakeAmount=0;
function render(){
  const shakeX=(Math.random()*2-1)*shakeAmount; const shakeY=(Math.random()*2-1)*shakeAmount; shakeAmount*=0.92;
  ctx.save(); ctx.translate(shakeX,shakeY);
  ctx.fillStyle='#0b1220'; ctx.fillRect(0,0,cw,ch);
  ctx.fillStyle='#071128'; ctx.fillRect(0,groundY,cw,ch-groundY);
  ctx.strokeStyle='rgba(255,255,255,0.02)'; ctx.lineWidth=1; ctx.beginPath();
  ctx.moveTo(leftBound,groundY); ctx.lineTo(leftBound,groundY-160); ctx.moveTo(rightBound,groundY); ctx.lineTo(rightBound,groundY-160); ctx.stroke();
  drawStickman(p1); drawStickman(p2); ctx.restore();
  bar1.style.width=p1.health+'%'; label1.textContent=Math.round(p1.health)+'%';
  bar2.style.width=p2.health+'%'; label2.textContent=Math.round(p2.health)+'%';
  bar1.style.background=p1.health<35?'linear-gradient(90deg,#f97316,#ef4444)':'linear-gradient(90deg,var(--hp-win),var(--accent))';
  bar2.style.background=p2.health<35?'linear-gradient(90deg,#f97316,#ef4444)':'linear-gradient(90deg,var(--accent-2),#f472b6)';
}
function drawStickman(p){
  const r=p.rect();
  ctx.fillStyle='rgba(0,0,0,0.16)'; ctx.beginPath(); ctx.ellipse(p.x,groundY+6,26,8,0,0,Math.PI*2); ctx.fill();
  const headX=p.x, headY=p.y-p.height+14;
  ctx.strokeStyle=p.color; ctx.lineWidth=4; ctx.beginPath(); ctx.arc(headX,headY,12,0,Math.PI*2); ctx.stroke();
  const torsoTopX=headX, torsoTopY=headY+14, torsoBottomX=headX, torsoBottomY=p.y-18;
  ctx.beginPath(); ctx.moveTo(torsoTopX,torsoTopY); ctx.lineTo(torsoBottomX,torsoBottomY); ctx.stroke();
  const armLen=28, armAngle=p.punching?0.1*p.facing:-0.6;
  ctx.beginPath(); ctx.moveTo(torsoTopX,torsoTopY+6); ctx.lineTo(torsoTopX-18,torsoTopY+22); ctx.stroke();
  const arX=torsoTopX+Math.cos(armAngle)*armLen*p.facing, arY=torsoTopY+12+Math.sin(armAngle)*armLen;
  ctx.beginPath(); ctx.moveTo(torsoTopX,torsoTopY+6); ctx.lineTo(arX,arY); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(torsoBottomX,torsoBottomY); ctx.lineTo(torsoBottomX-14,p.y); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(torsoBottomX,torsoBottomY); ctx.lineTo(torsoBottomX+18,p.y); ctx.stroke();
  if(p.punching){const hb=p.punchHitbox(); ctx.fillStyle='rgba(255,255,255,0.04)'; ctx.fillRect(hb.x,hb.y,hb.w,hb.h);}
  ctx.fillStyle='rgba(255,255,255,0.06)'; ctx.fillRect(p.x-36,p.y-p.height-28,72,18);
  ctx.fillStyle='#cbd5e1'; ctx.font='11px Inter, Arial'; ctx.textAlign='center';
  ctx.fillText(p===p1?'P1':'P2',p.x,p.y-p.height-16);
}

// Start / reset / win
function startMatch(){
  p1.x=360; p1.y=groundY; p1.vx=p1.vy=0; p1.health=100; p1.stun=0; p1.punching=false;
  p2.x=840; p2.y=groundY; p2.vx=p2.vy=0; p2.health=100; p2.stun=0; p2.punching=false;
  lastTime=performance.now(); running=true; if(AudioEnabled.val&&!bgNode) bgNode=backgroundDrone();
  requestAnimationFrame(update);
}
function resetMatch(){demoMode=false; overlay.style.display='none'; startMatch();}
function showWin(who){
  overlay.style.display='flex'; const card=overlay.querySelector('.card');
  card.innerHTML=`<div class="title">${who} Wins!</div>
  <div class="small">Nice fight — press Restart to play again.</div>
  <div style="margin:12px 0;display:flex;gap:10px;justify-content:center">
    <button id="restart2">Restart</button>
    <button id="menu">Menu</button>
  </div>
  <div class="footer">Built with WebAudio and Canvas — no external assets.</div>`;
  document.getElementById('restart2').addEventListener('click',()=>{overlay.style.display='none'; startMatch();});
  document.getElementById('menu').addEventListener('click',()=>{location.reload();});
}

document.addEventListener('visibilitychange',()=>{if(document.hidden && bgNode){try{bgNode.osc.stop();}catch(e){} bgNode=null;}});
window.onload=()=>{}; // overlay already shows
</script>
</body>
</html>
