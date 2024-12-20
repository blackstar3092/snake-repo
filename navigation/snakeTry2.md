---
layout: base
title: Snake
permalink: /snake-risha/
---

<style>
  /* Overall page layout */

  /* Title */
  h1 {
    font-size: 32px;
    margin-bottom: 20px;
  }

  /* Container for game canvas */
  .game-container {
    display: flex;
    justify-content: center;
    width: 100%; /* Stretch across the page horizontally */
    max-width: 800px; /* Limit max width of the canvas */
    margin: 0 auto;
  }

  /* The canvas itself */
  #gameCanvas {
    border: 1px solid #000;
    display: block;
    filter: none;
  }

  /* Container for controls */
  .controls-container {
    text-align: center;
    margin-top: 20px;
  }

  /* Controls for settings */
  .controls {
    display: flex;
    justify-content: center;
    gap: 20px;
    margin-top: 10px;
  }

  /* Styling for the score display */
  .score {
    font-size: 18px;
    margin-bottom: 10px;
  }

  /* Button styling */
  button {
    font-size: 14px;
    cursor: pointer;
  }

  #startButton {
    margin-top: 20px;
    font-size: 18px;
    padding: 10px 20px;
    display: block;
    margin-left: auto;
    margin-right: auto;
  }

  /* Disabled button state */
  button:disabled {
    cursor: not-allowed;
  }
  
</style>

  <h1> Snake Game</h1>
  <div class="controls-container">
    <div class="score">Score: 0</div>
  </div>
  <div class="game-container">
    <canvas id="gameCanvas" width="400" height="400"></canvas>
  </div>
  <div class="controls-container">
    <div class="controls">
      <label>
        Walls:
        <select id="wallsToggle">
          <option value="true">On</option>
          <option value="false">Off</option>
        </select>
      </label>
      <label>
        Speed:
        <select id="speedToggle">
          <option value="slow">Slow</option>
          <option value="medium" selected>Medium</option>
          <option value="fast">Fast</option>
        </select>
      </label>
     <label>
      Mode:
      <select id="modeToggle">
        <option value="regular">Regular</option>
        <option value="challenge">Challenge</option>
      </select>
    </label>
      <label>
        Background:
        <select id="backgroundSelect">
          <option value="space">Snake in Space</option>
          <option value="underwater">Snake Underwater</option>
          <option value="classic" selected>Classic</option>
        </select>
      </label>
     <button id="restartButton" disabled>Restart</button>
    </div>
  </div>
  <button id="startButton">Start Game</button>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.querySelector('.score');
    const wallsToggle = document.getElementById('wallsToggle');
    const speedToggle = document.getElementById('speedToggle');
    const restartButton = document.getElementById('restartButton');
    const startButton = document.getElementById('startButton');
    const backgroundSelect = document.getElementById('backgroundSelect');
    const modeToggle = document.getElementById('modeToggle');

    const gridSize = 20;
    const tileCount = canvas.width / gridSize;

    let snake = [{ x: 10, y: 10 }];
    let food = { x: 15, y: 15 };
    let direction = { x: 1, y: 0 };
    let walls = true;
    let speed = 150;
    let score = 0;
    let gameLoop = null;
    let backgroundImage = new Image();
    let mode = 'regular'; // Regular mode by default


    const backgrounds = {
      space: '{{site.baseurl}}/images/snake-in-space.jpg',  // Replace with actual path
      underwater: '{{site.baseurl}}/images/snake-underwater.jpg',  // Replace with actual path
      classic: '{{site.baseurl}}/images/classic-bkgd.png',  // Classic background is just blue
    };
    
    backgroundImage.src = backgrounds.classic; // Default to 'classic' background

    function drawBoard() {
      ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear the canvas
      ctx.drawImage(backgroundImage, 0, 0, canvas.width, canvas.height); // Draw the background
    }

    function drawBoard() {
    ctx.filter = 'none';
    ctx.clearRect(0, 0, canvas.width, canvas.height); // Clear the canvas before drawing the new background

    // Draw the background image to fill the entire canvas
    ctx.drawImage(backgroundImage, 0, 0, canvas.width, canvas.height);
    }


    /// function drawBoard() {
      /// ctx.fillStyle = '#e7f5a4';
      /// ctx.fillRect(0, 0, canvas.width, canvas.height);
    /// }

