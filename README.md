<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pong Game</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            font-family: 'Arial', sans-serif;
        }

        .container {
            text-align: center;
        }

        h1 {
            color: #fff;
            margin-bottom: 20px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .scoreboard {
            display: flex;
            justify-content: space-around;
            background: rgba(0, 0, 0, 0.3);
            padding: 15px 30px;
            border-radius: 10px;
            margin-bottom: 20px;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
        }

        .score-item {
            color: #fff;
            font-size: 1.5em;
        }

        .score-item p {
            font-size: 0.8em;
            opacity: 0.8;
            margin-top: 5px;
        }

        #gameCanvas {
            display: block;
            background-color: #000;
            border: 3px solid #fff;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.3);
            margin: 0 auto;
        }

        .instructions {
            color: #fff;
            margin-top: 20px;
            font-size: 0.95em;
            background: rgba(0, 0, 0, 0.3);
            padding: 15px;
            border-radius: 10px;
            max-width: 800px;
            margin-left: auto;
            margin-right: auto;
            line-height: 1.6;
        }

        .instructions h3 {
            margin-bottom: 10px;
            color: #4db8ff;
        }

        .instructions p {
            margin: 5px 0;
        }

        .status {
            color: #4db8ff;
            margin-top: 15px;
            font-size: 1.1em;
            height: 30px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎮 PONG GAME</h1>
        
        <div class="scoreboard">
            <div class="score-item">
                <div id="playerScore">0</div>
                <p>Player (Left)</p>
            </div>
            <div class="score-item">
                <div id="computerScore">0</div>
                <p>Computer (Right)</p>
            </div>
        </div>

        <canvas id="gameCanvas" width="800" height="400"></canvas>

        <div class="status" id="status">Press SPACE to start</div>

        <div class="instructions">
            <h3>How to Play:</h3>
            <p><strong>Controls:</strong> Move your paddle with ARROW KEYS (↑/↓) or MOUSE</p>
            <p><strong>Goal:</strong> Bounce the ball past the computer to score points</p>
            <p><strong>First to 11 wins!</strong></p>
            <p style="margin-top: 10px; color: #ffaa00;"><strong>SPACE</strong> to start/restart | <strong>P</strong> to pause</p>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const statusDiv = document.getElementById('status');

        // Game objects
        const paddleWidth = 10;
        const paddleHeight = 80;
        const ballSize = 8;

        // Player paddle (left)
        const player = {
            x: 15,
            y: canvas.height / 2 - paddleHeight / 2,
            width: paddleWidth,
            height: paddleHeight,
            dy: 0,
            speed: 6
        };

        // Computer paddle (right)
        const computer = {
            x: canvas.width - paddleWidth - 15,
            y: canvas.height / 2 - paddleHeight / 2,
            width: paddleWidth,
            height: paddleHeight,
            dy: 0,
            speed: 5.5
        };

        // Ball
        const ball = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            dx: 5,
            dy: 5,
            radius: ballSize,
            speed: 5
        };

        // Score
        let playerScore = 0;
        let computerScore = 0;

        // Game state
        let gameRunning = false;
        let gamePaused = false;

        // Keyboard input
        const keys = {};

        // Mouse input
        let mouseY = canvas.height / 2;

        // Event listeners
        document.addEventListener('keydown', (e) => {
            keys[e.key] = true;

            if (e.key === ' ') {
                e.preventDefault();
                if (!gameRunning) {
                    startGame();
                } else if (gamePaused) {
                    resumeGame();
                }
            }

            if (e.key === 'p' || e.key === 'P') {
                if (gameRunning && !gamePaused) {
                    pauseGame();
                }
            }
        });

        document.addEventListener('keyup', (e) => {
            keys[e.key] = false;
        });

        document.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            mouseY = e.clientY - rect.top;
        });

        function startGame() {
            gameRunning = true;
            gamePaused = false;
            resetBall();
            statusDiv.textContent = 'Game Running... (P to pause)';
            gameLoop();
        }

        function pauseGame() {
            gamePaused = true;
            statusDiv.textContent = 'Paused (SPACE to resume)';
        }

        function resumeGame() {
            gamePaused = false;
            statusDiv.textContent = 'Game Running... (P to pause)';
            gameLoop();
        }

        function resetBall() {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.dx = (Math.random() > 0.5 ? 1 : -1) * ball.speed;
            ball.dy = (Math.random() - 0.5) * ball.speed;
        }

        function update() {
            if (!gameRunning || gamePaused) return;

            // Update player paddle with keyboard
            if (keys['ArrowUp'] && player.y > 0) {
                player.y -= player.speed;
            }
            if (keys['ArrowDown'] && player.y < canvas.height - player.height) {
                player.y += player.speed;
            }

            // Update player paddle with mouse
            if (mouseY > 0 && mouseY < canvas.height) {
                const targetY = mouseY - player.height / 2;
                if (targetY < 0) {
                    player.y = 0;
                } else if (targetY > canvas.height - player.height) {
                    player.y = canvas.height - player.height;
                } else if (keys['ArrowUp'] || keys['ArrowDown']) {
                    // Keyboard takes priority
                } else {
                    player.y = targetY;
                }
            }

            // Computer AI - track the ball
            const computerCenter = computer.y + computer.height / 2;
            const ballCenter = ball.y;
            const difficulty = 0.8; // Lower = easier

            if (computerCenter < ballCenter - 35 && computer.y < canvas.height - computer.height) {
                computer.y += computer.speed * difficulty;
            } else if (computerCenter > ballCenter + 35 && computer.y > 0) {
                computer.y -= computer.speed * difficulty;
            }

            // Ball movement
            ball.x += ball.dx;
            ball.y += ball.dy;

            // Ball collision with top and bottom walls
            if (ball.y - ball.radius <= 0 || ball.y + ball.radius >= canvas.height) {
                ball.dy = -ball.dy;
                ball.y = Math.max(ball.radius, Math.min(canvas.height - ball.radius, ball.y));
            }

            // Ball collision with paddles
            if (
                ball.x - ball.radius <= player.x + player.width &&
                ball.y >= player.y &&
                ball.y <= player.y + player.height
            ) {
                ball.dx = -ball.dx;
                ball.x = player.x + player.width + ball.radius;
                // Add spin based on where the ball hits the paddle
                ball.dy += (ball.y - (player.y + player.height / 2)) * 0.1;
            }

            if (
                ball.x + ball.radius >= computer.x &&
                ball.y >= computer.y &&
                ball.y <= computer.y + computer.height
            ) {
                ball.dx = -ball.dx;
                ball.x = computer.x - ball.radius;
                // Add spin based on where the ball hits the paddle
                ball.dy += (ball.y - (computer.y + computer.height / 2)) * 0.1;
            }

            // Ball goes off screen - score!
            if (ball.x - ball.radius < 0) {
                computerScore++;
                document.getElementById('computerScore').textContent = computerScore;
                checkWin();
                resetBall();
            }

            if (ball.x + ball.radius > canvas.width) {
                playerScore++;
                document.getElementById('playerScore').textContent = playerScore;
                checkWin();
                resetBall();
            }

            // Cap ball speed to prevent it from going too fast
            const maxSpeed = 8;
            const currentSpeed = Math.sqrt(ball.dx ** 2 + ball.dy ** 2);
            if (currentSpeed > maxSpeed) {
                ball.dx = (ball.dx / currentSpeed) * maxSpeed;
                ball.dy = (ball.dy / currentSpeed) * maxSpeed;
            }
        }

        function checkWin() {
            if (playerScore === 11) {
                statusDiv.textContent = '🎉 You Win! Press SPACE to play again';
                gameRunning = false;
                playerScore = 0;
                computerScore = 0;
                document.getElementById('playerScore').textContent = '0';
                document.getElementById('computerScore').textContent = '0';
            } else if (computerScore === 11) {
                statusDiv.textContent = '💻 Computer Wins! Press SPACE to play again';
                gameRunning = false;
                playerScore = 0;
                computerScore = 0;
                document.getElementById('playerScore').textContent = '0';
                document.getElementById('computerScore').textContent = '0';
            }
        }

        function draw() {
            // Clear canvas
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw center line
            ctx.strokeStyle = '#fff';
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(canvas.width / 2, 0);
            ctx.lineTo(canvas.width / 2, canvas.height);
            ctx.stroke();
            ctx.setLineDash([]);

            // Draw paddles
            ctx.fillStyle = '#fff';
            ctx.fillRect(player.x, player.y, player.width, player.height);
            ctx.fillRect(computer.x, computer.y, computer.width, computer.height);

            // Draw ball
            ctx.fillStyle = '#ffff00';
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fill();

            // Draw ball glow effect
            ctx.strokeStyle = 'rgba(255, 255, 0, 0.5)';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius + 3, 0, Math.PI * 2);
            ctx.stroke();
        }

        function gameLoop() {
            update();
            draw();

            if (gameRunning) {
                requestAnimationFrame(gameLoop);
            }
        }

        function animate() {
            draw();
            if (gameRunning) {
                requestAnimationFrame(animate);
            } else {
                requestAnimationFrame(animate);
            }
        }

        // Initial draw
        draw();
        requestAnimationFrame(animate);
    </script>
</body>
</html>
