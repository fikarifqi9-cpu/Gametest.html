<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Bird XL - Lompatan Tinggi</title>
    <style>
        body { margin: 0; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; background: #1a1a1a; font-family: 'Arial', sans-serif; }
        #game-container { position: relative; border: 5px solid #333; border-radius: 12px; box-shadow: 0 0 50px rgba(0,0,0,0.5); overflow: hidden; }
        canvas { background: #70c5ce; display: block; }
        
        .overlay {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.5);
            display: flex; flex-direction: column; justify-content: center; align-items: center;
            color: white; text-align: center; z-index: 20;
        }
        
        button {
            padding: 18px 50px; font-size: 24px; cursor: pointer;
            background: #2ecc71; border: none; border-radius: 10px;
            font-weight: bold; color: white; transition: 0.3s;
            box-shadow: 0 6px #27ae60; text-transform: uppercase;
        }
        button:hover { background: #27ae60; transform: scale(1.05); }
        button:active { transform: translateY(4px); box-shadow: 0 2px #27ae60; }

        .score-tag { position: absolute; top: 30px; width: 100%; text-align: center; 
                     color: white; font-size: 60px; font-weight: bold; 
                     text-shadow: 4px 4px 0 #000; pointer-events: none; z-index: 10; }
        .hidden { display: none !important; }
    </style>
</head>
<body>

    <div id="game-container">
        <div id="scoreDisplay" class="score-tag hidden">0</div>
        
        <div id="lobby" class="overlay">
            <h1 style="font-size: 50px; margin-bottom: 10px; color: #f1c40f; text-shadow: 3px 3px #000;">FLAPPY XL</h1>
            <p style="margin-bottom: 30px; font-size: 18px;">Layar Besar & Lompatan Tinggi</p>
            <button onclick="startGame()">MULAI GAME</button>
        </div>

        <div id="gameOver" class="overlay hidden">
            <h2 style="font-size: 40px; color: #ff4757;">YAH, MATI!</h2>
            <h3 id="finalScore" style="margin-bottom: 30px; font-size: 24px;">Skor: 0</h3>
            <button onclick="startGame()">COBA LAGI</button>
        </div>

        <canvas id="gameCanvas"></canvas>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const lobby = document.getElementById("lobby");
        const gameOverScreen = document.getElementById("gameOver");
        const scoreDisplay = document.getElementById("scoreDisplay");
        const finalScoreText = document.getElementById("finalScore");

        // --- Ukuran Layar Lebih Besar ---
        canvas.width = 450; 
        canvas.height = 700;

        // --- Variabel Game ---
        let bird, pipes, frame, score, gameRunning;

        function initVars() {
            // lift diubah ke -9 (lebih tinggi dari sebelumnya yang -7)
            bird = { x: 70, y: 300, w: 34, h: 26, gravity: 0.5, lift: -9, velocity: 0 };
            pipes = [];
            frame = 0;
            score = 0;
            gameRunning = false;
        }

        function startGame() {
            initVars();
            gameRunning = true;
            lobby.classList.add("hidden");
            gameOverScreen.classList.add("hidden");
            scoreDisplay.classList.remove("hidden");
            scoreDisplay.innerText = "0";
            requestAnimationFrame(gameLoop);
        }

        function flap() {
            if (gameRunning) bird.velocity = bird.lift;
        }

        // Input
        window.addEventListener("keydown", (e) => { if (e.code === "Space" || e.code === "ArrowUp") flap(); });
        canvas.addEventListener("mousedown", flap);
        canvas.addEventListener("touchstart", (e) => { e.preventDefault(); flap(); });

        function createPipe() {
            let pWidth = 85; // Pipa tetap besar/lebar
            let pGap = 160;   // Celah sedikit lebih lebar untuk layar besar
            let minH = 80;
            let maxH = canvas.height - pGap - minH;
            let topH = Math.floor(Math.random() * (maxH - minH + 1)) + minH;
            pipes.push({ x: canvas.width, top: topH, width: pWidth, gap: pGap, passed: false });
        }

        function update() {
            bird.velocity += bird.gravity;
            bird.y += bird.velocity;

            if (bird.y + bird.h > canvas.height || bird.y < 0) endGame();

            if (frame % 85 === 0) createPipe();

            for (let i = pipes.length - 1; i >= 0; i--) {
                pipes[i].x -= 3.2; // Sedikit lebih cepat mengikuti skala layar

                // Deteksi Tabrakan
                if (
                    bird.x < pipes[i].x + pipes[i].width &&
                    bird.x + bird.w > pipes[i].x &&
                    (bird.y < pipes[i].top || bird.y + bird.h > pipes[i].top + pipes[i].gap)
                ) {
                    endGame();
                }

                if (!pipes[i].passed && pipes[i].x < bird.x) {
                    score++;
                    pipes[i].passed = true;
                    scoreDisplay.innerText = score;
                }

                if (pipes[i].x + pipes[i].width < 0) pipes.splice(i, 1);
            }
            frame++;
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Gambar Burung (Kuning Cerah)
            ctx.fillStyle = "#fffa65";
            ctx.strokeStyle = "#000";
            ctx.lineWidth = 2;
            ctx.fillRect(bird.x, bird.y, bird.w, bird.h);
            ctx.strokeRect(bird.x, bird.y, bird.w, bird.h);
            // Mata
            ctx.fillStyle = "white";
            ctx.fillRect(bird.x + 22, bird.y + 4, 8, 8);
            ctx.fillStyle = "black";
            ctx.fillRect(bird.x + 26, bird.y + 6, 4, 4);

            // Gambar Pipa Besar
            pipes.forEach(p => {
                ctx.fillStyle = "#2ecc71";
                ctx.strokeStyle = "#27ae60";
                ctx.lineWidth = 4;
                
                // Pipa Atas
                ctx.fillRect(p.x, 0, p.width, p.top);
                ctx.strokeRect(p.x, -5, p.width, p.top + 5);
                
                // Pipa Bawah
                let bY = p.top + p.gap;
                let bH = canvas.height - bY;
                ctx.fillRect(p.x, bY, p.width, bH);
                ctx.strokeRect(p.x, bY, p.width, bH + 5);
            });
        }

        function endGame() {
            gameRunning = false;
            finalScoreText.innerText = "Skor Kamu: " + score;
            gameOverScreen.classList.remove("hidden");
        }

        function gameLoop() {
            if (!gameRunning) return;
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        initVars(); // Inisialisasi awal
    </script>
</body>
</html>
