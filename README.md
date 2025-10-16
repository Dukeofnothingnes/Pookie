# Pookie
For you
<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>For You — I'm Sorry (Interactive)</title>
  <style>
    :root{
      --bg:#111;
      --pink:#ff6fa3;
      --rose:#ff4d6d;
      --glass: rgba(255,255,255,0.06);
      --card: rgba(255,255,255,0.05);
      --accent: rgba(255,200,120,0.12);
      color-scheme: dark;
    }
    html,body{height:100%;margin:0;font-family: Inter, system-ui, Segoe UI, Roboto, Helvetica, Arial; background: radial-gradient(1200px 700px at 10% 10%, rgba(255,150,120,0.06), transparent), linear-gradient(180deg,#0f0f12 0%, #081018 100%); overflow:hidden}/* Center card */
.card{
  position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);
  width:min(940px,92%);max-width:1100px;padding:28px;border-radius:18px;background:linear-gradient(135deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));box-shadow:0 10px 40px rgba(0,0,0,0.6);backdrop-filter: blur(6px);
  display:grid;grid-template-columns:1fr 360px;gap:18px;align-items:center
}
.left{padding:6px}
h1{margin:0 0 8px 0;font-size:28px;letter-spacing:-0.3px}
p.lead{margin:0 0 16px 0;color:#e8dfe6}
.message{
  background: linear-gradient(90deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));
  padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.03);font-size:18px;line-height:1.5;color:#ffdfe8
}
.controls{margin-top:12px;display:flex;gap:10px;align-items:center}
.btn{padding:10px 16px;border-radius:10px;border:0;background:linear-gradient(90deg,var(--pink),var(--rose));color:white;font-weight:600;cursor:pointer;box-shadow:0 6px 18px rgba(255,80,120,0.12)}
.muted{background:transparent;border:1px solid rgba(255,255,255,0.06);color:#ffdfe8}

/* Right column: canvas */
.stage{position:relative;height:380px;border-radius:12px;overflow:hidden;background:linear-gradient(180deg, rgba(255,180,180,0.02), rgba(220,120,160,0.01));display:flex;align-items:center;justify-content:center}
canvas{display:block}

/* footer small */
.footer{font-size:12px;color:rgba(255,255,255,0.45);margin-top:10px}

/* Love badge */
.badge{position:fixed;right:20px;bottom:20px;background:linear-gradient(90deg,var(--pink),var(--rose));padding:10px 14px;border-radius:999px;color:white;font-weight:700;box-shadow:0 10px 30px rgba(255,100,140,0.12)}

/* responsive */
@media (max-width:880px){.card{grid-template-columns:1fr;padding:18px;gap:10px}.stage{height:260px}}

  </style>
</head>
<body>  <div class="card" role="main">
    <div class="left">
      <h1>For you — I made this</h1>
      <p class="lead">I’m sorry for not being able to come see you on Friday, so I made this. I’m not the greatest with words, but I hope this helps 💌</p><div class="message">
    <strong>Message:</strong>
    <div style="margin-top:8px">You mean the world to me — I miss you and I love you more than I can say. Every little heartbeat on this page is a tiny reminder of how much I care.</div>
  </div>

  <div class="controls">
    <button id="playBtn" class="btn">Play music — Golden Hour</button>
    <button id="pauseBtn" class="btn muted">Pause</button>
    <div style="margin-left:12px;font-size:13px;color:rgba(255,255,255,0.7)">Tap hearts to make them bounce!</div>
  </div>

  <div class="footer">Tip: If music doesn't autoplay, click "Play music" (some browsers block autoplay). This page uses the official YouTube video for "Golden Hour" by JVKE.</div>
</div>

<div class="stage">
  <canvas id="hearts"></canvas>
</div>

  </div>  <div class="badge">❤️ I miss you</div>  <!-- Hidden YouTube player -->  <div id="player-container" style="display:none"></div>  <script>
    /* ----------------------------
       HEARTS CANVAS — bouncing hearts
       ----------------------------*/
    const canvas = document.getElementById('hearts');
    const ctx = canvas.getContext('2d');

    // resize to stage
    function resize(){
      const rect = canvas.getBoundingClientRect();
      canvas.width = rect.width * devicePixelRatio;
      canvas.height = rect.height * devicePixelRatio;
      ctx.setTransform(devicePixelRatio,0,0,devicePixelRatio,0,0);
    }
    window.addEventListener('resize', resize);
    // initial size after layout
    requestAnimationFrame(resize);

    // Heart drawing helper (path)
    function drawHeart(x,y,size,alpha,rotation){
      ctx.save();
      ctx.translate(x,y);
      ctx.rotate(rotation);
      ctx.scale(size,size);
      ctx.globalAlpha = alpha;
      ctx.beginPath();
      ctx.moveTo(0,0.35);
      ctx.bezierCurveTo(0,0.35,-0.6,-0.15,-0.6,-0.55);
      ctx.bezierCurveTo(-0.6,-1.05, -0.2,-1.4,0,-1.05);
      ctx.bezierCurveTo(0.2,-1.4,0.6,-1.05,0.6,-0.55);
      ctx.bezierCurveTo(0.6,-0.15,0,0.35,0,0.35);
      ctx.closePath();
      const grd = ctx.createLinearGradient(-0.6,-1.3,0.6,0.5);
      grd.addColorStop(0,'#ff9fb6');
      grd.addColorStop(0.6,'#ff5f7a');
      ctx.fillStyle = grd;
      ctx.fill();
      ctx.restore();
    }

    // Heart objects
    const hearts = [];
    const STAGE = {w:400,h:300};

    function spawnHeart(x,y){
      const size = 8 + Math.random()*18;
      hearts.push({
        x: x ?? (Math.random()*(canvas.width/devicePixelRatio - 40)+20),
        y: y ?? (Math.random()*(canvas.height/devicePixelRatio - 40)+20),
        vx: (Math.random()-0.5)*1.4,
        vy: (Math.random()-0.5)*1.4,
        size:size/40, // scale
        alpha:0.85 - Math.random()*0.35,
        rot: (Math.random()-0.5)*0.6
      });
    }

    // initial hearts
    for(let i=0;i<26;i++) spawnHeart();

    // animation loop
    function step(){
      const W = canvas.width/devicePixelRatio;
      const H = canvas.height/devicePixelRatio;
      ctx.clearRect(0,0,W,H);

      for(let h of hearts){
        h.x += h.vx;
        h.y += h.vy;
        h.rot += (h.vx+h.vy)*0.005;

        // bounce off walls
        if(h.x < 16){ h.x = 16; h.vx = Math.abs(h.vx); }
        if(h.x > W-16){ h.x = W-16; h.vx = -Math.abs(h.vx); }
        if(h.y < 16){ h.y = 16; h.vy = Math.abs(h.vy); }
        if(h.y > H-16){ h.y = H-16; h.vy = -Math.abs(h.vy); }

        drawHeart(h.x,h.y,h.size,h.alpha,h.rot);
      }

      requestAnimationFrame(step);
    }
    requestAnimationFrame(step);

    // Interaction: on click/touch, find nearest heart and bump it away
    function interact(clientX,clientY){
      const rect = canvas.getBoundingClientRect();
      const x = (clientX - rect.left);
      const y = (clientY - rect.top);

      // spawn a little new heart where user tapped
      spawnHeart(x,y);

      // push nearest hearts
      for(let h of hearts){
        const dx = h.x - x; const dy = h.y - y; const dist = Math.hypot(dx,dy);
        if(dist < 120){
          const force = 3*(1 - dist/140);
          h.vx += (dx/dist||0)*force + (Math.random()-0.5)*0.6;
          h.vy += (dy/dist||0)*force + (Math.random()-0.5)*0.6;
        }
      }
    }
    canvas.addEventListener('pointerdown', e=>{ interact(e.clientX,e.clientY); });

    // Clicking anywhere on the page also interacts
    document.body.addEventListener('pointerdown', e=>{
      // ignore clicks on controls
      if(e.target.closest('.controls')) return;
      interact(e.clientX,e.clientY);
    });

    /* ----------------------------
       YouTube player for Golden Hour
       ----------------------------*/
    // We load the iframe API and create a hidden player so we can control playback.
    // Official video ID used below.
    const VIDEO_ID = 'PEM0Vs8jf1w'; // JVKE - Golden Hour (official)

    let ytPlayer;
    function onYouTubeIframeAPIReady(){
      ytPlayer = new YT.Player('player-container',{
        height: '0',width:'0',videoId:VIDEO_ID,playerVars:{rel:0,loop:1,controls:0,showinfo:0,modestbranding:1,playsinline:1,autoplay:0,iv_load_policy:3,disablekb:1,playlist:VIDEO_ID},events:{onReady:e=>{ /* ready */ }}
      });
    }

    // load the YouTube API script
    (function loadYT(){
      const tag = document.createElement('script');
      tag.src = "https://www.youtube.com/iframe_api";
      document.head.appendChild(tag);
      window.onYouTubeIframeAPIReady = onYouTubeIframeAPIReady;
    })();

    // Controls
    const playBtn = document.getElementById('playBtn');
    const pauseBtn = document.getElementById('pauseBtn');

    playBtn.addEventListener('click', ()=>{
      if(ytPlayer && ytPlayer.playVideo){
        ytPlayer.playVideo();
      } else {
        // If API not ready, open YouTube in new tab as fallback
        window.open('https://www.youtube.com/watch?v='+VIDEO_ID,'_blank');
      }
    });
    pauseBtn.addEventListener('click', ()=>{ if(ytPlayer && ytPlayer.pauseVideo) ytPlayer.pauseVideo(); });

    // small accessibility: space toggles play/pause
    window.addEventListener('keydown', e=>{ if(e.code==='Space'){ e.preventDefault(); if(ytPlayer && ytPlayer.getPlayerState && ytPlayer.getPlayerState()===1) ytPlayer.pauseVideo(); else if(ytPlayer) ytPlayer.playVideo(); }});

    /* Resize canvas to parent element size initially */
    function fitCanvasToParent(){
      const parent = canvas.parentElement;
      canvas.style.width = parent.clientWidth + 'px';
      canvas.style.height = parent.clientHeight + 'px';
      resize();
    }
    window.addEventListener('load', fitCanvasToParent);
    window.addEventListener('resize', fitCanvasToParent);

  </script></body>
</html>