function drawSnake() {
  ctx.fillStyle = '#cfffe3'; // Snake body color

  // Function to draw a square with rounded corners
  function drawRoundedSquare(x, y, radius) {
    ctx.beginPath();
    ctx.moveTo(x + radius, y);
    ctx.arcTo(x + gridSize, y, x + gridSize, y + gridSize, radius);
    ctx.arcTo(x + gridSize, y + gridSize, x, y + gridSize, radius);
    ctx.arcTo(x, y + gridSize, x, y, radius);
    ctx.arcTo(x, y, x + gridSize, y, radius);
    ctx.closePath();
    ctx.fill();
  }

  // Function to draw the face on the snake's head
  function drawFace(x, y, direction) {
    const eyeRadius = gridSize / 6; // Eye radius relative to grid size
    const eyeOffsetX = gridSize / 4; // Horizontal offset for the eyes
    const eyeOffsetY = gridSize / 4; // Vertical offset for the eyes
    const mouthWidth = gridSize / 2;
    const mouthHeight = gridSize / 4;

    // Save the current state of the canvas to restore later
    ctx.save();

    // Rotate the canvas according to the direction
    ctx.translate(x + gridSize / 2, y + gridSize / 2); // Move to the center of the head
    if (direction === 'up') {
      ctx.rotate(Math.PI); // Rotate 180 degrees (up)
    } else if (direction === 'down') {
      ctx.rotate(0); // No rotation needed (down)
    } else if (direction === 'left') {
      ctx.rotate(Math.PI / 2); // Rotate 90 degrees (left)
    } else if (direction === 'right') {
      ctx.rotate(-Math.PI / 2); // Rotate -90 degrees (right)
    }

    // Draw the eyes
    ctx.fillStyle = 'white';
    ctx.beginPath();
    ctx.arc(eyeOffsetX, eyeOffsetY, eyeRadius, 0, Math.PI * 2); // Left eye
    ctx.fill();
    ctx.closePath();

    ctx.beginPath();
    ctx.arc(-eyeOffsetX, eyeOffsetY, eyeRadius, 0, Math.PI * 2); // Right eye
    ctx.fill();
    ctx.closePath();

    // Draw the pupils (black)
    ctx.fillStyle = 'black';
    ctx.beginPath();
    ctx.arc(eyeOffsetX, eyeOffsetY, eyeRadius / 2, 0, Math.PI * 2); // Left pupil
    ctx.fill();
    ctx.closePath();

    ctx.beginPath();
    ctx.arc(-eyeOffsetX, eyeOffsetY, eyeRadius / 2, 0, Math.PI * 2); // Right pupil
    ctx.fill();
    ctx.closePath();

    // Draw the mouth (smiling)
    ctx.strokeStyle = 'black';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.arc(0, gridSize / 2 + mouthHeight / 2, mouthWidth, 0, Math.PI); // Smiling mouth
    ctx.stroke();
    ctx.closePath();

    // Restore the canvas state (remove the rotation)
    ctx.restore();
  }

  snake.forEach((segment, index) => {
    const x = segment.x * gridSize;
    const y = segment.y * gridSize;

    const nextSegment = snake[index + 1] || { x: segment.x, y: segment.y };
    const prevSegment = snake[index - 1] || { x: segment.x, y: segment.y };

    const directionToNext = { x: nextSegment.x - segment.x, y: nextSegment.y - segment.y };
    const directionToPrev = { x: segment.x - prevSegment.x, y: segment.y - prevSegment.y };

    // Determine the direction of the snake's movement
    let direction = 'right'; // Default direction is 'right'
    if (directionToNext.x === 1 && directionToNext.y === 0) direction = 'right'; // Moving right
    else if (directionToNext.x === -1 && directionToNext.y === 0) direction = 'left'; // Moving left
    else if (directionToNext.x === 0 && directionToNext.y === 1) direction = 'down'; // Moving down
    else if (directionToNext.x === 0 && directionToNext.y === -1) direction = 'up'; // Moving up

    // Check if this is the first segment (head)
    if (index === 0) {
      // Draw the head with rounded corners
      drawRoundedSquare(x, y, gridSize / 4);
      // Draw the face on the head with the direction it is facing
      drawFace(x, y, direction);
    }
    // Check if this is the last segment (tail) — No rounded corners for the tail
    else if (index === snake.length - 1) {
      ctx.fillRect(x, y, gridSize, gridSize); // Draw tail as a normal square
    }
    // Body segments in the middle (check for turns)
    else {
      const isTurn = (directionToNext.x !== directionToPrev.x) || (directionToNext.y !== directionToPrev.y);
      if (isTurn) {
        // Draw a rounded corner for the turning body segment
        const turnDirection = (directionToNext.x === 0 || directionToNext.y === 0) ? gridSize / 4 : gridSize / 2;
        drawRoundedSquare(x, y, turnDirection);
      } else {
        // Draw straight body segments
        ctx.fillRect(x, y, gridSize, gridSize);
      }
    }
  });
}

