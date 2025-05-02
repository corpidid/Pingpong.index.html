# Pingpong.index.html
<!DOCTYPE html>
<html>
<head>
    <title>Ping Pong Mobile + Teclado</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #000;
            font-family: Arial, sans-serif;
            touch-action: manipulation;
            overflow: hidden;
        }
        #game-container {
            position: relative;
            width: 100%;
            max-width: 800px;
            height: 100vh;
            max-height: 600px;
            background-color: #000;
            border: 2px solid #fff;
            overflow: hidden;
        }
        #paddle-left, #paddle-right {
            position: absolute;
            width: 15px;
            height: 100px;
            background-color: #fff;
        }
        #paddle-left {
            left: 20px;
        }
        #paddle-right {
            right: 20px;
        }
        #ball {
            position: absolute;
            width: 15px;
            height: 15px;
            background-color: #fff;
            border-radius: 50%;
        }
        #center-line {
            position: absolute;
            width: 2px;
            height: 100%;
            background-color: #fff;
            left: 50%;
            opacity: 0.5;
        }
        #score {
            position: absolute;
            width: 100%;
            text-align: center;
            color: #fff;
            font-size: 36px;
            top: 20px;
        }
        #instructions {
            position: absolute;
            width: 100%;
            text-align: center;
            color: #fff;
            font-size: 16px;
            bottom: 120px;
            opacity: 0.7;
        }
        
        /* Controles mobile */
        #controls {
            position: absolute;
            width: 100%;
            bottom: 10px;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            box-sizing: border-box;
        }
        .player-controls {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .control-btn {
            width: 80px;
            height: 60px;
            background-color: rgba(255, 255, 255, 0.3);
            border: 2px solid #fff;
            border-radius: 10px;
            color: white;
            font-size: 24px;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none;
            -webkit-user-select: none;
        }
        .control-btn:active {
            background-color: rgba(255, 255, 255, 0.6);
        }
        
        /* Botão de pause */
        #pause-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            width: 60px;
            height: 60px;
            background-color: rgba(255, 255, 255, 0.3);
            border: 2px solid #fff;
            border-radius: 50%;
            color: white;
            font-size: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none;
        }
        
        /* Tela de início */
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }
        #start-btn {
            margin-top: 20px;
            padding: 15px 30px;
            background-color: #fff;
            color: #000;
            border: none;
            border-radius: 10px;
            font-size: 20px;
            cursor: pointer;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="paddle-left"></div>
        <div id="paddle-right"></div>
        <div id="ball"></div>
        <div id="center-line"></div>
        <div id="score">0 - 0</div>
        <div id="instructions">Teclado: Jogador 1 (W/S) | Jogador 2 (↑/↓)</div>
        
        <div id="controls">
            <div class="player-controls">
                <div class="control-btn" id="left-up">↑</div>
                <div class="control-btn" id="left-down">↓</div>
            </div>
            <div class="player-controls">
                <div class="control-btn" id="right-up">↑</div>
                <div class="control-btn" id="right-down">↓</div>
            </div>
        </div>
        
        <div id="pause-btn">II</div>
        
        <div id="start-screen">
            <h1 style="color: white;">Ping Pong</h1>
            <p style="color: white; text-align: center; max-width: 80%;">
                Use os botões na parte inferior para controlar as raquetes<br>
                ou as teclas W/S (Jogador 1) e ↑/↓ (Jogador 2)
            </p>
            <button id="start-btn">Começar Jogo</button>
        </div>
    </div>

    <script>
        // Elementos do jogo
        const gameContainer = document.getElementById('game-container');
        const paddleLeft = document.getElementById('paddle-left');
        const paddleRight = document.getElementById('paddle-right');
        const ball = document.getElementById('ball');
        const scoreDisplay = document.getElementById('score');
        const startScreen = document.getElementById('start-screen');
        const startBtn = document.getElementById('start-btn');
        const pauseBtn = document.getElementById('pause-btn');

        // Controles mobile
        const leftUpBtn = document.getElementById('left-up');
        const leftDownBtn = document.getElementById('left-down');
        const rightUpBtn = document.getElementById('right-up');
        const rightDownBtn = document.getElementById('right-down');

        // Configurações do jogo
        const gameWidth = gameContainer.clientWidth;
        const gameHeight = gameContainer.clientHeight;
        const paddleWidth = 15;
        const paddleHeight = 100;
        const ballSize = 15;
        const paddleSpeed = 8;
        let ballSpeedX = 5;
        let ballSpeedY = 5;

        // Posições iniciais
        let paddleLeftY = gameHeight / 2 - paddleHeight / 2;
        let paddleRightY = gameHeight / 2 - paddleHeight / 2;
        let ballX = gameWidth / 2 - ballSize / 2;
        let ballY = gameHeight / 2 - ballSize / 2;

        // Pontuação
        let scoreLeft = 0;
        let scoreRight = 0;

        // Estados do jogo
        let gameRunning = false;
        let gamePaused = false;
        let animationId;

        // Controles
        const keys = {
            w: false,
            s: false,
            ArrowUp: false,
            ArrowDown: false
        };

        // Event listeners para teclado
        document.addEventListener('keydown', (e) => {
            if (e.key in keys) {
                keys[e.key] = true;
            }
        });

        document.addEventListener('keyup', (e) => {
            if (e.key in keys) {
                keys[e.key] = false;
            }
        });

        // Controles touch
        function setupTouchControls() {
            // Jogador esquerdo
            leftUpBtn.addEventListener('touchstart', () => keys.w = true);
            leftUpBtn.addEventListener('touchend', () => keys.w = false);
            leftDownBtn.addEventListener('touchstart', () => keys.s = true);
            leftDownBtn.addEventListener('touchend', () => keys.s = false);
            
            // Jogador direito
            rightUpBtn.addEventListener('touchstart', () => keys.ArrowUp = true);
            rightUpBtn.addEventListener('touchend', () => keys.ArrowUp = false);
            rightDownBtn.addEventListener('touchstart', () => keys.ArrowDown = true);
            rightDownBtn.addEventListener('touchend', () => keys.ArrowDown = false);
            
            // Também funciona para mouse
            leftUpBtn.addEventListener('mousedown', () => keys.w = true);
            leftUpBtn.addEventListener('mouseup', () => keys.w = false);
            leftDownBtn.addEventListener('mousedown', () => keys.s = true);
            leftDownBtn.addEventListener('mouseup', () => keys.s = false);
            rightUpBtn.addEventListener('mousedown', () => keys.ArrowUp = true);
            rightUpBtn.addEventListener('mouseup', () => keys.ArrowUp = false);
            rightDownBtn.addEventListener('mousedown', () => keys.ArrowDown = true);
            rightDownBtn.addEventListener('mouseup
