<h1 align="center">
  Olá! Eu sou Carlos Henrique 👋
</h1>

<p align="center">
  💻 Desenvolvedor em formação | COTEMIG
</p>

<p align="center">
  🐍 Python • PHP • Laravel • JavaScript
</p>

---

## 🚀 Tecnologias

<p align="center">
  <img src="https://shields.io"/>
  <img src="https://shields.io"/>
  <img src="https://shields.io"/>
  <img src="https://shields.io"/>
</p>

---

## 🕹️ GitHub Snake: OVERDRIVE EDITION

Se você baixou este repositório, abra este arquivo em um navegador para jogar a versão arcade com áudio e partículas!

<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    .game-body {
      background-color: #050508;
      color: #00ff66;
      font-family: 'Courier New', Courier, monospace;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 20px;
      border-radius: 8px;
      margin: 20px 0;
      border: 1px solid #30363d;
      text-shadow: 0 0 5px rgba(0, 255, 102, 0.5);
    }
    .game-container {
      text-align: center;
      position: relative;
    }
    .game-title {
      margin: 0 0 5px 0;
      font-size: 28px;
      letter-spacing: 3px;
      color: #00ff66;
    }
    #hud {
      display: flex;
      justify-content: space-between;
      width: 700px;
      font-size: 18px;
      margin-bottom: 10px;
      font-weight: bold;
      border-bottom: 2px solid #30363d;
      padding-bottom: 5px;
    }
    .combo-box {
      color: #ff0055;
      text-shadow: 0 0 8px #ff0055;
    }
    #gameCanvas {
      background-color: #0d1117;
      border: 3px solid #00ff66;
      border-radius: 8px;
      box-shadow: 0 0 30px rgba(0, 255, 102, 0.3);
      cursor: none;
    }
    .instructions {
      margin-top: 15px;
      font-size: 13px;
      color: #8b949e;
    }
    #start-btn {
      background: #00ff66;
      color: #000;
      border: none;
      padding: 10px 20px;
      font-family: inherit;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
      border-radius: 4px;
      box-shadow: 0 0 15px #00ff66;
      margin-bottom: 15px;
    }
    #start-btn:hover {
      background: #ffffff;
      box-shadow: 0 0 25px #ffffff;
    }
  </style>
