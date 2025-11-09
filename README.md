<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Stickman Duel — 2 Player / Bot / Spring</title>
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
      <div style="font-weight:700; color:var(--muted); letter-spacing:1px">Stickman Duel</div>
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
      <div class="names" style="text-align:right">Player 2 — Arrows</div>
    </div>
  </div>

  <canvas id="game" width="1200" height="520"></canvas>

  <div class="instructions">
    Controls — Player 1: W A D (move/jump) + S (punch) · Player 2: ↑ ← → (move/jump) + ↓ (punch) · First to deplete opponent wins.
  </div>
</div>

<div id="overlay" class="overlay">
  <div class="card">
    <div class="title">Stickman Duel</div>
    <div class="small">Two-player local or Bot — simple, modern, and responsive.</div>
    <div style="margin-bottom:12px">
      <strong>Player 1</strong> — WASD &middot; <strong>Player 2</strong> — Arrow keys
    </div>
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
const groundY = ch - 80, leftBound = 80, rightBound = cw - 80;

const AudioEnabled={val:true};
const audioCtx=new (window.AudioContext||window.webkitAudioContext)();
function playTone(freq,type='sine',length=0.12,gain=0.12,decay=0.02){
  if(!AudioEnabled.val) return;
  const now=audioCtx.currentTime, o=audioCtx.createOscillator(), g=audioCtx.createGain();
  o.type=type; o.frequency.setValueAtTime(freq,now);
  g.gain.setValueAtTime(gain,now); g.gain.exponentialRampToValueAtTime(0.0001,now+length+decay);
  o.connect(g); g.connect(audioCtx.destination); o.start(now); o.stop(now+length+decay);
}
function playPunchSound(){ playTone(600,'square',0.08,0.09,0.02); setTimeout(()=>playTone(420,'sawtooth',0.12,0.06,0.03),10);}
function playHitSound(){ playTone(160,'sine',0.18,0.15,0.03); setTimeout(()=>playTone(220,'triangle',0.1,0.08,0.02),30);}
function playWinSound(){ playTone(880,'sine',0.28,0.14,0.04); setTimeout(()=>playTone(1100,'sine',0.22,0.12,0.03),200);}
function playSpringSound(){ playTone(880,'triangle',0.15,0.15,0.02);}
function backgroundDrone(){ if(!AudioEnabled.val) return; const o=audioCtx.createOscillator(), g=audioCtx.createGain(); o.type='sine'; o.frequency.value=55; g.gain.value=0.02; o.connect(g); g.connect(audioCtx.destination); o.start(); return {osc:o,gain:g}; }
let bgNode = null;

class Player{
  constructor(x,color,facing=1){
    this.x=x; this.y=0; this.vx=0; this.vy=0; this.width=36; this.height=70;
    this.onGround=false; this.health=100; this.color=color; this.facing=facing;
    this.punching=false; this.punchTimer=0; this.stun=0; this.combo=0; this.comboTimer=0;
  }
  rect(){ return {x:this.x-this.width/2, y:this.y-this.height, w:this.width, h:this.height}; }
  punchHitbox(){
    const range=38, px=this.x+this.facing*(this.width/2+range/2), py=this.y-this.height/2;
    return {x:px-range/2, y:py-16, w:range, h:32};
  }
}

const p1 = new Player(360,'#7dd3fc',1);
const p2 = new Player(840,'#f472b6',-1);
let keys = {}, lastTime = performance.now(), running=false, botMode=false, botDifficulty='easy';

const spring = { x:cw/2, y:groundY, width:60, height:20, active:false, timer:0, maxLift:20 };

window.addEventListener('keydown', e => keys[e.code]=true);
window.addEventListener('keyup', e => keys[e.code]=false);

const bar1=document.getElementById('bar1'), bar2=document.getElementById('bar2'),
      label1=document.getElementById('label1'), label2=document.getElementById('label2'),
      overlay=document.getElementById('overlay');

