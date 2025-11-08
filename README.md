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
  --accent-2:#a78bfa;
  --muted:#94a3b8;
  --hp-bg:#243244;
  --hp-win:#34d399;
  font-family: Inter, system-ui, sans-serif;
}
html,body{height:100%;margin:0;background:var(--bg);color:#e6eef8;}
.wrap{display:flex;flex-direction:column;align-items:center;justify-content:center;min-height:100vh;gap:18px;padding:24px;}
.ui{width:min(1000px,96vw);display:flex;justify-content:space-between;align-items:center;gap:16px;background:rgba(255,255,255,0.02);border:1px solid rgba(255,255,255,0.03);padding:12px;border-radius:14px;}
.player-panel{width:48%;display:flex;align-items:center;gap:12px;}
.hp{flex:1;background:var(--hp-bg);border-radius:10px;height:14px;overflow:hidden;position:relative;}
.hp .bar{height:100%;width:100%;background:linear-gradient(90deg,var(--hp-win),var(--accent));transition:width .22s linear;}
.hp .label{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);font-size:11px;color:var(--muted);}
.names{font-size:13px;color:var(--muted);font-weight:600;}
.center-controls{display:flex;flex-direction:column;align-items:center;gap:6px;}
button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:10px;color:var(--muted);cursor:pointer;}
button:hover{border-color:rgba(255,255,255,0.12);color:var(--accent);}
canvas{display:block;width:min(1000px,96vw);height:420px;border-radius:12px;background:#000;box-shadow:0 20px 50px rgba(2,6,23,0.6);}
.instructions{max-width:1000px;color:var(--muted);font-size:13px;text-align:center;}
.overlay{position:fixed;inset:0;display:flex;align-items:center;justify-content:center;background:rgba(2,6,23,0.6);z-index:20;}
.card{background:var(--panel);padding:20px;border-radius:14px;border:1px solid rgba(255,255,255,0.04);text-align:center;max-width:560px;color:#dbeafe;}
.title{font-size:20px;margin-bottom:8px;}
.small{font-size:13px;color:var(--muted);margin-bottom:12px;}
.footer{font-size:12px;color:var(--muted);margin-top:10px;}
</style>
</head>
<body>
<div class="wrap">
  <div class="ui">
    <div class="player-panel">
      <div class="names">Player 1 — WASD</div>
      <div class="hp" id="hp1"><div class="bar" id="bar1"></div><div class="label" id="label1">100%</div></div>
    </div>
    <div class="center-controls">
      <div style="font-weight:700; color:var(--muted);">Stickman Duel</div>
      <div style="display:flex;gap:8px;">
        <button id="restart">Restart</button>
        <button id="toggleSound">Toggle Sound</button>
      </div>
    </div>
    <div class="player-panel" style="justify-content:flex-end;">
      <div class="hp" id="hp2"><div class="bar" id="bar2" style="background:linear-gradient(90deg,var(--accent-2),#f472b6)"></div><div class="label" id="label2">100%</div></div>
      <div class="names" style="text-align:right;">Player 2 — Arrows</div>
    </div>
  </div>

  <canvas id="game" width="1200" height="520"></canvas>
  <div class="instructions">Choose an arena and fight! Player 1: W A D + S · Player 2: ↑ ← → + ↓</div>
</div>

<div id="overlay" class="overlay">
  <div class="card">
    <div class="title">Stickman Duel</div>
    <div class="small">Choose your battleground</div>
    <div id="arenaSelect" style="display:flex;flex-wrap:wrap;gap:8px;justify-content:center;margin-bottom:10px;">
      <button id="arenaGrasslands">🌿 Grasslands</button>
      <button id="arenaSahara">🏜️ Sahara</button>
      <button id="arenaRainyForest">🌧️ Rainy Forest</button>
      <button id="arenaDryland">🏞️ Dryland</button>
    </div>
    <div style="display:flex;gap:8px;justify-content:center;">
      <button id="demoBtn">Demo (AI vs AI)</button>
    </div>
    <div class="footer">All by <strong>aboodicoder</strong></div>
  </div>
</div>

<script>
/* ======= Setup ======= */
const canvas = document.getElementById('game'), ctx = canvas.getContext('2d',{alpha:false});
let cw = canvas.width, ch = canvas.height;
const groundY = ch - 80, leftBound=80, rightBound=cw-80;
function resize(){cw=canvas.width=1200; ch=canvas.height=520;} resize();

const arenas = {
  grasslands: {bg:"#89c15b",sky:"#bde0a8",ground:"#6aa84f", effect:"grass"},
  sahara: {bg:"#f4d03f",sky:"#f9e79f",ground:"#f1c40f", effect:"dust"},
  rainyforest: {bg:"#355e3b",sky:"#6ab04c",ground:"#145a32", effect:"rain"},
  dryland: {bg:"#d2b48c",sky:"#f7e7ce",ground:"#a67b5b", effect:"shimmer"}
};
let currentArena = "grasslands";
let particles=[];

/* ======= Audio ======= */
const AudioEnabled = {val:true};
const audioCtx = new (window.AudioContext||window.webkitAudioContext)();
function playTone(freq,type='sine',len=0.12,gain=0.12){if(!AudioEnabled.val)return; const o=audioCtx.createOscillator(),v=audioCtx.createGain(),t=audioCtx.currentTime; o.type=type;o.frequency.value=freq;v.gain.value=gain;o.connect(v);v.connect(audioCtx.destination);o.start(t);v.gain.exponentialRampToValueAtTime(0.0001,t+len);o.stop(t+len);}
function playPunchSound(){playTone(600,'square',0.08,0.09);}
function playHitSound(){playTone(180,'sine',0.18,0.15);}
function playWinSound(){playTone(880,'triangle',0.28,0.14);}

/* ======= Player ======= */
class Player{
  constructor(x,color,facing=1){this.x=x;this.y=0;this.vx=0;this.vy=0;this.width=36;this.height=70;this.onGround=false;this.health=100;this.color=color;this.facing=facing;this.punching=false;this.punchTimer=0;this.stun=0;}
  rect(){return {x:this.x-this.width/2,y:this.y-this.height,w:this.width,h:this.height};}
  punchHitbox(){const r=38,px=this.x+this.facing*(this.width/2+r/2),py=this.y-this.height/2;return {x:px-r/2,y:py-16,w:r,h:32};}
}

const p1=new Player(360,'#7dd3fc',1);
const p2=new Player(840,'#f472b6',-1);
let keys={},running=false,demoMode=false;

window.addEventListener('keydown', e=>keys[e.code]=true);
window.addEventListener('keyup', e=>keys[e.code]=false);

/* ======= Physics ======= */
function applyPhysics(dt,p){const g=1400;p.vy+=g*dt;p.x+=p.vx*dt;p.y+=p.vy*dt;if(p.y>groundY){p.y=groundY;p.vy=0;p.onGround=true;}else p.onGround=false;if(p.x<leftBound)p.x=leftBound;if(p.x>rightBound)p.x=rightBound;if(p.onGround)p.vx*=0.88;}
function rectInt(a,b){return !(a.x+a.w<b.x || a.x>b.x+b.w || a.y+a.h<b.y || a.y>b.y+b.h);}
function resolvePunch(a,b){if(a.punching && a.punchTimer>0.06){const h=a.punchHitbox(),r=b.rect();if(rectInt(h,r)&&b.stun<=0){b.health=Math.max(0,b.health-10);b.vx=a.facing*220;b.vy=-140;b.stun=0.26;playHitSound();}}}

/* ======= Render ======= */
function drawStickman(p){
  ctx.strokeStyle=p.color; ctx.lineWidth=3;
  const headY=p.y-p.height+14;
  ctx.beginPath(); ctx.arc(p.x,headY,12,0,Math.PI*2); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(p.x,headY+14); ctx.lineTo(p.x,p.y-18); ctx.stroke();
  ctx.beginPath(); ctx.moveTo(p.x,p.y-18); ctx.lineTo(p.x-14,p.y); ctx.moveTo(p.x,p.y-18); ctx.lineTo(p.x+18,p.y); ctx.stroke();
  const armLen=28; const aX=p.x+p.facing*armLen;
  ctx.beginPath(); ctx.moveTo(p.x,headY+16); ctx.lineTo(aX,headY+28); ctx.stroke();
}
function updateUI(){
  bar1.style.width = p1.health+'%'; label1.textContent=Math.round(p1.health)+'%';
  bar2.style.width = p2.health+'%'; label2.textContent=Math.round(p2.health)+'%';
}

/* ======= Particles ======= */
function clearParticles(){particles=[];}
function spawnParticles(){
  if(!running) return;
  const effect = arenas[currentArena].effect;
  if(effect==="rain"){for(let i=0;i<4;i++){particles.push({x:Math.random()*cw,y:-10,vy:400+Math.random()*200,type:"rain"});}}
  else if(effect==="dust"){for(let i=0;i<2;i++){particles.push({x:Math.random()*cw,y:groundY-10,vy:-20-Math.random()*40,type:"dust"});}}
  else if(effect==="shimmer"){for(let i=0;i<1;i++){particles.push({x:Math.random()*cw,y:groundY-50,vy:-10,type:"shimmer",alpha:Math.random()});}}
  else if(effect==="grass"){for(let i=0;i<2;i++){particles.push({x:Math.random()*cw,y:groundY-2,vy:0,type:"grass",alpha:0.3+Math.random()*0.4});}}
}
function drawParticles(dt){
  particles.forEach(p=>{
    if(p.type==="rain"){ctx.strokeStyle="rgba(173,216,230,0.6)";ctx.beginPath();ctx.moveTo(p.x,p.y);ctx.lineTo(p.x,p.y+10);ctx.stroke();p.y+=p.vy*dt;}
    else if(p.type==="dust"){ctx.fillStyle="rgba(237,201,175,0.4)";ctx.beginPath();ctx.arc(p.x,p.y,3,0,Math.PI*2);ctx.fill();p.y+=p.vy*dt;p.x+=Math.random()*1-0.5;}
    else if(p.type==="shimmer"){ctx.fillStyle=`rgba(255,255,255,${p.alpha})`;ctx.beginPath();ctx.arc(p.x,p.y,2,0,Math.PI*2);ctx.fill();p.y+=p.vy*dt;p.alpha-=dt*0.2;}
    else if(p.type==="grass"){ctx.fillStyle=`rgba(0,100,0,${p.alpha})`;ctx.fillRect(p.x,p.y,2,6);}
  });
  particles = particles.filter(p=> (p.type==="rain"?p.y<ch+10: p.type==="dust"?p.y>groundY-60: p.type==="shimmer"?p.alpha>0:true));
}

/* ======= Main Loop ======= */
let last = performance.now();
function loop(now){
  if(!running) return;
  const dt = Math.min(0.032,(now-last)/1000); last=now;
  handleInput(p1,dt,'KeyA','KeyD','KeyW','KeyS');
  handleInput(p2,dt,'ArrowLeft','ArrowRight','ArrowUp','ArrowDown');
  update(p1,dt,p2); update(p2,dt,p1);
  render(dt);
  if(p1.health<=0||p2.health<=0){running=false;playWinSound();setTimeout(()=>showWin(p1.health<=0?'Player 2':'Player 1'),400);}
  requestAnimationFrame(loop);
}
function handleInput(p,dt,left,right,jump,punch){
  if(p.stun>0) return;
  if(keys[left]){p.vx-=1200*dt;p.facing=-1;}
  if(keys[right]){p.vx+=1200*dt;p.facing=1;}
  if(keys[jump]&&p.onGround){p.vy=-450;p.onGround=false;}
  if(keys[punch]&&!p.punching){p.punching=true;p.punchTimer=0;playPunchSound();}
}
function update(p,dt,o){if(p.stun>0)p.stun=Math.max(0,p.stun-dt);if(p.punching){p.punchTimer+=dt;if(p.punchTimer>0.25){p.punching=false;p.punchTimer=0;}}resolvePunch(p,o);applyPhysics(dt,p);}
function render(dt){
  const env = arenas[currentArena];
  ctx.fillStyle=env.sky; ctx.fillRect(0,0,cw,ch);
  ctx.fillStyle=env.ground; ctx.fillRect(0,groundY,cw,ch-groundY);
  drawParticles(dt); drawStickman(p1); drawStickman(p2); updateUI();
}

/* ======= Arena Selection ======= */
document.getElementById('arenaGrasslands').onclick=()=>selectArena('grasslands');
document.getElementById('arenaSahara').onclick=()=>selectArena('sahara');
document.getElementById('arenaRainyForest').onclick=()=>selectArena('rainyforest');
document.getElementById('arenaDryland').onclick=()=>selectArena('dryland');
document.getElementById('demoBtn').onclick=()=>{demoMode=true;overlay.style.display='none';startGame();};
function selectArena(a){currentArena=a;clearParticles();overlay.style.display='none';startGame();}

/* ======= Start/Reset ======= */
function startGame(){
  Object.assign(p1,{x:360,y:groundY,vx:0,vy:0,health:100,stun:0,punching:false});
  Object.assign(p2,{x:840,y:groundY,vx:0,vy:0,health:100,stun:0,punching:false});
  running=true; last=performance.now();
  requestAnimationFrame(loop);
}
document.getElementById('restart').onclick=()=>{overlay.style.display='flex';}

/* ======= Win Screen ======= */
function showWin(w){overlay.style.display='flex';overlay.querySelector('.card').innerHTML=
`<div class="title">${w} Wins!</div>
<button onclick="location.reload()">Play Again</button>
<div class="footer">All by <strong>aboodicoder</strong></div>`;}

/* ======= Particle Spawn Interval ======= */
setInterval(spawnParticles,120);
</script>
</body>
</html>