</head>
<body>

  <div class="game-body">
    <div class="game-container">
        <h2 class="game-title">SNAKE // OVERDRIVE</h2>
        <button id="start-btn" onclick="startAudioAndGame()">INICIAR SISTEMA (ÁUDIO ON)</button>
        
        <div id="hud">
            <div>SCORE: <span id="score">0000</span></div>
            <div class="combo-box">COMBO: X<span id="combo">1.0</span></div>
            <div>SPEED: <span id="speed-val">100%</span></div>
        </div>
        
        <canvas id="gameCanvas" width="700" height="140"></canvas>
        <div class="instructions">CONTROLES: MOUSE CORTA O ESPAÇO // SETAS SE PREFERIR ALTA PRECISÃO</div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreElement = document.getElementById("score");
    const comboElement = document.getElementById("combo");
    const speedElement = document.getElementById("speed-val");
    const startBtn = document.getElementById("start-btn");

    const GRID_SIZE = 14; 
    const ROWS = canvas.height / GRID_SIZE; 
    const COLS = canvas.width / GRID_SIZE;  

    let snake = [];
    let velocity = {x: 1, y: 0};
    let nextVelocity = {x: 1, y: 0};
    let food = {x: 0, y: 0};
    let score = 0;
    let combo = 1.0;
    let baseSpeed = 100;
    let currentSpeed = baseSpeed;
    let gameInterval;
    let gameOver = false;
    let particles = [];
    let comboTimeout;
    let mousePos = { x: 0, y: 0 };
    let audioCtx = null;

    function startAudioAndGame() {
        if(!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
        startBtn.style.display = 'none';
        resetGame();
    }

    function playSound(type) {
        if (!audioCtx) return;
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.connect(gain);
        gain.connect(audioCtx.destination);

        if (type === 'eat') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(150 + (combo * 50), audioCtx.currentTime);
            osc.frequency.exponentialRampToValueAtTime(600 + (combo * 100), audioCtx.currentTime + 0.15);
            gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
            gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.15);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.15);
        } else if (type === 'die') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(300, audioCtx.currentTime);
            osc.frequency.linearRampToValueAtTime(40, audioCtx.currentTime + 0.5);
            gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gain.gain.linearRampToValueAtTime(0, audioCtx.currentTime + 0.5);
            osc.start();
            osc.stop(audioCtx.currentTime + 0.5);
        }
    }

    function init() {
        document.addEventListener("keydown", handleKeyDown);
        document.addEventListener("mousemove", handleMouseMove);
        ctx.fillStyle = "#0d1117";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    function placeFood() {
        food.x = Math.floor(Math.random() * COLS);
        food.y = Math.floor(Math.random() * ROWS);
        for (let cell of snake) {
            if (cell.x === food.x && cell.y === food.y) { placeFood(); break; }
        }
    }

    function createParticles(x, y, color) {
        for (let i = 0; i < 15; i++) {
            particles.push({
                x: x * GRID_SIZE + GRID_SIZE/2,
                y: y * GRID_SIZE + GRID_SIZE/2,
                vx: (Math.random() - 0.5) * 8,
                vy: (Math.random() - 0.5) * 8,
                alpha: 1,
                color: color
            });
        }
    }

    function update() {
        if (gameOver) return;

        velocity = nextVelocity;
        const head = {x: snake[0].x + velocity.x, y: snake[0].y + velocity.y};

        if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS) {
            endGame(); return;
        }

        for (let cell of snake) {
            if (head.x === cell.x && head.y === cell.y) {
                endGame(); return;
            }
        }

        snake.unshift(head);

        if (head.x === food.x && head.y === food.y) {
            score += Math.floor(10 * combo);
            scoreElement.innerText = String(score).padStart(4, '0');
            
            createParticles(food.x, food.y, "#ff0055");
            playSound('eat');

            combo = parseFloat((combo + 0.2).toFixed(1));
            comboElement.innerText = combo.toFixed(1);
            
            clearTimeout(comboTimeout);
            comboTimeout = setTimeout(() => {
                combo = 1.0;
                comboElement.innerText = "1.0";
            }, 4000);

            currentSpeed = Math.max(50, baseSpeed - (combo * 8));
            speedElement.innerText = Math.floor((baseSpeed / currentSpeed) * 100) + "%";
            
            clearInterval(gameInterval);
            gameInterval = setInterval(update, currentSpeed);

            placeFood();
        } else {
            snake.pop();
        }

        particles.forEach((p, idx) => {
            p.x += p.vx;
            p.y += p.vy;
            p.alpha -= 0.04;
            if (p.alpha <= 0) particles.splice(idx, 1);
        });

        draw();
    }

    function draw() {
        ctx.fillStyle = "rgba(13, 17, 23, 0.3)"; 
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.strokeStyle = "rgba(48, 54, 61, 0.2)";
        ctx.lineWidth = 1;
        for (let i = 0; i < COLS; i++) {
            for (let j = 0; j < ROWS; j++) {
                ctx.strokeRect(i * GRID_SIZE, j * GRID_SIZE, GRID_SIZE, GRID_SIZE);
            }
        }

        ctx.shadowBlur = 15;
        ctx.shadowColor = "#ff0055";
        ctx.fillStyle = "#ff0055";
        ctx.fillRect(food.x * GRID_SIZE + 2, food.y * GRID_SIZE + 2, GRID_SIZE - 4, GRID_SIZE - 4);

        snake.forEach((cell, index) => {
            ctx.shadowBlur = index === 0 ? 20 : 5;
            ctx.shadowColor = index === 0 ? "#00ff66" : "#26a641";
            ctx.fillStyle = index === 0 ? "#ffffff" : `hsl(135, 100%, ${Math.max(30, 60 - index * 2)}%)`;
            ctx.fillRect(cell.x * GRID_SIZE + 1, cell.y * GRID_SIZE + 1, GRID_SIZE - 2, GRID_SIZE - 2);
        });

        ctx.shadowBlur = 0;

        particles.forEach(p => {
            ctx.fillStyle = p.color;
            ctx.globalAlpha = p.alpha;
            ctx.fillRect(p.x, p.y, 3, 3);
        });
        ctx.globalAlpha = 1;

        ctx.strokeStyle = "rgba(0, 255, 102, 0.4)";
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.arc(mousePos.x, mousePos.y, 8, 0, Math.PI * 2);
        ctx.stroke();
    }

    function handleKeyDown(event) {
        const key = event.key;
        if(["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(key)) event.preventDefault();

        if (gameOver) { startAudioAndGame(); return; }

        switch (key) {
            case "ArrowUp": if (velocity.y !== 1) nextVelocity = {x: 0, y: -1}; break;
            case "ArrowDown": if (velocity.y !== -1) nextVelocity = {x: 0, y: 1}; break;
            case "ArrowLeft": if (velocity.x !== 1) nextVelocity = {x: -1, y: 0}; break;
