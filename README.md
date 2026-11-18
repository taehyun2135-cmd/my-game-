<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="피하기 게임">
  <title>아이폰 미니 게임</title>
  <style>
    * { box-sizing: border-box; user-select: none; -webkit-user-select: none; -webkit-touch-callout: none; }
    body, html { margin: 0; padding: 0; width: 100%; height: 100%; overflow: hidden; background-color: #1C1C1E; font-family: -apple-system, BlinkMacSystemFont, sans-serif; }
    canvas { display: block; width: 100%; height: 100%; }
    #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; display: flex; flex-direction: column; justify-content: space-between; padding: env(safe-area-inset-top) 20px env(safe-area-inset-bottom) 20px; }
    #score { color: white; font-size: 28px; font-weight: 800; margin-top: 20px; }
    #start-screen { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center; color: white; pointer-events: auto; }
    button { background-color: #0A84FF; color: white; border: none; padding: 16px 36px; font-size: 20px; font-weight: 700; border-radius: 16px; cursor: pointer; box-shadow: 0 4px 12px rgba(10, 132, 255, 0.4); }
    button:active { transform: scale(0.96); }
  </style>
</head>
<body>
  <canvas id="gameCanvas"></canvas>
  <div id="ui-layer"><div id="score">0</div></div>
  <div id="start-screen">
    <h1 id="title-text" style="font-size: 36px; margin-bottom: 10px;">장애물 피하기</h1>
    <p id="sub-text" style="color: #AEAEB2; margin-bottom: 30px;">화면을 터치해서 좌우로 움직이세요</p>
    <button onclick="startGame()">게임 시작</button>
  </div>
  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const startScreen = document.getElementById('start-screen');
    const titleText = document.getElementById('title-text');
    const subText = document.getElementById('sub-text');
    let width, height, isPlaying = false, score = 0, animationId;
    const player = { x: 0, y: 0, radius: 20, color: '#0A84FF' };
    let enemies = [], spawnTimer = 0;

    function resize() {
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
      player.y = height - 100;
      if (!isPlaying) player.x = width / 2;
    }
    window.addEventListener('resize', resize);
    resize();

    function movePlayer(clientX) {
      if (!isPlaying) return;
      player.x = Math.max(player.radius, Math.min(width - player.radius, clientX));
    }

    window.addEventListener('touchmove', (e) => {
      e.preventDefault();
      movePlayer(e.touches[0].clientX);
    }, { passive: false });

    window.addEventListener('touchstart', (e) => {
      movePlayer(e.touches[0].clientX);
    });

    function startGame() {
      isPlaying = true; score = 0; enemies = []; spawnTimer = 0;
      player.x = width / 2; scoreEl.innerText = '0';
      startScreen.style.display = 'none';
      loop();
    }

    function gameOver() {
      isPlaying = false;
      cancelAnimationFrame(animationId);
      titleText.innerText = '게임 오버!';
      subText.innerText = `최종 점수: ${Math.floor(score / 10)}점`;
      startScreen.style.display = 'block';
    }

    function loop() {
      if (!isPlaying) return;
      ctx.clearRect(0, 0, width, height);
      score++;
      scoreEl.innerText = Math.floor(score / 10);
      spawnTimer++;
      const spawnRate = Math.max(15, 45 - Math.floor(score / 200));
      if (spawnTimer > spawnRate) {
        enemies.push({
          x: Math.random() * (width - 30) + 15, y: -20,
          size: Math.random() * 15 + 15, speed: Math.random() * 3 + 4 + (score / 500)
        });
        spawnTimer = 0;
      }
      ctx.beginPath();
      ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
      ctx.fillStyle = player.color;
      ctx.fill();

      for (let i = enemies.length - 1; i >= 0; i--) {
        const e = enemies[i];
        e.y += e.speed;
        ctx.fillStyle = '#FF453A';
        ctx.beginPath();
        ctx.roundRect(e.x - e.size / 2, e.y - e.size / 2, e.size, e.size, 6);
        ctx.fill();
        if (Math.hypot(player.x - e.x, player.y - e.y) < player.radius + e.size / 2 - 4) {
          gameOver(); return;
        }
        if (e.y > height + 30) enemies.splice(i, 1);
      }
      animationId = requestAnimationFrame(loop);
    }
  </script>
</body>
</html>