document.getElementById('startBtn').addEventListener('click',()=>{ overlay.style.display='none'; botMode=false; startMatch(); });
document.getElementById('easyBotBtn').addEventListener('click',()=>{ overlay.style.display='none'; botMode=true; botDifficulty='easy'; startMatch(); });
document.getElementById('hardBotBtn').addEventListener('click',()=>{ overlay.style.display='none'; botMode=true; botDifficulty='hard'; startMatch(); });
document.getElementById('restart').addEventListener('click',()=>{ resetMatch(); });
document.getElementById('toggleSound').addEventListener('click',()=>{
  AudioEnabled.val = !AudioEnabled.val;
  if(!AudioEnabled.val && bgNode){ bgNode.osc.stop(); bgNode=null; }
  if(AudioEnabled.val && running && !bgNode) bgNode = backgroundDrone();
  document.getElementById('toggleSound').textContent = AudioEnabled.val?'Sound: On':'Sound: Off';
});

function applyPhysics(dt,player){
  const gravity=1400;
  player.vy+=gravity*dt; player.x+=player.vx*dt; player.y+=player.vy*dt;
  if(player.y>groundY){ player.y=groundY; player.vy=0; player.onGround=true; } else { player.onGround=false; }
  if(player.x<leftBound){ player.x=leftBound; player.vx=0; }
  if(player.x>rightBound){ player.x=rightBound; player.vx=0; }
  if(player.onGround && Math.abs(player.vx)<5) player.vx=0;
  if(player.onGround) player.vx*=0.88;
}
function rectIntersect(a,b){ return !(a.x+a.w<b.x || a.x>b.x+b.w || a.y+a.h<b.y || a.y>b.y+b.h); }
function resolvePunch(attacker,defender){
  if(attacker.punching && attacker.punchTimer>0.06){
    const aHit = attacker.punchHitbox(), dRect=defender.rect();
    if(rectIntersect(aHit,dRect) && defender.stun<=0){
      defender.health=Math.max(0, defender.health-8-Math.floor(Math.random()*6));
      defender.vx=attacker.facing*220; defender.vy=-140; defender.stun=0.26;
      attacker.combo++; attacker.comboTimer=1.0; defender.combo=0; defender.comboTimer=0;
      playHitSound(); return true;
    }
  }
  return false;
}

function botAI(controlled,target){
  const dx=target.x-controlled.x, dist=Math.abs(dx);
  if(botDifficulty==='easy'){
    controlled.vx += Math.sign(dx)*200*0.016;
    if(dist<80 && Math.random()<0.01 && !controlled.punching && controlled.stun<=0){
      controlled.punching=true; controlled.punchTimer=0; playPunchSound();
    }
    if(dist<60 && Math.random()<0.01 && controlled.onGround){ controlled.vy=-350; controlled.onGround=false; }
  } else {
    if(dist>100) controlled.vx += Math.sign(dx)*350*0.016; else controlled.vx*=0.8;
    if(dist<90 && Math.random()<0.03 && !controlled.punching && controlled.stun<=0){ controlled.punching=true; controlled.punchTimer=0; playPunchSound(); }
    if(target.punching && Math.abs(target.x-controlled.x)<60 && controlled.onGround){ controlled.vy=-400; controlled.onGround=false; }
    if(Math.random()<0.005 && controlled.onGround){ controlled.vy=-420; controlled.onGround=false; }
  }
}

function updatePlayer(player, dt, opponent){
  if(player.stun>0){ player.stun=Math.max(0, player.stun-dt); player.vx*=0.96; }
  if(player.punching){ player.punchTimer+=dt; if(player.punchTimer>0.25){ player.punching=false; player.punchTimer=0; } }
  if(player.punching && player.punchTimer>0.05){ resolvePunch(player, opponent); }
  if(player.comboTimer>0){ player.comboTimer=Math.max(0, player.comboTimer-dt); if(player.comboTimer===0) player.combo=0; }
  const maxSpeed=420; if(player.vx>maxSpeed) player.vx=maxSpeed; if(player.vx<-maxSpeed) player.vx=-maxSpeed;
  applyPhysics(dt, player);

  // spring
  const springTopY = spring.y - spring.height;
  const pRect = player.rect();
  if(rectIntersect({x:spring.x-spring.width/2, y:springTopY, w:spring.width, h:spring.height}, pRect) && player.vy>=0){
    player.vy=-900; spring.active=true; spring.timer=0.15; playSpringSound();
  }
}