function drawFood() {

if (mode === 'challenge') {
    if (food.type === 'apple') {
        // Draw the apple's body (red)
        ctx.fillStyle = '#ff4d4d'; // Red color for the apple
        ctx.beginPath();
        ctx.arc(food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 2, gridSize / 2, 0, Math.PI * 2);
        ctx.fill();
        ctx.closePath();

        // Draw the apple's stem (brown)
        ctx.fillStyle = '#6a4e23'; // Brown color for the stem
        ctx.beginPath();
        ctx.moveTo(food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 4); // Starting at the top of the apple
        ctx.lineTo(food.x * gridSize + gridSize / 2, food.y * gridSize); // Going down to the center of the apple
        ctx.lineWidth = 4;
        ctx.stroke();
        ctx.closePath();

        // Optionally, add a leaf to the apple (green)
        ctx.fillStyle = '#2e8b57'; // Green color for the leaf
        ctx.beginPath();
        ctx.moveTo(food.x * gridSize + gridSize / 2, food.y * gridSize - gridSize / 4); // Position the leaf
        ctx.quadraticCurveTo(food.x * gridSize + gridSize / 1.5, food.y * gridSize - gridSize / 2, food.x * gridSize + gridSize / 2, food.y * gridSize - gridSize / 3); // Leaf shape
        ctx.fill();
        ctx.closePath();
    } else if (food.type === 'pizza') {
        // Draw pizza (yellow crust and topping)
        ctx.fillStyle = '#f1c232'; // Yellow color for pizza crust
        ctx.beginPath();
        ctx.arc(food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 2, gridSize / 2, 0, Math.PI * 2);
        ctx.fill();
        ctx.closePath();

        // Draw pizza toppings (red)
        ctx.fillStyle = '#ff6347'; // Tomato topping color
        ctx.beginPath();
        ctx.arc(food.x * gridSize + gridSize / 2 - gridSize / 4, food.y * gridSize + gridSize / 2 - gridSize / 4, gridSize / 8, 0, Math.PI * 2);
        ctx.fill();
        ctx.closePath();

        // Draw more toppings
        ctx.fillStyle = '#ff6347'; // Another tomato topping
        ctx.beginPath();
        ctx.arc(food.x * gridSize + gridSize / 2 + gridSize / 4, food.y * gridSize + gridSize / 2 + gridSize / 4, gridSize / 8, 0, Math.PI * 2);
        ctx.fill();
        ctx.closePath();
    } 
} else {
    ctx.fillStyle = '#ff4d4d'; // Red color for the apple
    ctx.beginPath();
    ctx.arc(food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 2, gridSize / 2, 0, Math.PI * 2);
    ctx.fill();
    ctx.closePath();

    // Draw the apple's stem (brown)
    ctx.fillStyle = '#6a4e23'; // Brown color for the stem
    ctx.beginPath();
    ctx.moveTo(food.x * gridSize + gridSize / 2, food.y * gridSize + gridSize / 4); // Starting at the top of the apple
    ctx.lineTo(food.x * gridSize + gridSize / 2, food.y * gridSize); // Going down to the center of the apple
    ctx.lineWidth = 4;
    ctx.stroke();
    ctx.closePath();

    // Optionally, add a leaf to the apple (green)
    ctx.fillStyle = '#2e8b57'; // Green color for the leaf
    ctx.beginPath();
    ctx.moveTo(food.x * gridSize + gridSize / 2, food.y * gridSize - gridSize / 4); // Position the leaf
    ctx.quadraticCurveTo(food.x * gridSize + gridSize / 1.5, food.y * gridSize - gridSize / 2, food.x * gridSize + gridSize / 2, food.y * gridSize - gridSize / 3); // Leaf shape
    ctx.fill();
    ctx.closePath();
    }
}
    function moveSnake() {
      if (!gameLoop) return; // Don't move before the game starts

      const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

      if (walls) {
        if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
          endGame();
          return;
        }
      } else {
        head.x = (head.x + tileCount) % tileCount;
        head.y = (head.y + tileCount) % tileCount;
      }

      if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
        endGame();
        return;
      }

      snake.unshift(head);
        if (head.x === food.x && head.y === food.y) {
            if (food.type === 'apple') {
                score += 1;
            } else if (food.type === 'pizza') {
                score += 3;
            }
            scoreDisplay.textContent = `Score: ${score}`;
            placeFood();
            } else {
            snake.pop();
            }
        }
            
