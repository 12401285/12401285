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
  <img src="https://skillicons.dev/icons?i=python,php,laravel,html,css,js,mysql,git,github,vscode" />
</p>

---

## 📊 GitHub Stats
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GitHub Contributions Snake Game - Teclado e Mouse</title>
  <style>
    body {
      background-color: #0d1117;
      color: #c9d1d9;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      margin: 0;
      overflow: hidden;
    }

    h1 {
      margin-bottom: 5px;
      font-size: 24px;
    }

    #score {
      font-size: 18px;
      margin-bottom: 20px;
      color: #8b949e;
    }

    canvas {
      background-color: #161b22;
      border: 1px solid #30363d;
      border-radius: 6px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.5);
    }

    .instructions {
      margin-top: 15px;
      font-size: 14px;
      color: #8b949e;
      text-align: center;
      max-width: 500px;
      line-height: 1.5;
    }
  </style>
</head>

<body>

  <h1>GitHub Contribution Snake</h1>
  <div id="score">Pontuação: 0</div>
  <canvas id="gameCanvas" width="700" height="140"></canvas>
  <div class="instructions">
    Controle por <b>Teclado</b> (Setas) ou por <b>Movimento do Mouse</b>!<br>
        (Mova o mouse ao redor do tabuleiro para guiar a cabeça da cobra na direção desejada)
    </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const scoreElement = document.getElementById("score");

        const GRID_SIZE = 14; 
        const ROWS = canvas.height / GRID_SIZE; 
        const COLS = canvas.width / GRID_SIZE;  

        let snake = [{x: 10, y: 5}, {x: 9, y: 5}, {x: 8, y: 5}];
        let velocity = {x: 1, y: 0};
        let nextVelocity = {x: 1, y: 0}; // Evita inversões bruscas no mesmo ciclo
        let food = {x: 0, y: 0};
        let score = 0;
        let gameInterval;
        let gameOver = false;

        // Cores GitHub Dark Mode
        const COLOR_BG = "#161b22";
        const COLOR_SNAKE_HEAD = "#39d353"; 
        const COLOR_SNAKE_BODY = "#26a641"; 
        const COLOR_FOOD = "#0e4429";       

        function init() {
            placeFood();
            document.addEventListener("keydown", handleKeyDown);
            document.addEventListener("mousemove", handleMouseMove);
            gameInterval = setInterval(update, 130); 
        }

        function placeFood() {
            food.x = Math.floor(Math.random() * COLS);
            food.y = Math.floor(Math.random() * ROWS);
            
            for (let cell of snake) {
                if (cell.x === food.x && cell.y === food.y) {
                    placeFood();
                    break;
                }
            }
        }

        function update() {
            if (gameOver) return;

            velocity = nextVelocity;
            const head = {x: snake[0].x + velocity.x, y: snake[0].y + velocity.y};

            // Colisão com bordas
            if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS) {
                endGame();
                return;
            }

            // Colisão com o próprio corpo
            for (let cell of snake) {
                if (head.x === cell.x && head.y === cell.y) {
                    endGame();
                    return;
                }
            }

            snake.unshift(head);

            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreElement.innerText = "Pontuação: " + score;
                placeFood();
            } else {
                snake.pop();
            }

            draw();
        }

        function draw() {
            ctx.fillStyle = COLOR_BG;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = "#30363d";
            ctx.lineWidth = 1;
            for (let i = 0; i < COLS; i++) {
                for (let j = 0; j < ROWS; j++) {
                    ctx.strokeRect(i * GRID_SIZE, j * GRID_SIZE, GRID_SIZE, GRID_SIZE);
                }
            }

            ctx.fillStyle = COLOR_FOOD;
            ctx.fillRect(food.x * GRID_SIZE + 2, food.y * GRID_SIZE + 2, GRID_SIZE - 4, GRID_SIZE - 4);

            snake.forEach((cell, index) => {
                ctx.fillStyle = index === 0 ? COLOR_SNAKE_HEAD : COLOR_SNAKE_BODY;
                ctx.fillRect(cell.x * GRID_SIZE + 2, cell.y * GRID_SIZE + 2, GRID_SIZE - 4, GRID_SIZE - 4);
            });
        }

        // CAPTURA DO TECLADO
        function handleKeyDown(event) {
            const key = event.key;
            if(["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].includes(key)) {
                event.preventDefault();
            }

            if (gameOver) {
                resetGame();
                return;
            }

            switch (key) {
                case "ArrowUp":
                    if (velocity.y !== 1) { nextVelocity = {x: 0, y: -1}; }
                    break;
                case "ArrowDown":
                    if (velocity.y !== -1) { nextVelocity = {x: 0, y: 1}; }
                    break;
                case "ArrowLeft":
                    if (velocity.x !== 1) { nextVelocity = {x: -1, y: 0}; }
                    break;
                case "ArrowRight":
                    if (velocity.x !== -1) { nextVelocity = {x: 1, y: 0}; }
                    break;
            }
        }

        // CAPTURA DO MOVIMENTO DO MOUSE
        function handleMouseMove(event) {
            if (gameOver) return;

            // Obtém as coordenadas do canvas na tela
            const rect = canvas.getBoundingClientRect();
            
            // Posição do mouse em relação ao canvas
            const mouseX = event.clientX - rect.left;
            const mouseY = event.clientY - rect.top;

            // Posição atual da cabeça da cobra em pixels no canvas
            const headX = (snake[0].x * GRID_SIZE) + (GRID_SIZE / 2);
            const headY = (snake[0].y * GRID_SIZE) + (GRID_SIZE / 2);

            // Calcula a diferença de distância
            const diffX = mouseX - headX;
            const diffY = mouseY - headY;

            // Decide mudar a direção baseado na maior distância (Eixo X ou Eixo Y)
            if (Math.abs(diffX) > Math.abs(diffY)) {
                if (diffX > GRID_SIZE && velocity.x !== -1) {
                    nextVelocity = {x: 1, y: 0};  // Direita
                } else if (diffX < -GRID_SIZE && velocity.x !== 1) {
                    nextVelocity = {x: -1, y: 0}; // Esquerda
                }
            } else {
                if (diffY > GRID_SIZE && velocity.y !== -1) {
                    nextVelocity = {x: 0, y: 1};  // Baixo
                } else if (diffY < -GRID_SIZE && velocity.y !== 1) {
                    nextVelocity = {x: 0, y: -1}; // Cima
                }
            }
        }

        function endGame() {
            gameOver = true;
            clearInterval(gameInterval);
            
            ctx.fillStyle = "rgba(0, 0, 0, 0.8)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = "#f0f6fc";
            ctx.font = "bold 20px sans-serif";
            ctx.textAlign = "center";
            ctx.fillText("Fim de Jogo!", canvas.width / 2, canvas.height / 2 - 5);
            
            ctx.font = "14px sans-serif";
            ctx.fillStyle = "#8b949e";
            ctx.fillText("Clique ou pressione qualquer seta para reiniciar", canvas.width / 2, canvas.height / 2 + 20);
            
            // Permite reiniciar clicando com o mouse
            canvas.addEventListener("click", resetGame, { once: true });
        }

        function resetGame() {
            snake = [{x: 10, y: 5}, {x: 9, y: 5}, {x: 8, y: 5}];
            velocity = {x: 1, y: 0};
            nextVelocity = {x: 1, y: 0};
            score = 0;
            scoreElement.innerText = "Pontuação: " + score;
            gameOver = false;
            placeFood();
            gameInterval = setInterval(update, 130);
        }

        init();
  </script>
</body>

</html>


</div>

---

## 📫 Contato

<p align="center">

<a href="SEU_LINK_LINKEDIN">
<img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white"/>
</a>

<a href="SEU_LINK_INSTAGRAM">
<img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/>
</a>

</p>
