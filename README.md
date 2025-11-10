<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
<title>Bounce Tales — Retro Rebound</title>

<!-- PWA manifest -->
<link rel="manifest" href="/manifest.json" />

<!-- Theme / preview meta -->
<meta name="theme-color" content="#0b1220" />
<meta property="og:title" content="Bounce Tales — Retro Rebound" />
<meta property="og:description" content="Classic bouncing ball platformer rebuilt for mobile & Farcaster frames. Tap to jump, collect stars, survive!" />
<meta property="og:image" content="https://your-vercel-domain.vercel.app/preview.png" /> <!-- replace after deploy -->
<meta property="og:url" content="https://your-vercel-domain.vercel.app" />
<meta name="fc:frame" content="vNext" />
<meta name="fc:frame:image" content="https://your-vercel-domain.vercel.app/preview.png" />
<meta name="fc:frame:button:1" content="▶ Play" />
<meta name="fc:frame:post_url" content="https://your-vercel-domain.vercel.app" />

<style>
  :root{
    --bg:#071226; --panel:rgba(255,255,255,0.04); --accent:#ff4757; --text:#e6f0ff;
  }
  html,body{height:100%;margin:0;background:linear-gradient(180deg,#06101a,#071226);font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
  .app{
    display:flex;flex-direction:column;align-items:center;justify-content:flex-start;height:100vh;padding:14px;
    -webkit-tap-highlight-color: transparent;
  }

  header{width:100%;display:flex;align-items:center;justify-content:space-between;color:var(--text);margin-bottom:10px}
  header h1{font-size:18px;margin:0}
  header .controls{display:flex;gap:8px}

  #viewport{
    width:100%;max-width:720px;flex:1;display:flex;align-items:center;justify-content:center;
  }

  #gameWrap{position:relative;width:100%;max-width:520px;background:linear-gradient(180deg,#0f1b2a,#071226);border-radius:14px;padding:12px;box-shadow:0 8px 30px rgba(0,0,0,0.6);border:1px solid rgba(255,255,255,0.03)}
  canvas{width:100%;height:auto;display:block;border-radius:10px;background:linear-gradient(180deg,#08121b,#071226);}

  .uiRow{display:flex;align-items:center;gap:8px;margin-top:10px;justify-content:center;color:var(--text)}
  .badge{background:var(--panel);padding:8px 12px;border-radius:10px;font-weight:600;color:var(--text);font-size:14px}
  .btn{background:linear-gradient(180deg,#0d2140,#0b1828);border:1px solid rgba(255,255,255,0.03);color:var(--text);padding:8px 12px;border-radius:10px;cursor:pointer}
  .btn.primary{background:linear-gradient(90deg,var(--accent),#ff7b7b);color:#071226;font-weight:700}

  /* overlays */
  .overlay{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:86%;max-width:420px;padding:14px;border-radius:12px;background:linear-gradient(180deg,rgba(0,0,0,0.5),rgba(0,0,0,0.35));backdrop-filter:blur(6px);box-shadow:0 8px 40px rgba(0,0,0,0.6);color:var(--text);z-index:50}
  .hidden{display:none!important}
  .center{text-align:center}

  /* mobile controls hint */
  .hint{font-size:13px;color:#9fb1c9;margin-top:8px;text-align:center}
  /* leaderboard */
  ol.leader{max-height:220px;overflow:auto;padding-left:18px;margin:8px 0}
  ol.leader li{margin:6px 0;color:var(--text)}

  /* small screens */
  @media (max-width:420px){
    header h1{font-size:16px}
  }
</style>
</head>
<body>
<div class="app" id="app">
  <header>
    <h1>Bounce Tales — Retro Rebound</h1>
    <div class="controls">
      <div class="badge" id="scoreBadge">Score: 0</div>
      <button class="btn" id="soundToggle">🔊</button>
    </div>
  </header>

  <main id="viewport">
    <div id="gameWrap">
      <canvas id="c" width="540" height="720" aria-label="Bounce Tales game"></canvas>

      <!-- Start overlay -->
      <div class="overlay center" id="startOverlay">
        <h2>Bounce Tales</h2>
        <p style="opacity:.9">Tap to jump • Hold for higher jump • Swipe to move • Collect stars</p>
        <div style="display:flex;gap:8px;justify-content:center;margin-top:12px">
          <button class="btn primary" id="startBtn">Start Game</button>
          <button class="btn" id="leaderBtn">Leaderboard</button>
        </div>
        <p class="hint">Built for mobile & Farcaster mini-apps</p>
      </div>

      <!-- Game Over overlay -->
      <div class="overlay center hidden" id="overOverlay">
        <h2>Game Over</h2>
        <div id="overScore" style="font-weight:700;margin:8px 0">Score: 0</div>
        <div style="display:flex;gap:10px;justify-content:center">
          <button class="btn primary" id="retryBtn">Try Again</button>
          <button class="btn" id="homeBtn">Home</button>
        </div>
      </div>

      <!-- Leaderboard -->
      <div class="overlay center hidden" id="leaderOverlay">
        <h3>Local Leaderboard</h3>
        <ol class="leader" id="leaderList"></ol>
        <div style="display:flex;gap:10px;justify-content:center">
          <button class="btn" id="leaderClose">Close</button>
          <button class="btn" id="leaderClear">Reset</button>
        </div>
      </div>

    </div>
  </main>

  <div class="uiRow">
    <div class="badge">High: <span id="highBadge">0</span></div>
    <div class="badge" id="starsBadge">Stars: 0</div>
  </div>

  <p class="hint">Tip: single tap to jump. Press & hold for higher jumps. Tilt to move (optional).</p>
</div>

<!-- sounds -->
<audio id="sfxJump" src="https://assets.mixkit.co/sfx/preview/mixkit-quick-jump-arcade-2076.mp3"></audio>
<audio id="sfxHit" src="https://assets.mixkit.co/sfx/preview/mixkit-retro-game-impact-2124.mp3"></audio>
<audio id="sfxStar" src="https://assets.mixkit.co/sfx/preview/mixkit-coin-arcade-2432.mp3"></audio>
<audio id="bgMusic" loop src="https://assets.mixkit.co/music/preview/mixkit-arcade-8-bit-2365.mp3"></audio>

<script>
/* ============================
   Bounce Tales — single-file game
   Mobile-first, Farcaster-ready
   ============================ */

const canvas = document.getElementById('c');
const ctx = canvas.getContext('2d');
let W = canvas.width, H = canvas.height;

// scaling for mobile (keeps logical canvas size but CSS fits)
function resizeCanvas(){
  const container = document.getElementById('gameWrap');
  const maxW = Math.min(window.innerWidth - 32, 540);
  canvas.style.width = maxW + 'px';
  // keep aspect ratio 3:4 (540x720)
}
resizeCanvas();
window.addEventListener('resize', resizeCanvas);

/* game state */
let gameState = 'menu'; // menu, playing, over
let score = 0, starsCollected = 0;
let high = Number(localStorage.getItem('bounce_high') || 0);
document.getElementById('highBadge').innerText = high;
document.getElementById('scoreBadge').innerText = `Score: ${score}`;
document.getElementById('starsBadge').innerText = `Stars: ${starsCollected}`;

const sfxJump = document.getElementById('sfxJump');
const sfxHit = document.getElementById('sfxHit');
const sfxStar = document.getElementById('sfxStar');
const bgMusic = document.getElementById('bgMusic');
let soundOn = true;
document.getElementById('soundToggle').addEventListener('click', ()=>{
  soundOn = !soundOn;
  document.getElementById('soundToggle').innerText = soundOn ? '🔊' : '🔈';
  if(soundOn) bgMusic.play(); else bgMusic.pause();
});

/* physics + world */
const gravity = 0.8;
const ballRadius = 18;
let player;
let platforms = [];
let stars = [];
let scrollY = 0; // vertical camera offset

function newGame(){
  score = 0; starsCollected = 0;
  player = { x: W/2, y: H - 200, vx:0, vy:0, onGround:false, size: ballRadius };
  platforms = [];
  stars = [];
  spawnInitialPlatforms();
  updateUI();
}

function spawnInitialPlatforms(){
  // create a set of platforms spaced upward for a climbing experience
  const gap = 90;
  for(let i=0;i<14;i++){
    const py = H - (i * gap) - 60;
    platforms.push({ x: 60 + (i%2===0?0:80), y: py, w: 140, h: 14, type:'normal' });
    // add star randomly
    if(Math.random() < 0.45){
      stars.push({ x: platforms[i].x + 20 + Math.random()*80, y: py - 30, r:8, collected:false });
    }
  }
}

function spawnPlatformAbove(y){
  const x = 40 + Math.random()*(W-120);
  const w = 100 + Math.random()*80;
  platforms.push({ x, y, w, h:14, type:'normal' });
  if(Math.random() < 0.5) stars.push({ x: x + 20 + Math.random()*(w-40), y: y-30, r:8, collected:false });
}

/* input */
let pointerDown = false, pointerStartY = 0, pointerStartX = 0, pointerHeld = false;
canvas.addEventListener('touchstart', (e)=>{
  e.preventDefault();
  pointerDown = true; pointerHeld = true;
  if(gameState === 'menu'){ startPlay(); return; }
  if(gameState === 'over'){ return; }
});
canvas.addEventListener('touchend', (e)=>{
  e.preventDefault();
  pointerDown = false; pointerHeld = false;
});
canvas.addEventListener('mousedown', (e)=>{
  pointerDown = true; pointerHeld = true;
  if(gameState === 'menu'){ startPlay(); return; }
});
canvas.addEventListener('mouseup', ()=>{ pointerDown=false; pointerHeld=false; });

/* basic tilt support */
if(window.DeviceOrientationEvent){
  window.addEventListener('deviceorientation', (ev)=>{
    // use gamma for left-right tilt
    const tilt = ev.gamma || 0;
    if(player) player.vx = tilt * 0.25; // subtle control
  });
}

/* core loop */
let last=0;
function loop(ts){
  const dt = Math.min(32, ts - last); last = ts;
  update(dt/16);
  render();
  requestAnimationFrame(loop);
}

/* update world */
function update(dt){
  if(gameState !== 'playing') return;

  // player physics
  player.vy += gravity * (dt/2);
  player.x += player.vx * (dt/1.2);
  player.y += player.vy * (dt/1.2);

  // horizontal wrap
  if(player.x < -20) player.x = W + 20;
  if(player.x > W + 20) player.x = -20;

  // collision with platforms (simple AABB, from above only)
  player.onGround = false;
  for(let p of platforms){
    if(player.vy > 0 &&
       player.x + player.size > p.x &&
       player.x - player.size < p.x + p.w &&
       player.y + player.size > p.y &&
       player.y + player.size < p.y + p.h + Math.abs(player.vy)+6){
      // land
      player.y = p.y - player.size;
      player.vy = -Math.abs(player.vy) * 0.6; // bounce
      player.onGround = true;
      // scoring when bounce with velocity
      if(Math.abs(player.vy) > 4){ score += 1; updateUI(); }
    }
  }

  // stars pickup
  for(let s of stars){
    if(!s.collected && Math.hypot(player.x - s.x, player.y - s.y) < player.size + s.r){
      s.collected = true; starsCollected++; score += 5; updateUI();
      if(soundOn) sfxStar.currentTime = 0, sfxStar.play();
    }
  }

  // scrolling camera: keep player above quarter height
  const cameraTarget = H * 0.4;
  if(player.y < cameraTarget){
    const dy = cameraTarget - player.y;
    scrollY += dy;
    // move platforms and stars down
    for(let p of platforms) p.y += dy;
    for(let s of stars) s.y += dy;
    // spawn new platforms above
    while(platforms.length < 18){
      const highestY = Math.min(...platforms.map(p=>p.y));
      spawnPlatformAbove(highestY - 90);
    }
  }

  // lose condition: falls below bottom
  if(player.y - player.size > H + 120){
    endGame();
  }

  // limit velocities
  player.vx *= 0.995;
  player.vy = Math.max(Math.min(player.vy, 40), -40);

  // if pointer is pressed -> jump stronger
  if(pointerHeld && player.onGround){
    player.vy = -14; // big jump
    if(soundOn) sfxJump.currentTime = 0, sfxJump.play();
    pointerHeld = false; // single jump per hold to avoid runaway
  }
}

/* rendering */
function render(){
  ctx.clearRect(0,0,W,H);

  // background gradient + subtle grid lines
  const g = ctx.createLinearGradient(0,0,0,H);
  g.addColorStop(0,'#081226'); g.addColorStop(1,'#021018');
  ctx.fillStyle = g; ctx.fillRect(0,0,W,H);

  // parallax stars background
  for(let i=0;i<40;i++){
    const x = (i*73) % W;
    const y = (i*97 + (scrollY*0.02)) % H;
    ctx.fillStyle = 'rgba(255,255,255,0.02)'; ctx.fillRect(x,y,2,2);
  }

  // draw platforms
  for(let p of platforms){
    // platform shadow
    ctx.fillStyle = 'rgba(0,0,0,0.35)';
    ctx.fillRect(p.x+2, p.y+6, p.w, p.h);
    // platform body
    const pg = ctx.createLinearGradient(p.x, p.y, p.x + p.w, p.y + 20);
    pg.addColorStop(0, '#ffffff11'); pg.addColorStop(1, '#ffffff06');
    ctx.fillStyle = pg;
    ctx.fillRect(p.x, p.y, p.w, p.h);
    // accent line
    ctx.fillStyle = '#00b36b';
    ctx.fillRect(p.x + p.w/2 - 18, p.y + 2, 36, 4);
  }

  // draw stars
  for(let s of stars){
    if(s.collected) continue;
    ctx.save();
    ctx.translate(s.x, s.y);
    ctx.fillStyle = '#ffd166';
    ctx.beginPath();
    ctx.arc(0,0,s.r,0,Math.PI*2); ctx.fill();
    ctx.restore();
  }

  // draw player (ball) with shading
  const px = player.x, py = player.y;
  const radial = ctx.createRadialGradient(px-6, py-8, 4, px, py, player.size+6);
  radial.addColorStop(0,'#ffffff'); radial.addColorStop(0.2,'#fffb'); radial.addColorStop(0.6,'#ffecb7'); radial.addColorStop(1,'#e74c3c');
  ctx.fillStyle = radial;
  ctx.beginPath();
  ctx.arc(px, py, player.size, 0, Math.PI*2); ctx.fill();

  // rim
  ctx.strokeStyle = 'rgba(0,0,0,0.25)'; ctx.lineWidth = 2; ctx.beginPath(); ctx.arc(px,py,player.size-2,0,Math.PI*2); ctx.stroke();

  // UI text overlay (score done in badges)
}

/* UI helpers */
function updateUI(){
  document.getElementById('scoreBadge').innerText = `Score: ${score}`;
  document.getElementById('starsBadge').innerText = `Stars: ${starsCollected}`;
}

/* start & end */
function startPlay(){
  newGame();
  gameState = 'playing';
  document.getElementById('startOverlay').classList.add('hidden');
  document.getElementById('overOverlay').classList.add('hidden');
  document.getElementById('leaderOverlay').classList.add('hidden');
  if(soundOn) bgMusic.play();
}

function endGame(){
  gameState = 'over';
  document.getElementById('overOverlay').classList.remove('hidden');
  document.getElementById('overScore').innerText = `Score: ${score}`;
  // save leaderboard locally
  const lb = JSON.parse(localStorage.getItem('bounce_leader')||'[]');
  lb.push({score, ts:Date.now()}); lb.sort((a,b)=>b.score-a.score); localStorage.setItem('bounce_leader', JSON.stringify(lb.slice(0,10)));
  if(score > high){ high = score; localStorage.setItem('bounce_high', high); document.getElementById('highBadge').innerText = high; }
  if(soundOn) sfxHit.currentTime = 0, sfxHit.play();
}

/* leaderboard controls */
document.getElementById('leaderBtn').addEventListener('click', ()=>{
  const lb = JSON.parse(localStorage.getItem('bounce_leader')||'[]');
  const el = document.getElementById('leaderList'); el.innerHTML = '';
  if(lb.length === 0) el.innerHTML = '<li style="opacity:.7">No scores yet — play to add</li>';
  else lb.forEach((r,i)=>{ const li = document.createElement('li'); li.innerText = `${i+1}. ${r.score} — ${new Date(r.ts).toLocaleString()}`; el.appendChild(li); });
  document.getElementById('leaderOverlay').classList.remove('hidden');
});
document.getElementById('leaderClose').addEventListener('click', ()=> document.getElementById('leaderOverlay').classList.add('hidden'));
document.getElementById('leaderClear').addEventListener('click', ()=>{ if(confirm('Clear local leaderboard?')){ localStorage.removeItem('bounce_leader'); document.getElementById('leaderList').innerHTML=''; }});

document.getElementById('startBtn').addEventListener('click', ()=> startPlay());
document.getElementById('retryBtn').addEventListener('click', ()=> { startPlay(); });
document.getElementById('homeBtn').addEventListener('click', ()=> { document.getElementById('overOverlay').classList.add('hidden'); document.getElementById('startOverlay').classList.remove('hidden'); });

/* quick keyboard helpers for desktop */
window.addEventListener('keydown', (e)=>{
  if(e.key === ' ' || e.key === 'ArrowUp') {
    if(gameState === 'menu') startPlay();
    else if(gameState === 'playing' && player.onGround){ player.vy = -14; if(soundOn) sfxJump.currentTime = 0, sfxJump.play(); }
    else if(gameState === 'over'){ startPlay(); }
  }
  if(e.key === 'p'){ if(gameState === 'playing') { gameState = 'menu'; document.getElementById('startOverlay').classList.remove('hidden'); } }
});

/* initial setup */
newGame();
requestAnimationFrame(loop);

/* ============================
   FARCASTER SDK READY() FIX
   - loads UMD SDK dynamically and calls ready()
   - safe for Vercel static hosting
   ============================ */
(async ()=>{
  try{
    const s = document.createElement('script');
    s.src = 'https://cdn.jsdelivr.net/npm/@farcaster/frame-sdk@latest/dist/sdk.umd.js';
    s.onload = ()=>{
      // UMD exposes window.frameSDK
      if(window.frameSDK && window.frameSDK.actions && typeof window.frameSDK.actions.ready === 'function'){
        try{ window.frameSDK.actions.ready(); console.log('✅ Farcaster ready() called'); }catch(e){console.warn('ready() error',e)}
      } else {
        console.warn('frameSDK loaded but actions.ready not found');
      }
    };
    s.onerror = (e)=> console.warn('Farcaster SDK failed to load', e);
    document.body.appendChild(s);
  }catch(err){
    console.error('Farcaster SDK injection failed', err);
  }
})();
</script>
</body>
</html>
