# test1
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>아이작 라이트 (Isaac Lite)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #1a1a1a; font-family: 'Inter', sans-serif; }
        canvas { display: block; cursor: crosshair; }
        #ui { position: absolute; top: 20px; left: 20px; pointer-events: none; color: white; }
    </style>
</head>
<body>

    <div id="ui">
        <h1 class="text-2xl font-bold mb-2">ISAAC LITE</h1>
        <div id="health" class="text-red-500 font-bold">HP: 100</div>
        <div id="score" class="text-blue-300">Score: 0</div>
    </div>

    <div id="gameOver" class="hidden absolute inset-0 flex items-center justify-center bg-black/80 text-white z-10">
        <div class="text-center">
            <h2 class="text-4xl mb-4">GAME OVER</h2>
            <button onclick="location.reload()" class="bg-red-600 px-6 py-2 rounded-lg hover:bg-red-700 transition">다시 하기</button>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const healthEl = document.getElementById('health');
        const scoreEl = document.getElementById('score');
        const gameOverEl = document.getElementById('gameOver');

        let width, height;
        let score = 0;
        let isGameOver = false;

        const keys = {};
        const player = { x: 0, y: 0, r: 15, hp: 100, speed: 4 };
        const bullets = [];
        const enemies = [];

        function resize() {
            width = canvas.width = window.innerWidth;
            height = canvas.height = window.innerHeight;
            player.x = width / 2;
            player.y = height / 2;
        }
        window.addEventListener('resize', resize);
        resize();

        window.addEventListener('keydown', e => keys[e.key] = true);
        window.addEventListener('keyup', e => keys[e.key] = false);

        function shoot(dirX, dirY) {
            bullets.push({ x: player.x, y: player.y, dx: dirX * 7, dy: dirY * 7, life: 30 });
        }

        function spawnEnemy() {
            if (isGameOver) return;
            const side = Math.floor(Math.random() * 4);
            let x, y;
            if (side === 0) { x = Math.random() * width; y = -20; }
            else if (side === 1) { x = width + 20; y = Math.random() * height; }
            else if (side === 2) { x = Math.random() * width; y = height + 20; }
            else { x = -20; y = Math.random() * height; }
            enemies.push({ x, y, r: 12, speed: 1.5 + Math.random() * 1 });
        }
        setInterval(spawnEnemy, 1000);

        function update() {
            if (isGameOver) return;

            // Player Movement
            if (keys['w']) player.y -= player.speed;
            if (keys['s']) player.y += player.speed;
            if (keys['a']) player.x -= player.speed;
            if (keys['d']) player.x += player.speed;

            // Shooting
            if (keys['ArrowUp']) shoot(0, -1);
            if (keys['ArrowDown']) shoot(0, 1);
            if (keys['ArrowLeft']) shoot(-1, 0);
            if (keys['ArrowRight']) shoot(1, 0);

            // Bounds
            player.x = Math.max(player.r, Math.min(width - player.r, player.x));
            player.y = Math.max(player.r, Math.min(height - player.r, player.y));

            // Bullets
            bullets.forEach((b, i) => {
                b.x += b.dx;
                b.y += b.dy;
                b.life--;
                if (b.life <= 0) bullets.splice(i, 1);
            });

            // Enemies
            enemies.forEach((e, i) => {
                const dx = player.x - e.x;
                const dy = player.y - e.y;
                const dist = Math.hypot(dx, dy);
                e.x += (dx / dist) * e.speed;
                e.y += (dy / dist) * e.speed;

                // Collision with player
                if (dist < player.r + e.r) {
                    player.hp -= 0.5;
                    healthEl.innerText = `HP: ${Math.floor(player.hp)}`;
                    if (player.hp <= 0) {
                        isGameOver = true;
                        gameOverEl.classList.remove('hidden');
                    }
                }

                // Collision with bullets
                bullets.forEach((b, bi) => {
                    if (Math.hypot(e.x - b.x, e.y - b.y) < e.r + 5) {
                        enemies.splice(i, 1);
                        bullets.splice(bi, 1);
                        score += 10;
                        scoreEl.innerText = `Score: ${score}`;
                    }
                });
            });
        }

        function draw() {
            ctx.clearRect(0, 0, width, height);

            // Draw Player
            ctx.fillStyle = '#fca5a5';
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.r, 0, Math.PI * 2);
            ctx.fill();

            // Draw Bullets
            ctx.fillStyle = '#60a5fa';
            bullets.forEach(b => {
                ctx.beginPath();
                ctx.arc(b.x, b.y, 5, 0, Math.PI * 2);
                ctx.fill();
            });

            // Draw Enemies
            ctx.fillStyle = '#4ade80';
            enemies.forEach(e => {
                ctx.beginPath();
                ctx.arc(e.x, e.y, e.r, 0, Math.PI * 2);
                ctx.fill();
            });

            requestAnimationFrame(() => {
                update();
                draw();
            });
        }

        draw();
    </script>
</body>
</html>
