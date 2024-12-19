---
layout: base
title: Snake
description: Snake Game by Aranya, Elliot, and Risha
---

<style>

    body{
    }
    .wrap{
        margin-left: auto;
        margin-right: auto;
    }

    canvas{
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: #FFFFFF;
    }
    canvas:focus{
        outline: none;
    }

    /* All screens style */
    #gameover p, #setting p, #menu p{
        font-size: 20px;
    }

    #gameover a, #setting a, #menu a{
        font-size: 30px;
        display: block;
    }

    #gameover a:hover, #setting a:hover, #menu a:hover{
        cursor: pointer;
    }

    #gameover a:hover::before, #setting a:hover::before, #menu a:hover::before{
        content: ">";
        margin-right: 10px;
    }

    #menu{
        display: block;
    }

    #gameover{
        display: none;
    }

    #setting{
        display: none;
    }

    #setting input{
        display:none;
    }

    #setting label{
        cursor: pointer;
    }

    #setting input:checked + label{
        background-color: #FFF;
        color: #000;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="120" checked/>
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75"/>
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35"/>
                <label for="speed3">Fast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked/>
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0"/>
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
(function(){
    /* Attributes of Game */
    /////////////////////////////////////////////////////////////
    // Canvas & Context
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");
    // HTML Game IDs
    const SCREEN_SNAKE = 0;
    const screen_snake = document.getElementById("snake");
    const ele_score = document.getElementById("score_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");
    // HTML Screen IDs (div)
    const SCREEN_MENU = -1, SCREEN_GAME_OVER=1, SCREEN_SETTING=2;
    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");
    // HTML Event IDs (a tags)
    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");

    const BLOCK = 10;   // size of block rendering
    let SCREEN = SCREEN_MENU;
    let snake, snake_dir, snake_next_dir, snake_speed;
    let maze = [];
    let apples = [];
    let score = 0;
    let wall;

    const levels = [{
        maze: [
            "############################",
            "#..........................#",
            "#.#####.####.######.#####.#",
            "#.#.....................#.#",
            "#.#.###.###########.###.#.#",
            "#.#...#.............#...#.#",
            "#.###.#####.###.#####.###.#",
            "#.....#.....#.#.....#.....#",
            "#####.#.#####.#####.#.#####",
            "#..............A...........#",
            "#####.#.#####.#####.#.#####",
            "#.....#.....#.#.....#.....#",
            "#.###.#####.###.#####.###.#",
            "#.#...#.............#...#.#",
            "#.#.###.###########.###.#.#",
            "#.#.....................#.#",
            "#.#####.####.######.#####.#",
            "#..........................#",
            "############################"
        ],
        apples: [
            {x: 14, y: 9, type: 'open', paths: [{x: 5, y: 5}, {x: 20, y: 5}]},
            {x: 20, y: 15, type: 'reset'}
        ]
    }];

    const drawMaze = function(level) {
        maze = level.maze;
        apples = level.apples;
        ctx.fillStyle = "#000";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        for (let y = 0; y < maze.length; y++) {
            for (let x = 0; x < maze[y].length; x++) {
                if (maze[y][x] === "#") {
                    ctx.fillStyle = "#FFF";
                    ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
                }
            }
        }

        apples.forEach(apple => {
            ctx.fillStyle = apple.type === 'open' ? "#0F0" : "#F00";
            ctx.fillRect(apple.x * BLOCK, apple.y * BLOCK, BLOCK, BLOCK);
        });
    };

    const newGame = function(){
        SCREEN = SCREEN_SNAKE;
        canvas.focus();
        snake = [{x: 1, y: 1}];
        snake_dir = 1;
        snake_next_dir = 1;
        snake_speed = 150;
        score = 0;
        ele_score.innerHTML = score;
        drawMaze(levels[0]);
        mainLoop();
    };

    const mainLoop = function(){
        let head = {x: snake[0].x, y: snake[0].y};

        snake_dir = snake_next_dir;
        switch(snake_dir){
            case 0: head.y--; break;
            case 1: head.x++; break;
            case 2: head.y++; break;
            case 3: head.x--; break;
        }

        if(checkCollision(head)){
            SCREEN = SCREEN_GAME_OVER;
            return;
        }

        let appleIndex = apples.findIndex(a => checkBlock(head.x, head.y, a.x, a.y));
        if (appleIndex !== -1) {
            let apple = apples[appleIndex];
            if (apple.type === 'open') {
                apple.paths.forEach(path => {
                    maze[path.y] = maze[path.y].substring(0, path.x) + '.' + maze[path.y].substring(path.x + 1);
                });
                apples.splice(appleIndex, 1);
            } else if (apple.type === 'reset') {
                snake_speed -= 10;
                newGame();
                return;
            }
        } else {
            snake.pop();
        }

        snake.unshift(head);
        drawMaze(levels[0]);
        snake.forEach(part => activeDot(part.x, part.y));
        setTimeout(mainLoop, snake_speed);
    };

    const activeDot = function(x, y){
        ctx.fillStyle = "#FFF";
        ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
    };

    const checkCollision = function(head){
        if (head.x < 0 || head.y < 0 || head.x >= canvas.width / BLOCK || head.y >= canvas.height / BLOCK) {
            return true;
        }
        if (maze[head.y][head.x] === "#") {
            return true;
        }
        return snake.some(part => checkBlock(part.x, part.y, head.x, head.y));
    };

    const checkBlock = function(x, y, _x, _y){
        return (x === _x && y === _y);
    };

    const setupButtons = function() {
        button_new_game.onclick = newGame;
        button_new_game1.onclick = newGame;
        button_new_game2.onclick = newGame;
    };

    window.onload = function(){
        setupButtons();
        newGame();
    };
})();

</script>