function placeFood() {
   let newFoodPosition;
   do {
     newFoodPosition = {
       x: Math.floor(Math.random() * tileCount),
       y: Math.floor(Math.random() * tileCount)
     };
   } while (snake.some(segment => segment.x === newFoodPosition.x && segment.y === newFoodPosition.y));

   // Random food type selection with bias for poison
   if (mode === 'challenge') {
     const foodTypes = ['apple', 'apple', 'pizza']; 
     const randomFoodType = foodTypes[Math.floor(Math.random() * foodTypes.length)];
     food = { ...newFoodPosition, type: randomFoodType };
   } else {
     food = { ...newFoodPosition, type: 'apple' }; // Regular mode is only apples
   }
}



    function changeDirection(event) {
      const { key } = event;
      if (key === 'ArrowUp' && direction.y === 0) direction = { x: 0, y: -1 };
      if (key === 'ArrowDown' && direction.y === 0) direction = { x: 0, y: 1 };
      if (key === 'ArrowLeft' && direction.x === 0) direction = { x: -1, y: 0 };
      if (key === 'ArrowRight' && direction.x === 0) direction = { x: 1, y: 0 };

      // Prevent scrolling when arrow keys are pressed
      if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(key)) {
        event.preventDefault();
      }
    }

    function endGame() {
      clearInterval(gameLoop);
      gameLoop = null; // Set gameLoop to null to stop moving the snake
      alert(`Game over! Your final score is ${score}.`);
      restartButton.disabled = false;
      startButton.style.display = 'block';
    }

    function updateSettings() {
      walls = wallsToggle.value === 'true';
      speed = speedToggle.value === 'slow' ? 300 : speedToggle.value === 'medium' ? 150 : 75;
    }

    function updateBackground() {
    const selectedBackground = backgroundSelect.value;
    backgroundImage.src = backgrounds[selectedBackground]; // Set the image source to the selected background
    backgroundImage.onload = function() {
        drawBoard(); // Redraw the board with the new background once the image has loaded
    };
    }

    function restartGame() {
      snake = [{ x: 10, y: 10 }];
      direction = { x: 1, y: 0 };
      score = 0;
      scoreDisplay.textContent = `Score: 0`;
      placeFood();
      updateSettings();
      updateBackground();
      clearInterval(gameLoop);
      gameLoop = setInterval(() => {
        drawBoard();
        drawFood();
        moveSnake();
        drawSnake();
      }, speed);
    }

    function startGame() {
      startButton.style.display = 'none';
      restartButton.disabled = false;
      restartGame();
    }

    // Event Listeners
    wallsToggle.addEventListener('change', updateSettings);
    speedToggle.addEventListener('change', restartGame);
    restartButton.addEventListener('click', restartGame);
    startButton.addEventListener('click', startGame);
    backgroundSelect.addEventListener('change', updateBackground);
    modeToggle.addEventListener('change', (event) => {
       mode = event.target.value;
        placeFood();  // Recreate food based on the selected mode
    });


    window.addEventListener('keydown', changeDirection);

</script>
