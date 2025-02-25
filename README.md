# snake-game-advanced


```html
<!DOCTYPE html>
<html>
<head>
    <title>Advanced Snake Game</title>
</head>
<body>
<script>
// Advanced Snake Game with Super Food and Enemy Snake
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
canvas.width = 600;
canvas.height = 400;
canvas.style.border = '1px solid black';
canvas.style.backgroundColor = '#f0f0f0';
document.body.appendChild(canvas);

// Game variables
const gridSize = 20;
const initialSpeed = 150;
let gameSpeed = initialSpeed;
let gameRunning = false;
let gameOver = false;
let score = 0;
let highScore = 0;
let lastRenderTime = 0;

// Snake variables
let snake = [];
let direction = 'right';
let nextDirection = 'right';

// Enemy snake variables
let enemySnake = [];
let enemyDirection = 'left';
let enemyUpdateCounter = 0;

// Food variables
let food = {};
let superFood = {};
let superFoodActive = false;
let superFoodTimer = 0;
let superFoodEffect = false;
let superFoodEffectTimer = 0;

// Initialize the game
function initGame() {
    snake = [
        { x: 5, y: 10 },
        { x: 4, y: 10 },
        { x: 3, y: 10 }
    ];
    
    enemySnake = [
        { x: 25, y: 5 },
        { x: 26, y: 5 },
        { x: 27, y: 5 }
    ];
    
    direction = 'right';
    nextDirection = 'right';
    enemyDirection = 'left';
    
    score = 0;
    gameSpeed = initialSpeed;
    gameRunning = true;
    gameOver = false;
    
    generateFood();
    superFoodActive = false;
    superFoodEffect = false;
    
    requestAnimationFrame(gameLoop);
}

// Game loop
function gameLoop(currentTime) {
    if (!gameRunning) return;
    
    requestAnimationFrame(gameLoop);
    
    const secondsSinceLastRender = (currentTime - lastRenderTime) / 1000;
    if (secondsSinceLastRender < gameSpeed / 1000) return;
    
    lastRenderTime = currentTime;
    
    update();
    draw();
}

// Update game state
function update() {
    if (gameOver) return;
    
    direction = nextDirection;
    
    const head = { x: snake[0].x, y: snake[0].y };
    
    switch (direction) {
        case 'up': head.y--; break;
        case 'down': head.y++; break;
        case 'left': head.x--; break;
        case 'right': head.x++; break;
    }
    
    if (head.x < 0 || head.x >= canvas.width / gridSize || 
        head.y < 0 || head.y >= canvas.height / gridSize) {
        gameOver = true;
        return;
    }
    
    for (let i = 0; i < snake.length; i++) {
        if (snake[i].x === head.x && snake[i].y === head.y) {
            gameOver = true;
            return;
        }
    }
    
    for (let i = 0; i < enemySnake.length; i++) {
        if (enemySnake[i].x === head.x && enemySnake[i].y === head.y) {
            gameOver = true;
            return;
        }
    }
    
    snake.unshift(head);
    
    let ate = false;
    if (head.x === food.x && head.y === food.y) {
        score += 10;
        generateFood();
        ate = true;
        
        if (!superFoodActive && Math.random() < 0.2) {
            generateSuperFood();
        }
    }
    
    if (superFoodActive && head.x === superFood.x && head.y === superFood.y) {
        score += 50;
        superFoodActive = false;
        superFoodEffect = true;
        superFoodEffectTimer = 100;
        gameSpeed = initialSpeed * 0.6;
        ate = true;
    }
    
    if (!ate) {
        snake.pop();
    }
    
    if (superFoodActive) {
        superFoodTimer--;
        if (superFoodTimer <= 0) {
            superFoodActive = false;
        }
    }
    
    if (superFoodEffect) {
        superFoodEffectTimer--;
        if (superFoodEffectTimer <= 0) {
            superFoodEffect = false;
            gameSpeed = initialSpeed;
        }
    }
    
    updateEnemySnake();
    
    if (score > highScore) {
        highScore = score;
    }
}

// Update enemy snake
function updateEnemySnake() {
    enemyUpdateCounter++;
    if (enemyUpdateCounter < 2) return;
    enemyUpdateCounter = 0;
    
    const enemyHead = { x: enemySnake[0].x, y: enemySnake[0].y };
    const playerHead = snake[0];
    
    let possibleDirections = ['up', 'down', 'left', 'right'];
    const oppositeDirections = {
        'up': 'down',
        'down': 'up',
        'left': 'right',
        'right': 'left'
    };
    possibleDirections = possibleDirections.filter(dir => dir !== oppositeDirections[enemyDirection]);
    
    const distances = {};
    possibleDirections.forEach(dir => {
        const newPos = { x: enemyHead.x, y: enemyHead.y };
        switch (dir) {
            case 'up': newPos.y--; break;
            case 'down': newPos.y++; break;
            case 'left': newPos.x--; break;
            case 'right': newPos.x++; break;
        }
        
        if (newPos.x < 0 || newPos.x >= canvas.width / gridSize || 
            newPos.y < 0 || newPos.y >= canvas.height / gridSize) {
            distances[dir] = Infinity;
            return;
        }
        
        for (let i = 0; i < enemySnake.length; i++) {
            if (enemySnake[i].x === newPos.x && enemySnake[i].y === newPos.y) {
                distances[dir] = Infinity;
                return;
            }
        }
        
        distances[dir] = Math.abs(newPos.x - playerHead.x) + Math.abs(newPos.y - playerHead.y);
    });
    
    let bestDir = enemyDirection;
    let shortestDistance = Infinity;
    
    if (Math.random() < 0.8) {
        for (const dir in distances) {
            if (distances[dir] < shortestDistance) {
                shortestDistance = distances[dir];
                bestDir = dir;
            }
        }
    } else {
        const validDirections = Object.keys(distances).filter(dir => distances[dir] !== Infinity);
        if (validDirections.length > 0) {
            bestDir = validDirections[Math.floor(Math.random() * validDirections.length)];
        }
    }
    
    enemyDirection = bestDir;
    
    const newHead = { x: enemyHead.x, y: enemyHead.y };
    switch (enemyDirection) {
        case 'up': newHead.y--; break;
        case 'down': newHead.y++; break;
        case 'left': newHead.x--; break;
        case 'right': newHead.x++; break;
    }
    
    if (newHead.x < 0 || newHead.x >= canvas.width / gridSize || 
        newHead.y < 0 || newHead.y >= canvas.height / gridSize) {
        enemyDirection = oppositeDirections[enemyDirection];
        return;
    }
    
    for (let i = 0; i < snake.length; i++) {
        if (snake[i].x === newHead.x && snake[i].y === newHead.y) {
            const validDirections = possibleDirections.filter(
                dir => dir !== bestDir && distances[dir] !== Infinity
            );
            if (validDirections.length > 0) {
                enemyDirection = validDirections[0];
                return;
            } else {
                return;
            }
        }
    }
    
    enemySnake.unshift(newHead);
    enemySnake.pop();
}

// Generate random food position
function generateFood() {
    food = {
        x: Math.floor(Math.random() * (canvas.width / gridSize)),
        y: Math.floor(Math.random() * (canvas.height / gridSize))
    };
    
    while (isPositionOccupied(food.x, food.y)) {
        food = {
            x: Math.floor(Math.random() * (canvas.width / gridSize)),
            y: Math.floor(Math.random() * (canvas.height / gridSize))
        };
    }
}

// Generate super food
function generateSuperFood() {
    superFood = {
        x: Math.floor(Math.random() * (canvas.width / gridSize)),
        y: Math.floor(Math.random() * (canvas.height / gridSize))
    };
    
    while (isPositionOccupied(superFood.x, superFood.y) || 
          (superFood.x === food.x && superFood.y === food.y)) {
        superFood = {
            x: Math.floor(Math.random() * (canvas.width / gridSize)),
            y: Math.floor(Math.random() * (canvas.height / gridSize))
        };
    }
    
    superFoodActive = true;
    superFoodTimer = 100;
}

// Check if position is occupied
function isPositionOccupied(x, y) {
    for (let i = 0; i < snake.length; i++) {
        if (snake[i].x === x && snake[i].y === y) return true;
    }
    
    for (let i = 0; i < enemySnake.length; i++) {
        if (enemySnake[i].x === x && enemySnake[i].y === y) return true;
    }
    
    return false;
}

// Draw game elements
function draw() {
    ctx.fillStyle = '#f0f0f0';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.strokeStyle = '#e0e0e0';
    ctx.lineWidth = 0.5;
    
    for (let x = 0; x < canvas.width; x += gridSize) {
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
    }
    
    for (let y = 0; y < canvas.height; y += gridSize) {
        ctx.beginPath();
        ctx.moveTo(0, y);
        ctx.lineTo(canvas.width, y);
        ctx.stroke();
    }
    
    ctx.fillStyle = superFoodEffect ? '#00a8ff' : '#4cd137';
    for (let i = 0; i < snake.length; i++) {
        const segment = snake[i];
        
        if (i === 0) {
            ctx.fillStyle = superFoodEffect ? '#0097e6' : '#44bd32';
            ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
            
            ctx.fillStyle = '#000';
            const eyeSize = gridSize / 5;
            const eyeOffset = gridSize / 3;
            
            let leftEyeX, leftEyeY, rightEyeX, rightEyeY;
            
            switch (direction) {
                case 'up':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + eyeOffset;
                    break;
                case 'down':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
                case 'left':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + eyeOffset;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
                case 'right':
                    leftEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
            }
            
            ctx.fillRect(leftEyeX, leftEyeY, eyeSize, eyeSize);
            ctx.fillRect(rightEyeX, rightEyeY, eyeSize, eyeSize);
            
            ctx.fillStyle = superFoodEffect ? '#00a8ff' : '#4cd137';
        } else {
            const intensity = Math.max(255 - (i * 5), 100);
            ctx.fillStyle = superFoodEffect 
                ? `rgba(0, ${intensity}, 255, 0.9)` 
                : `rgba(76, ${intensity}, 55, 0.9)`;
            
            const margin = 1;
            ctx.fillRect(
                segment.x * gridSize + margin,
                segment.y * gridSize + margin,
                gridSize - 2 * margin,
                gridSize - 2 * margin
            );
        }
    }
    
    for (let i = 0; i < enemySnake.length; i++) {
        const segment = enemySnake[i];
        
        if (i === 0) {
            ctx.fillStyle = '#e84118';
            ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize, gridSize);
            
            ctx.fillStyle = '#000';
            const eyeSize = gridSize / 5;
            const eyeOffset = gridSize / 3;
            
            let leftEyeX, leftEyeY, rightEyeX, rightEyeY;
            
            switch (enemyDirection) {
                case 'up':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + eyeOffset;
                    break;
                case 'down':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
                case 'left':
                    leftEyeX = segment.x * gridSize + eyeOffset;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + eyeOffset;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
                case 'right':
                    leftEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    leftEyeY = segment.y * gridSize + eyeOffset;
                    rightEyeX = segment.x * gridSize + gridSize - eyeOffset - eyeSize;
                    rightEyeY = segment.y * gridSize + gridSize - eyeOffset - eyeSize;
                    break;
            }
            
            ctx.fillRect(leftEyeX, leftEyeY, eyeSize, eyeSize);
            ctx.fillRect(rightEyeX, rightEyeY, eyeSize, eyeSize);
        } else {
            const intensity = Math.max(255 - (i * 5), 100);
            ctx.fillStyle = `rgba(${intensity}, 65, 24, 0.9)`;
            
            const margin = 1;
            ctx.fillRect(
                segment.x * gridSize + margin,
                segment.y * gridSize + margin,
                gridSize - 2 * margin,
                gridSize - 2 * margin
            );
        }
    }
    
    ctx.fillStyle = '#e74c3c';
    const foodMargin = 2;
    ctx.beginPath();
    ctx.arc(
        food.x * gridSize + gridSize / 2,
        food.y * gridSize + gridSize / 2,
        gridSize / 2 - foodMargin,
        0,
        Math.PI * 2
    );
    ctx.fill();
    
    ctx.fillStyle = '#27ae60';
    ctx.fillRect(
        food.x * gridSize + gridSize / 2 - 1,
        food.y * gridSize + foodMargin,
        2,
        gridSize / 4
    );
    
    if (superFoodActive) {
        const pulseSize = 1 + 0.2 * Math.sin(Date.now() / 100);
        
        ctx.shadowColor = 'gold';
        ctx.shadowBlur = 10;
        
        ctx.fillStyle = '#fbc531';
        ctx.beginPath();
        ctx.arc(
            superFood.x * gridSize + gridSize / 2,
            superFood.y * gridSize + gridSize / 2,
            (gridSize / 2 - foodMargin) * pulseSize,
            0,
            Math.PI * 2
        );
        ctx.fill();
        
        ctx.shadowBlur = 0;
        
        const sparkleCount = 5;
        const time = Date.now() / 200;
        
        for (let i = 0; i < sparkleCount; i++) {
            const angle = (i / sparkleCount) * Math.PI * 2 + time;
            const distance = gridSize * 0.6;
            const x = superFood.x * gridSize + gridSize / 2 + Math.cos(angle) * distance;
            const y = superFood.y * gridSize + gridSize / 2 + Math.sin(angle) * distance;
            
            ctx.fillStyle = `rgba(255, 215, 0, ${0.5 + 0.5 * Math.sin(time + i)})`;
            ctx.beginPath();
            ctx.arc(x, y, 2, 0, Math.PI * 2);
            ctx.fill();
        }
    }
    
    ctx.fillStyle = '#000';
    ctx.font = '16px Arial';
    ctx.fillText(`Score: ${score}`, 10, 20);
    ctx.fillText(`High Score: ${highScore}`, 10, 40);
    
    if (superFoodEffect) {
        ctx.fillStyle = '#00a8ff';
        ctx.fillText(`Super Speed! ${Math.ceil(superFoodEffectTimer / 20)}s`, 10, 60);
    }
    
    if (gameOver) {
        ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        
        ctx.fillStyle = '#fff';
        ctx.font = '30px Arial';
        ctx.textAlign = 'center';
        ctx.fillText('Game Over', canvas.width / 2, canvas.height / 2 - 30);
        
        ctx.font = '20px Arial';
        ctx.fillText(`Final Score: ${score}`, canvas.width / 2, canvas.height / 2 + 10);
        ctx.fillText('Press Space to Restart', canvas.width / 2, canvas.height / 2 + 40);
        
        ctx.textAlign = 'left';
    }
}

// Event listeners
document.addEventListener('keydown', function(e) {
    switch(e.key) {
        case 'ArrowUp':
        case 'w':
            if (direction !== 'down') nextDirection = 'up';
            e.preventDefault();
            break;
        case 'ArrowDown':
        case 's':
            if (direction !== 'up') nextDirection = 'down';
            e.preventDefault();
            break;
        case 'ArrowLeft':
        case 'a':
            if (direction !== 'right') nextDirection = 'left';
            e.preventDefault();
            break;
        case 'ArrowRight':
        case 'd':
            if (direction !== 'left') nextDirection = 'right';
            e.preventDefault();
            break;
        case ' ':
            if (gameOver) {
                initGame();
            } else if (!gameRunning) {
                gameRunning = true;
                requestAnimationFrame(gameLoop);
            }
            e.preventDefault();
            break;
        case 'p':
            gameRunning = !gameRunning;
            if (gameRunning) {
                requestAnimationFrame(gameLoop);
            }
            e.preventDefault();
            break;
    }
});

// Mobile controls
let touchStartX = 0;
let touchStartY = 0;

canvas.addEventListener('touchstart', function(e) {
    touchStartX = e.touches[0].clientX;
    touchStartY = e.touches[0].clientY;
    e.preventDefault();
});

canvas.addEventListener('touchmove', function(e) {
    if (!touchStartX || !touchStartY) return;
    
    const touchEndX = e.touches[0].clientX;
    const touchEndY = e.touches[0].clientY;
    
    const dx = touchEndX - touchStartX;
    const dy = touchEndY - touchStartY;
    
    if (Math.abs(dx) > Math.abs(dy)) {
        if (dx > 0 && direction !== 'left') nextDirection = 'right';
        else if (dx < 0 && direction !== 'right') nextDirection = 'left';
    } else {
        if (dy > 0 && direction !== 'up') nextDirection = 'down';
        else if (dy < 0 && direction !== 'down') nextDirection = 'up';
    }
    
    touchStartX = touchEndX;
    touchStartY = touchEndY;
    e.preventDefault();
});

canvas.addEventListener('touchend', function(e) {
    touchStartX = 0;
    touchStartY = 0;
    e.preventDefault();
    
    if (gameOver) {
        initGame();
    } else if (!gameRunning) {
        gameRunning = true;
        requestAnimationFrame(gameLoop);
    }
});

// Instructions
const instructions = document.createElement('div');
instructions.style.maxWidth = '600px';
instructions.style.margin = '10px auto';
instructions.style.padding = '10px';
instructions.style.border = '1px solid #ccc';
instructions.style.backgroundColor = '#f9f9f9';
instructions.innerHTML = `
    Advanced Snake Game
    Controls: Arrow keys or WASD to move, Space to start/restart, P to pause
    Features:
    - Regular Food: Red apples give 10 points
    - Super Food: Golden apples give 50 points and speed boost
    - Enemy Snake: AI-controlled snake that follows you
`;
document.body.insertBefore(instructions, canvas);

// Start button
const startButton = document.createElement('button');
startButton.textContent = 'Start Game';
startButton.style.display = 'block';
startButton.style.margin = '10px auto';
startButton.style.padding = '10px 20px';
startButton.style.fontSize = '16px';
startButton.style.backgroundColor = '#4cd137';
startButton.style.color = 'white';
startButton.style.border = 'none';
startButton.style.borderRadius = '5px';
startButton.style.cursor = 'pointer';
startButton.addEventListener('click', function() {
    initGame();
});
document.body.insertBefore(startButton, instructions);

// Draw initial game state
draw();
</script>
</body>
</html>
```
