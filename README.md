# SoniSnake
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Yüksek Çözünürlüklü Yılan Oyunu</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            background-image: linear-gradient(
                rgba(255, 255, 255, 0.7),
                rgba(255, 255, 255, 0.7)
            ),
            url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200"><text x="10" y="20" font-family="Arial" font-size="20" fill="black" opacity="0.1">SONER KARA YILAN OYUNU</text></svg>');
            background-repeat: repeat;
            background-size: 200px 200px;
        }
        #gameContainer {
            text-align: center;
        }
        canvas {
            border: 2px solid #333;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        #score {
            font-size: 24px;
            margin-top: 20px;
        }
        .controls {
            display: flex;
            justify-content: center;
            margin-top: 20px;
        }
        .controls div {
            display: flex;
            flex-direction: column;
        }
        .controls button {
            width: 50px;
            height: 50px;
            margin: 5px;
            font-size: 18px;
            background-color: #4caf50;
            border: none;
            border-radius: 5px;
            color: white;
            cursor: pointer;
        }
        #gameOver {
            display: none;
            font-size: 36px;
            color: red;
            margin-top: 20px;
        }
        #restartButton {
            display: none;
            font-size: 24px;
            background-color: #f44336;
            color: white;
            border: none;
            border-radius: 5px;
            padding: 10px 20px;
            cursor: pointer;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="800"></canvas>
        <div id="score">Skor: 0</div>
        <div id="gameOver">KAYBETTİN KIZIM BAŞTAN BAŞLA</div>
        <button id="restartButton" onclick="startGame()">TEKRAR OYNA</button>
    </div>
    <div class="controls">
        <div>
            <button id="up">↑</button>
        </div>
        <div>
            <button id="left">←</button>
            <button id="down">↓</button>
            <button id="right">→</button>
        </div>
    </div>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('gameOver');
        const restartButton = document.getElementById('restartButton');
        
        const upButton = document.getElementById('up');
        const downButton = document.getElementById('down');
        const leftButton = document.getElementById('left');
        const rightButton = document.getElementById('right');

        const gridSize = 20;
        const tileCount = canvas.width / gridSize;

        let snake = [{ x: 10, y: 10 }];
        let food = { x: 15, y: 15 };
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameInterval;

        function drawGame() {
            clearCanvas();
            moveSnake();
            drawSnake();
            drawFood();
            checkCollision();
            updateScore();
        }

        function clearCanvas() {
            ctx.fillStyle = '#e8f5e9';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        function moveSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };

            if (head.x < 0) head.x = tileCount - 1;
            if (head.x >= tileCount) head.x = 0;
            if (head.y < 0) head.y = tileCount - 1;
            if (head.y >= tileCount) head.y = 0;

            snake.unshift(head);

            if (head.x === food.x && head.y === food.y) {
                score++;
                generateFood();
            } else {
                snake.pop();
            }
        }

        function drawSnake() {
            snake.forEach((segment, index) => {
                const gradient = ctx.createRadialGradient(
                    (segment.x + 0.5) * gridSize, (segment.y + 0.5) * gridSize, 2,
                    (segment.x + 0.5) * gridSize, (segment.y + 0.5) * gridSize, gridSize / 2
                );
                gradient.addColorStop(0, '#4caf50');
                gradient.addColorStop(1, '#2e7d32');

                ctx.fillStyle = gradient;
                ctx.beginPath();
                ctx.arc((segment.x + 0.5) * gridSize, (segment.y + 0.5) * gridSize, gridSize / 2 - 1, 0, 2 * Math.PI);
                ctx.fill();

                if (index === 0) {
                    // Gözler
                    ctx.fillStyle = 'white';
                    ctx.beginPath();
                    ctx.arc((segment.x + 0.3) * gridSize, (segment.y + 0.3) * gridSize, gridSize / 10, 0, 2 * Math.PI);
                    ctx.arc((segment.x + 0.7) * gridSize, (segment.y + 0.3) * gridSize, gridSize / 10, 0, 2 * Math.PI);
                    ctx.fill();
                    ctx.fillStyle = 'black';
                    ctx.beginPath();
                    ctx.arc((segment.x + 0.3) * gridSize, (segment.y + 0.3) * gridSize, gridSize / 20, 0, 2 * Math.PI);
                    ctx.arc((segment.x + 0.7) * gridSize, (segment.y + 0.3) * gridSize, gridSize / 20, 0, 2 * Math.PI);
                    ctx.fill();
                }
            });
        }

        function drawFood() {
            const gradient = ctx.createRadialGradient(
                (food.x + 0.5) * gridSize, (food.y + 0.5) * gridSize, 2,
                (food.x + 0.5) * gridSize, (food.y + 0.5) * gridSize, gridSize / 2
            );
            gradient.addColorStop(0, '#ff5722');
            gradient.addColorStop(1, '#e64a19');

            ctx.fillStyle = gradient;
            ctx.beginPath();
            ctx.arc((food.x + 0.5) * gridSize, (food.y + 0.5) * gridSize, gridSize / 2 - 1, 0, 2 * Math.PI);
            ctx.fill();
        }

        function generateFood() {
            food.x = Math.floor(Math.random() * tileCount);
            food.y = Math.floor(Math.random() * tileCount);
        }

        function checkCollision() {
            const head = snake[0];

            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    endGame();
                    return;
                }
            }
        }

        function endGame() {
            clearInterval(gameInterval);
            gameOverElement.style.display = 'block';
            restartButton.style.display = 'block';
        }

        function resetGame() {
            snake = [{ x: 10, y: 10 }];
            dx = 0;
            dy = 0;
            score = 0;
            gameOverElement.style.display = 'none';
            restartButton.style.display = 'none';
            generateFood();
            startGame();
        }

        function updateScore() {
            scoreElement.textContent = `Skor: ${score}`;
        }

        function startGame() {
            gameInterval = setInterval(drawGame, 100);
        }

        document.addEventListener('keydown', (e) => {
            switch (e.key) {
                case 'ArrowUp':
                    if (dy === 0) { dx = 0; dy = -1; }
                    break;
                case 'ArrowDown':
                    if (dy === 0) { dx = 0; dy = 1; }
                    break;
                case 'ArrowLeft':
                    if (dx === 0) { dx = -1; dy = 0; }
                    break;
                case 'ArrowRight':
                    if (dx === 0) { dx = 1; dy = 0; }
                    break;
            }
        });

        upButton.addEventListener('click', () => {
            if (dy === 0) { dx = 0; dy = -1; }
        });
        downButton.addEventListener('click', () => {
            if (dy === 0) { dx = 0; dy = 1; }
        });
        leftButton.addEventListener('click', () => {
            if (dx === 0) { dx = -1; dy = 0; }
        });
        rightButton.addEventListener('click', () => {
            if (dx === 0) { dx = 1; dy = 0; }
        });

        startGame();
    </script>
</body>
</html>