function update(now){
  const dt=Math.min(0.032,(now-lastTime)/1000); lastTime=now;
  if(!running) return;

  // Input
  if(!botMode){
    if(keys['KeyA']){ p1.vx-=1200*dt; p1.facing=-1; }
    if(keys['KeyD']){ p1.vx+=1200*dt; p1.facing=1; }
    if(keys['KeyW'] && p1.onGround){ p1.vy=-450; p1.onGround=false; }
    if(keys['KeyS'] && !p1.punching && p1.stun<=0){ p1.punching=true; p1.punchTimer=0; playPunchSound(); }

    if(keys['ArrowLeft']){ p2.vx-=1200*dt; p2.facing=-1; }
    if(keys['ArrowRight']){ p2.vx+=1200*dt; p2.facing=1; }
    if(keys['ArrowUp'] && p2.onGround){ p2.vy=-450; p2.onGround=false; }
    if(keys['ArrowDown'] && !p2.punching && p2.stun<=0){ p2.punching=true; p2.punchTimer=0; playPunchSound(); }
  } else { botAI(p2,p1); }

  updatePlayer(p1, dt, p2);
  updatePlayer(p2, dt, p1);

  render();

  if(p1.health<=0 || p2.health<=0){
    running=false;
    setTimeout(()=>showWin(p1.health<=0?'Player 2':'Player 1'), 260);
    if(AudioEnabled.val) playWinSound();
    if(bgNode){ try{ bgNode.osc.stop(); }catch(e){}; bgNode=null; }
    return;
  }

  requestAnimationFrame(update);
}

let shakeAmount=0;
function render(){
  ctx.save();
  ctx.fillStyle='#0b1220'; ctx.fillRect(0,0,cw,ch);
  ctx.fillStyle='#071128'; ctx.fillRect(0,groundY,cw,ch-groundY);

  // spring
  if(spring.active){ spring.timer-=1/60; if(spring.timer<=0) spring.active=false; }
  const springYDraw = spring.active ? spring.y - spring.maxLift : spring.y;
  ctx.fillStyle='#facc15'; ctx.fillRect(spring.x-spring.width/2, springYDraw-spring.height, spring.width, spring.height);

  function drawPlayer(p){
    ctx.fillStyle=p.color;
    ctx.fillRect(p.x-p.width/2, p.y-p.height, p.width, p.height);
    if(p.punching){
      const hb=p.punchHitbox();
      ctx.fillStyle='rgba(255,255,255,0.2)';
      ctx.fillRect(hb.x,hb.y,hb.w,hb.h);
    }
  }
  drawPlayer(p1); drawPlayer(p2);

  // update bars
  bar1.style.width = p1.health+'%'; label1.textContent = p1.health+'%';
  bar2.style.width = p2.health+'%'; label2.textContent = p2.health+'%';

  ctx.restore();
}

function resetMatch(){
  p1.x=360; p1.y=0; p1.vx=0; p1.vy=0; p1.health=100; p1.stun=0; p1.punching=false; p1.combo=0; p1.comboTimer=0;
  p2.x=840; p2.y=0; p2.vx=0; p2.vy=0; p2.health=100; p2.stun=0; p2.punching=false; p2.combo=0; p2.comboTimer=0;
  spring.active=false; running=false; overlay.style.display='flex';
}

function startMatch(){
  lastTime=performance.now(); running=true;
  if(AudioEnabled.val && !bgNode) bgNode = backgroundDrone();
  requestAnimationFrame(update);
}

function showWin(winner){
  alert(winner+' wins!');
  resetMatch();
}
</script>
</body>
</html>
