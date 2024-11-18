---



index.html


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Avoidor Game</title>
  <link rel="stylesheet" href="css/style.css">
</head>
<body>
  <div id="gameContainer">
    <canvas id="gameCanvas"></canvas>
    <div id="score">Score: 0</div>
    <div id="highScore">High Score: 0</div>
    <div id="gameOver" class="hidden">
      <h1>Game Over</h1>
      <p>Your Final Score: <span id="finalScore"></span></p>
      <button onclick="restartGame()">Play Again</button>
    </div>
  </div>
  <audio id="coinSound" src="../assets/sounds/coin.mp3"></audio>
  <audio id="collisionSound" src="../assets/sounds/collision.mp3"></audio>
  <audio id="gameOverSound" src="../assets/sounds/gameOver.mp3"></audio>
  <script src="js/game.js"></script>
</body>
</html>


---

css/style.css

Include a style for the high score display.

#highScore {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 24px;
  z-index: 10;
}


---

js/game.js

Here’s the updated logic with all the new features added:

const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

canvas.width = 400;
canvas.height = 600;

// Game Variables
let player = { x: 180, y: 500, width: 40, height: 60, speed: 5 };
let obstacles = [];
let coins = [];
let powerUps = [];
let score = 0;
let highScore = localStorage.getItem("highScore") || 0;
let gameSpeed = 2;
let isGameOver = false;

// Load Images and Sounds
const playerImg = new Image();
const carImg = new Image();
const coinImg = new Image();
const powerUpImg = new Image();
playerImg.src = "../assets/images/player.png";
carImg.src = "../assets/images/car.png";
coinImg.src = "../assets/images/coin.png";
powerUpImg.src = "../assets/images/powerup.png";

const coinSound = document.getElementById("coinSound");
const collisionSound = document.getElementById("collisionSound");
const gameOverSound = document.getElementById("gameOverSound");

// Input Control
let keys = {};
document.addEventListener("keydown", (e) => (keys[e.key] = true));
document.addEventListener("keyup", (e) => (keys[e.key] = false));

// Utility Functions
function randomX(width) {
  return Math.random() * (canvas.width - width);
}

function detectCollision(a, b) {
  return (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  );
}

// Game Functions
function spawnObstacle() {
  const interval = Math.max(800 - gameSpeed * 20, 300); // Faster spawn over time
  obstacles.push({ x: randomX(40), y: -60, width: 40, height: 60 });
  if (!isGameOver) setTimeout(spawnObstacle, interval);
}

function spawnCoin() {
  const interval = Math.max(1500 - gameSpeed * 50, 500);
  coins.push({ x: randomX(20), y: -30, width: 20, height: 20 });
  if (!isGameOver) setTimeout(spawnCoin, interval);
}

function spawnPowerUp() {
  const interval = 10000 + Math.random() * 10000; // Rare spawn
  powerUps.push({ x: randomX(30), y: -30, width: 30, height: 30 });
  if (!isGameOver) setTimeout(spawnPowerUp, interval);
}

function drawPlayer() {
  ctx.drawImage(playerImg, player.x, player.y, player.width, player.height);
}

function drawObstacles() {
  obstacles.forEach((obstacle, index) => {
    obstacle.y += gameSpeed;
    ctx.drawImage(carImg, obstacle.x, obstacle.y, obstacle.width, obstacle.height);
    if (obstacle.y > canvas.height) obstacles.splice(index, 1);
  });
}

function drawCoins() {
  coins.forEach((coin, index) => {
    coin.y += gameSpeed;
    ctx.drawImage(coinImg, coin.x, coin.y, coin.width, coin.height);
    if (coin.y > canvas.height) coins.splice(index, 1);
  });
}

function drawPowerUps() {
  powerUps.forEach((powerUp, index) => {
    powerUp.y += gameSpeed;
    ctx.drawImage(powerUpImg, powerUp.x, powerUp.y, powerUp.width, powerUp.height);
    if (powerUp.y > canvas.height) powerUps.splice(index, 1);
  });
}

function updateGame() {
  // Move Player
  if (keys["ArrowLeft"] && player.x > 0) player.x -= player.speed;
  if (keys["ArrowRight"] && player.x < canvas.width - player.width)
    player.x += player.speed;
  if (keys["ArrowUp"] && player.y > 0) player.y -= player.speed;
  if (keys["ArrowDown"] && player.y < canvas.height - player.height)
    player.y += player.speed;

  // Collision Detection
  obstacles.forEach((obstacle) => {
    if (detectCollision(player, obstacle)) {
      collisionSound.play();
      endGame();
    }
  });

  coins.forEach((coin, index) => {
    if (detectCollision(player, coin)) {
      score += 10;
      coins.splice(index, 1);
      coinSound.play();
      updateScore();
    }
  });

  powerUps.forEach((powerUp, index) => {
    if (detectCollision(player, powerUp)) {
      powerUps.splice(index, 1);
      score += 50; // Bonus score
      updateScore();
    }
  });

  // Increase Difficulty
  gameSpeed += 0.001;
}

function updateScore() {
  document.getElementById("score").innerText = `Score: ${score}`;
  if (score > highScore) {
    highScore = score;
    localStorage.setItem("highScore", highScore);
    document.getElementById("highScore").innerText = `High Score: ${highScore}`;
  }
}

function drawGame() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawPlayer();
  drawObstacles();
  drawCoins();
  drawPowerUps();
}

function gameLoop() {
  if (!isGameOver) {
    updateGame();
    drawGame();
    requestAnimationFrame(gameLoop);
  }
}

function startGame() {
  obstacles = [];
  coins = [];
  powerUps = [];
  score = 0;
  gameSpeed = 2;
  isGameOver = false;

  document.getElementById("gameOver").classList.add("hidden");
  document.getElementById("score").innerText = "Score: 0";
  document.getElementById("highScore").innerText = `High Score: ${highScore}`;

  spawnObstacle();
  spawnCoin();
  spawnPowerUp();
  gameLoop();
}

function endGame() {
  isGameOver = true;
  gameOverSound.play();
  document.getElementById("finalScore").innerText = score;
  document.getElementById("gameOver").classList.remove("hidden");
}

function restartGame() {
  startGame();
}


---
