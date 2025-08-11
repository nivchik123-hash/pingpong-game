<!doctype html>
<html lang="he">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>פינג-פונג משופר</title>
  <style>
    html,body{height:100%;margin:0;background:linear-gradient(180deg,#1e1e1e,#0d0d0d);color:#eee;font-family:Arial,Helvetica,sans-serif}
    .wrap{display:flex;flex-direction:column;align-items:center;gap:8px;padding:12px}
    canvas{background:radial-gradient(circle at center,#222,#111);border:3px solid #444;display:block;box-shadow:0 0 20px rgba(0,0,0,0.7)}
    .controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap;justify-content:center}
    button,input{padding:6px 10px;border-radius:8px;border:1px solid #555;background:linear-gradient(#333,#222);color:#eee;cursor:pointer;transition:0.2s}
    button:hover,input:hover{background:linear-gradient(#444,#222)}
    label{font-size:14px}
    .hint{font-size:13px;color:#bbb;text-align:center}
  </style>
</head>
<body>
  <div class="wrap">
    <h2>🏓 פינג-פונג משופר</h2>
    <canvas id="c" width="800" height="480"></canvas>
    <div class="controls">
      <button id="modeBtn">מצב: נגד המחשב</button>
      <button id="restart">התחל מחדש</button>
      <button id="learnBtn">איך משחקים?</button>
      <label class="hint">קושי AI: <input id="difficulty" type="range" min="1" max="10" value="5"></label>
      <label class="hint">מהירות כדור: <input id="speed" type="range" min="2" max="8" value="4"></label>
    </div>
    <div class="hint">בקרות: שחקן שמאל - W / S או ⬆ / ⬇ | שחקן ימין - חצים למעלה/מטה | רווח - הפסקה/המשך</div>
  </div>

<script>
(() => {
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');
  const modeBtn = document.getElementById('modeBtn');
  const restartBtn = document.getElementById('restart');
  const learnBtn = document.getElementById('learnBtn');
  const diffRange = document.getElementById('difficulty');
  const speedRange = document.getElementById('speed');

  const W = canvas.width, H = canvas.height;
  const paddle = (x) => ({ x, y: H/2 - 50, w: 12, h: 100, vy: 0, speed: 6 });
  let left = paddle(20), right = paddle(W - 32);
  let ball = { x: W/2, y: H/2, r: 8, vx: 4, vy: 2 };
  let score = { left: 0, right: 0 };
  let paused = true;
  let mode = 'AI';

  function resetBall(direction) {
    ball.x = W/2; ball.y = H/2;
    const base = parseFloat(speedRange.value);
    const angle = (Math.random() * Math.PI/4) - Math.PI/8;
    const sign = direction || (Math.random() < 0.5 ? -1 : 1);
    ball.vx = sign * base * (Math.cos(angle));
    ball.vy = base * Math.sin(angle);
  }

  function resetAll(){
    left.y = H/2 - left.h/2; right.y = H/2 - right.h/2;
    score.left = 0; score.right = 0; paused = true; resetBall(1);
  }

  function aiMove(pad) {
    const difficulty = parseInt(diffRange.value,10);
    const center = pad.y + pad.h/2;
    const react = Math.min(8, 1 + difficulty);
    if (Math.abs(center - ball.y) > 10) {
      if (center < ball.y) pad.y += react;
      else pad.y -= react;
    }
    pad.y = Math.max(0, Math.min(H - pad.h, pad.y));
  }

  const keys = {};
  window.addEventListener('keydown', (e) => {
    keys[e.key.toLowerCase()] = true;
    if (e.code === 'ArrowUp') keys['w'] = true;
    if (e.code === 'ArrowDown') keys['s'] = true;
    if (e.code === 'Space') paused = !paused;
  });
  window.addEventListener('keyup', (e) => {
    keys[e.key.toLowerCase()] = false;
    if (e.code === 'ArrowUp') keys['w'] = false;
    if (e.code === 'ArrowDown') keys['s'] = false;
  });

  modeBtn.addEventListener('click', () => {
    mode = (mode === 'AI') ? '2P' : 'AI';
    modeBtn.textContent = mode === 'AI' ? 'מצב: נגד המחשב' : 'מצב: 2 שחקנים';
    resetAll();
  });
  restartBtn.addEventListener('click', resetAll);
  learnBtn.addEventListener('click', () => {
    alert(`איך משחקים:\n\nמטרתך היא להחזיר את הכדור עם המחבט שלך, ולא לתת לו לעבור אותך.\nשחקן שמאל: W/S או חצים למעלה/מטה, שחקן ימין: חצים למעלה/מטה.\nרווח - הפסקה/המשך, ניתן לבחור מצב נגד מחשב או 2 שחקנים.`);
  });

  function update() {
    if (keys['w']) left.y -= left.speed;
    if (keys['s']) left.y += left.speed;
    if (mode === '2P') {
      if (keys['arrowup']) right.y -= right.speed;
      if (keys['arrowdown']) right.y += right.speed;
    } else {
      aiMove(right);
    }

    left.y = Math.max(0, Math.min(H - left.h, left.y));
    right.y = Math.max(0, Math.min(H - right.h, right.y));

    if (!paused) {
      ball.x += ball.vx; ball.y += ball.vy;

      if (ball.y - ball.r < 0) { ball.y = ball.r; ball.vy *= -1; }
      if (ball.y + ball.r > H) { ball.y = H - ball.r; ball.vy *= -1; }

      if (ball.x - ball.r < left.x + left.w && ball.y > left.y && ball.y < left.y + left.h) {
        ball.x = left.x + left.w + ball.r; ball.vx *= -1.05;
        const hit = (ball.y - (left.y + left.h/2)) / (left.h/2);
        ball.vy += hit * 2;
      }
      if (ball.x + ball.r > right.x && ball.y > right.y && ball.y < right.y + right.h) {
        ball.x = right.x - ball.r; ball.vx *= -1.05;
        const hit = (ball.y - (right.y + right.h/2)) / (right.h/2);
        ball.vy += hit * 2;
      }

      if (ball.x < 0) { score.right++; paused = true; resetBall(1); }
      if (ball.x > W) { score.left++; paused = true; resetBall(-1); }

      const max = 14;
      const spd = Math.hypot(ball.vx, ball.vy);
      if (spd > max) {
        ball.vx = (ball.vx / spd) * max;
        ball.vy = (ball.vy / spd) * max;
      }
    }
  }

  function draw() {
    ctx.clearRect(0,0,W,H);
    ctx.strokeStyle = 'rgba(255,255,255,0.2)';
    ctx.setLineDash([10,15]);
    ctx.beginPath();
    ctx.moveTo(W/2,0); ctx.lineTo(W/2,H);
    ctx.stroke();
    ctx.setLineDash([]);

    ctx.fillStyle = '#fff';
    ctx.fillRect(left.x, left.y, left.w, left.h);
    ctx.fillRect(right.x, right.y, right.w, right.h);

    ctx.beginPath(); ctx.arc(ball.x, ball.y, ball.r, 0, Math.PI*2);
    ctx.shadowBlur = 10; ctx.shadowColor = '#fff'; ctx.fill();
    ctx.shadowBlur = 0;

    ctx.font = '28px Arial'; ctx.textAlign = 'center';
    ctx.fillText(score.left, W/2 - 60, 40);
    ctx.fillText(score.right, W/2 + 60, 40);

    ctx.font = '16px Arial'; ctx.fillStyle = '#bbb';
    if (paused) ctx.fillText('Space להמשך/השהייה — המשחק בהשהיה', W/2, H - 20);
  }

  function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
  }

  resetAll();
  loop();
  canvas.addEventListener('click', () => paused = !paused);
})();
</script>
</body>
</html>
