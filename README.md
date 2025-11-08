<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Duel — 2 Player</title>
<style>
:root{
  --bg:#0f1724; --panel:#0b1220; --accent:#7dd3fc; --accent-2:#a78bfa; --muted:#94a3b8;
  --hp-bg:#243244; --hp-win:#34d399;
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
canvas{display:block;width:min(1000px,96vw);height:520px;border-radius:12px;background:#000;box-shadow:0 20px 50px rgba(2,6,23,0.6);}
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
      <div class="names">Player 1 — WASD + Shift Kick</div>
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
      <div class="names" style="text-align:right;">Player 2 — Arrows + Shift Kick</div>
    </div>
  </div>
  <canvas id="game" width="1200" height="520"></canvas>
  <div class="instructions">Choose an arena and fight! Player 1: W A D + S + Shift Kick · Player 2: ↑ ← → + ↓ + Shift Kick</div>
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
const canvas=document.getElementById('game'),ctx=canvas.getContext('2d',{alpha:false});
let cw=canvas.width,ch=canvas.height;
const groundY=ch-80,leftBound=80,rightBound=cw-80;
function resize(){cw=canvas.width=1200;ch=canvas.height=520;}resize();

const arenas={
  grasslands:{sky:"#bde0a8",ground:"#6aa84f",effect:"grass"},
  sahara:{sky:"#f9e79f",ground:"#f1c40f",effect:"dust"},
  rainyforest:{sky:"#6ab04c",ground:"#145a32",effect:"rain"},
  dryland:{sky:"#f7e7ce",ground:"#a67b5b",effect:"hill"}
};
let currentArena="grasslands";
let particles=[],clouds=[],trees=[],desertBalls=[],pond={x:500,y:groundY-20,width:200,height:40};

class Player{
  constructor(x,color,facing=1){
    this.x=x;this.y=0;this.vx=0;this.vy=0;this.width=36;this.height=70;
    this.onGround=false;this.health=100;this.color=color;this.facing=facing;
    this.punching=false;this.punchTimer=0;this.stun=0;this.inPondTime=0;this.bubbles=[];
  }
  rect(){return {x:this.x-this.width/2,y:this.y-this.height,w:this.width,h:this.height};}
  punchHitbox(){const r=38,px=this.x+this.facing*(this.width/2+r/2),py=this.y-this.height/2;return {x:px-r/2,y:py-16,w:r,h:32};}
}

const p1=new Player(360,'#7dd3fc',1);
const p2=new Player(840,'#f472b6',-1);
let keys={},running=false;
window.addEventListener('keydown',e=>keys[e.code]=true);
window.addEventListener('keyup',e=>keys[e.code]=false);

/* ======= Audio ======= */
const AudioEnabled={val:true};
const audioCtx=new (window.AudioContext||window.webkitAudioContext)();
function playTone(freq,type='sine',len=0.12,gain=0.12){
  if(!AudioEnabled.val)return;
  const o=audioCtx.createOscillator(),v=audioCtx.createGain(),t=audioCtx.currentTime;
  o.type=type;o.frequency.value=freq;v.gain.value=gain;o.connect(v);v.connect(audioCtx.destination);
  o.start(t);v.gain.exponentialRampToValueAtTime(0.0001,t+len);o.stop(t+len);
}
function playPunchSound(){playTone(600,'square',0.08,0.09);}
function playHitSound(){playTone(180,'sine',0.18,0.15);}
function playWinSound(){playTone(880,'triangle',0.28,0.14);}
function playKickSound(){playTone(480,'square',0.08,0.08);}

/* ======= Kick cooldown ======= */
let lastKickP1=0,lastKickP2=0,kickCooldown=0.2;

/* ======= Physics ======= */
function applyPhysics(dt,p){
  const g=1400;p.vy+=g*dt;p.x+=p.vx*dt;p.y+=p.vy*dt;
  if(p.y>groundY){p.y=groundY;p.vy=0;p.onGround=true;}else p.onGround=false;
  if(p.x<leftBound)p.x=leftBound;if(p.x>rightBound)p.x=rightBound;
  // Dryland hill
  if(currentArena==='dryland'){
    let hillX=cw/2,hWidth=200,hHeight=50;
    if(p.x>hillX-hWidth/2 && p.x<hillX+hWidth/2){
      let hillTop=groundY-hHeight*Math.sin((p.x-(hillX-hWidth/2))/hWidth*Math.PI);
      if(p.y>hillTop)p.y=hillTop;
    }
  }
  if(p.onGround)p.vx*=0.88;
}

/* ======= Combat ======= */
function rectInt(a,b){return !(a.x+a.w<b.x || a.x>b.x+b.w || a.y+a.h<b.y || a.y>b.y+b.h);}
function resolvePunch(a,b){if(a.punching && a.punchTimer>0.06){const h=a.punchHitbox(),r=b.rect();if(rectInt(h,r)&&b.stun<=0){b.health=Math.max(0,b.health-10);b.vx=a.facing*220;b.vy=-140;b.stun=0.26;playHitSound();}}}
function resolveKick(a,b){const now=performance.now()/1000;if(a===p1){if(now-lastKickP1<kickCooldown)return;lastKickP1=now;}else{if(now-lastKickP2<kickCooldown)return;lastKickP2=now;};const r=b.rect();const hit={x:a.x+a.facing*30,y:a.y-35,w:30,h:20};if(rectInt(hit,r)){b.health=Math.max(0,b.health-10);b.vx=a.facing*220;b.vy=-140;playKickSound();}}

/* ======= Pond Damage/Bubbles ======= */
function checkPondDamage(p,dt){
  if(currentArena!=='grasslands')return;
  if(p.x>pond.x && p.x<pond.x+pond.width && p.y>pond.y-10){p.inPondTime+=dt;
    if(p.bubbles.length<3 && Math.floor(p.inPondTime)>p.bubbles.length)p.bubbles.push({time:0});
    if(p.inPondTime>3)p.health-=dt*10;
  }else{p.inPondTime=0;p.bubbles=[];}
}

/* ======= Render ======= */
const bar1=document.getElementById('bar1'),bar2=document.getElementById('bar2');
const label1=document.getElementById('label1'),label2=document.getElementById('label2');
function updateUI(){bar1.style.width=p1.health+'%';bar2.style.width=p2.health+'%';label1.textContent=Math.round(p1.health)+'%';label2.textContent=Math.round(p2.health)+'%';}
function drawStickman(p){
  ctx.strokeStyle=p.color;ctx.lineWidth=3;
  const headY=p.y-p.height+14;
  // Head
  ctx.beginPath();ctx.arc(p.x,headY,12,0,Math.PI*2);ctx.stroke();
  // Body
  ctx.beginPath();ctx.moveTo(p.x,headY+14);ctx.lineTo(p.x,p.y-18);ctx.stroke();
  // Legs
  ctx.beginPath();ctx.moveTo(p.x,p.y-18);ctx.lineTo(p.x-14,p.y);ctx.moveTo(p.x,p.y-18);ctx.lineTo(p.x+18,p.y);ctx.stroke();
  // Arms swing
  let armSwing=Math.sin(Date.now()/200)*10;
  const armLen=28; const aX=p.x+p.facing*armLen; ctx.beginPath();
  ctx.moveTo(p.x,headY+16);ctx.lineTo(aX,headY+28+armSwing);ctx.stroke();
  // Pond bubbles
  p.bubbles.forEach((b,i)=>{ctx.fillStyle='rgba(173,216,230,0.8)';ctx.beginPath();
    ctx.arc(p.x,p.y-p.height-10-i*12,6,0,Math.PI*2);ctx.fill();
  });
}

/* ======= Input ======= */
function handleInput(p,dt,left,right,jump,punch,kick){
  if(p.stun>0)return;
  if(keys[left]){p.vx-=1200*dt;p.facing=-1;}
  if(keys[right]){p.vx+=1200*dt;p.facing=1;}
  if(keys[jump]&&p.onGround){p.vy=-450;p.onGround=false;}
  if(keys[punch]&&!p.punching){p.punching=true;p.punchTimer=0;playPunchSound();}
  if(keys[kick])resolveKick(p,p===p1?p2:p1);
}

/* ======= Update ======= */
function update(p,dt,o){if(p.stun>0)p.stun=Math.max(0,p.stun-dt);if(p.punching){p.punchTimer+=dt;if(p.punchTimer>0.25){p.punching=false;p.punchTimer=0;}}resolvePunch(p,o);applyPhysics(dt,p);checkPondDamage(p,dt);}

/* ======= Render loop ======= */
function render(dt){
  ctx.fillStyle=arenas[currentArena].sky;ctx.fillRect(0,0,cw,ch);
  ctx.fillStyle=arenas[currentArena].ground;ctx.fillRect(0,groundY,cw,ch-groundY);
  // Grasslands pond
  if(currentArena==='grasslands'){
    ctx.fillStyle='rgba(50,100,255,0.6)';ctx.beginPath();
    ctx.ellipse(pond.x+pond.width/2,pond.y,pond.width/2,pond.height/2,0,0,Math.PI*2);ctx.fill();
  }
  // Dryland hill
  if(currentArena==='dryland'){
    ctx.fillStyle='#a67b5b';ctx.beginPath();
    const hillX=cw/2,hWidth=200,hHeight=50;
    for(let i=0;i<=hWidth;i++){
      let x=hillX-hWidth/2+i,y=groundY-hHeight*Math.sin(i/hWidth*Math.PI);
      if(i===0)ctx.moveTo(x,y);else ctx.lineTo(x,y);
    }ctx.lineTo(cw/2+hWidth/2,groundY);ctx.lineTo(cw/2-hWidth/2,groundY);ctx.fill();
  }
  drawStickman(p1);drawStickman(p2);updateUI();
}

/* ======= Game loop ======= */
let last=performance.now();
function loop(now){
  if(!running)return;
  const dt=Math.min(0.032,(now-last)/1000);last=now;
  handleInput(p1,dt,'KeyA','KeyD','KeyW','KeyS','ShiftLeft');
  handleInput(p2,dt,'ArrowLeft','ArrowRight','ArrowUp','ArrowDown','ShiftRight');
  update(p1,dt,p2);update(p2,dt,p1);render(dt);
  if(p1.health<=0||p2.health<=0){running=false;playWinSound();setTimeout(()=>showWin(p1.health<=0?'Player 2':'Player 1'),400);}
  requestAnimationFrame(loop);
}

/* ======= Start/Restart ======= */
function startGame(){
  Object.assign(p1,{x:360,y:groundY,vx:0,vy:0,health:100,stun:0,punching:false,inPondTime:0,bubbles:[]});
  Object.assign(p2,{x:840,y:groundY,vx:0,vy:0,health:100,stun:0,punching:false,inPondTime:0,bubbles:[]});
  running=true;last=performance.now();requestAnimationFrame(loop);
}
document.getElementById('arenaGrasslands').onclick=()=>{currentArena='grasslands';overlay.style.display='none';startGame();}
document.getElementById('arenaSahara').onclick=()=>{currentArena='sahara';overlay.style.display='none';startGame();}
document.getElementById('arenaRainyForest').onclick=()=>{currentArena='rainyforest';overlay.style.display='none';startGame();}
document.getElementById('arenaDryland').onclick=()=>{currentArena='dryland';overlay.style.display='none';startGame();}
document.getElementById('restart').onclick=()=>{overlay.style.display='flex';}

/* ======= Win Screen ======= */
function showWin(w){overlay.style.display='flex';overlay.querySelector('.card').innerHTML=
`<div class="title">${w} Wins!</div>
<button onclick="location.reload()">Play Again</button>
<div class="footer">All by <strong>aboodicoder</strong></div>`;}
</script
