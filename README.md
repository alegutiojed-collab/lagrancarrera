<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>La Gran Carrera - Reto Político</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #111;
            color: white;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
        }

        h1 {
            margin: 5px;
            color: #ff3333;
            text-shadow: 3px 3px #000;
            font-size: 28px;
            text-align: center;
        }

        #game-container {
            position: relative;
            width: 400px;
            height: 500px;
            background: linear-gradient(to bottom, #87CEEB 60%, #555 60%);
            border: 4px solid #fff;
            box-shadow: 0 0 20px rgba(255,255,255,0.2);
            overflow: hidden;
        }

        /* Elementos Pixel Art con CSS */
        #capitolio {
            position: absolute;
            top: 40px;
            left: 50%;
            transform: translateX(-50%);
            width: 120px;
            height: 60px;
            background: #ddd;
            border-top: 20px solid #bbb;
            opacity: 0.7;
        }

        #player {
            position: absolute;
            bottom: 20px;
            left: 185px;
            width: 30px;
            height: 45px;
            background-color: #0055ff; /* Traje azul */
            border-radius: 5px;
            transition: left 0.1s ease;
        }

        /* Cabeza del candidato */
        #player::before {
            content: '';
            position: absolute;
            top: -15px;
            left: 5px;
            width: 20px;
            height: 15px;
            background-color: #ffcc99;
            border-radius: 3px;
        }

        /* Corbata roja presidencial */
        #player::after {
            content: '';
            position: absolute;
            top: 5px;
            left: 13px;
            width: 4px;
            height: 15px;
            background-color: #ff0000;
        }

        .obstacle {
            position: absolute;
            width: 25px;
            height: 25px;
            background-color: #ff3333; /* Escándalo político */
            border: 2px solid #000;
            box-sizing: border-box;
        }

        .item {
            position: absolute;
            width: 20px;
            height: 20px;
            background-color: #ffd700; /* Votos / Monedas */
            border-radius: 50%;
            border: 2px solid #cc9900;
        }

        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            right: 10px;
            display: flex;
            justify-content: space-between;
            font-weight: bold;
            font-size: 16px;
            background: rgba(0,0,0,0.6);
            padding: 5px 10px;
            border-radius: 5px;
            z-index: 10;
        }

        #votos { color: #ffd700; }
        #comentarios { color: #ff5555; }

        #screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.85);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            z-index: 20;
            text-align: center;
            padding: 20px;
            box-sizing: border-box;
        }

        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 18px;
            font-family: inherit;
            cursor: pointer;
            border-bottom: 4px solid #2e7d32;
            margin-top: 15px;
        }

        button:active {
            border-bottom: 0;
            margin-top: 19px;
        }

        #controls-hint {
            margin-top: 10px;
            font-size: 12px;
            color: #aaa;
            text-align: center;
        }
    </style>
</head>
<body>

    <h1>LA GRAN CARRERA</h1>
    
    <div id="game-container">
        <div id="ui">
            <div id="votos">VOTOS: 0%</div>
            <div id="comentarios">ALERTA</div>
        </div>

        <div id="capitolio"></div>
        <div id="player"></div>

        <!-- Pantallas de Inicio / Game Over -->
        <div id="screen">
            <h2 id="screen-title">LA CARRERA POR EL PODER</h2>
            <p id="screen-desc">Esquiva los escándalos políticos (rojos) y recolecta votos (dorados) para llegar al 100% de aprobación popular.</p>
            <button id="start-btn">INICIAR CAMPAÑA</button>
        </div>
    </div>

    <div id="controls-hint">Usa las flechas ⬅️ ➡️ de tu teclado o toca los lados de la pantalla para moverte.</div>

    <script>
        const container = document.getElementById('game-container');
        const player = document.getElementById('player');
        const startBtn = document.getElementById('start-btn');
        const screen = document.getElementById('screen');
        const screenTitle = document.getElementById('screen-title');
        const screenDesc = document.getElementById('screen-desc');
        const votosTxt = document.getElementById('votos');
        const alertTxt = document.getElementById('comentarios');

        let playerX = 185;
        const containerWidth = 400;
        const playerWidth = 300; 
        let gameActive = false;
        let score = 0;
        let obstacles = [];
        let items = [];
        let gameInterval;
        let spawnCounter = 0;
        let speedMultiplier = 1;

        // Movimiento del jugador
        document.addEventListener('keydown', (e) => {
            if (!gameActive) return;
            if (e.key === 'ArrowLeft' && playerX > 10) {
                playerX -= 25;
            } else if (e.key === 'ArrowRight' && playerX < 360) {
                playerX += 25;
            }
            player.style.left = playerX + 'px';
        });

        // Controles táctiles para móvil
        container.addEventListener('touchstart', (e) => {
            if (!gameActive) return;
            const touchX = e.touches[0].clientX - container.getBoundingClientRect().left;
            if (touchX < containerWidth / 2 && playerX > 10) {
                playerX -= 25;
            } else if (touchX >= containerWidth / 2 && playerX < 360) {
                playerX += 25;
            }
            player.style.left = playerX + 'px';
        });

        startBtn.addEventListener('click', startGame);

        function startGame() {
            // Reiniciar variables
            gameActive = true;
            score = 0;
            playerX = 185;
            player.style.left = playerX + 'px';
            votosTxt.innerText = "VOTOS: 0%";
            alertTxt.innerText = "CAMPAÑA ACTIVA";
            alertTxt.style.color = "#55ff55";
            screen.style.display = 'none';
            
            // Limpiar elementos viejos
            obstacles.forEach(o => o.el.remove());
            items.forEach(i => i.el.remove());
            obstacles = [];
            items = [];
            spawnCounter = 0;
            speedMultiplier = 1;

            // Bucle principal del juego
            clearInterval(gameInterval);
            gameInterval = setInterval(updateGame, 20);
        }

        function updateGame() {
            spawnCounter++;
            
            // Aumentar dificultad gradualmente
            if (score > 0 && score % 20 === 0) {
                speedMultiplier = 1 + (score / 100);
            }

            // Generar obstáculos (Escándalos de la prensa)
            if (spawnCounter % Math.max(15, Math.floor(40 / speedMultiplier)) === 0) {
                createObstacle();
            }

            // Generar votos (Simpatizantes)
            if (spawnCounter % 50 === 0) {
                createItem();
            }

            // Mover y revisar obstáculos
            for (let i = obstacles.length - 1; i >= 0; i--) {
                let obs = obstacles[i];
                obs.y += 4 * speedMultiplier;
                obs.el.style.top = obs.y + 'px';

                // Colisión con jugador
                if (checkCollision(obs, 45)) {
                    gameOver("¡ESCÁNDALO CORRUPTO!", "Tu campaña colapsó por malos manejos. Quedaste fuera de la carrera electoral.");
                }

                // Salir de pantalla
                if (obs.y > 500) {
                    obs.el.remove();
                    obstacles.splice(i, 1);
                }
            }

            // Mover y revisar ítems (Votos)
            for (let i = items.length - 1; i >= 0; i--) {
                let it = items[i];
                it.y += 3 * speedMultiplier;
                it.el.style.top = it.y + 'px';

                // Recolectar votos
                if (checkCollision(it, 35)) {
                    score += 5;
                    votosTxt.innerText = `VOTOS: ${score}%`;
                    it.el.remove();
                    items.splice(i, 1);

                    // Condición de victoria
                    if (score >= 100) {
                        gameWin();
                    }
                }

                if (it.y > 500) {
                    it.el.remove();
                    items.splice(i, 1);
                }
            }
        }

        function createObstacle() {
            const x = Math.floor(Math.random() * 14) * 25 + 15;
            const el = document.createElement('div');
            el.className = 'obstacle';
            el.style.left = x + 'px';
            el.style.top = '0px';
            container.appendChild(el);
            obstacles.push({ el, x, y: 0 });
            
            const alertas = ["Fake News", "Soborno", "Fraude", "Chisme", "Debate"];
            if (Math.random() > 0.7) {
                alertTxt.innerText = alertas[Math.floor(Math.random() * alertas.length)].toUpperCase();
                alertTxt.style.color = "#ff3333";
            }
        }

        function createItem() {
            const x = Math.floor(Math.random() * 14) * 25 + 15;
            const el = document.createElement('div');
            el.className = 'item';
            el.style.left = x + 'px';
            el.style.top = '0px';
            container.appendChild(el);
            items.push({ el, x, y: 0 });
        }

        function checkCollision(obj, playerHeightOffset) {
            const playerTop = 500 - playerHeightOffset;
            return (
                obj.y + 20 >= playerTop &&
                obj.y <= 480 &&
                obj.x + 20 >= playerX &&
                obj.x <= playerX + 30
            );
        }

        function gameOver(title, desc) {
            gameActive = false;
            clearInterval(gameInterval);
            screenTitle.innerText = title;
            screenTitle.style.color = "#ff3333";
            screenDesc.innerText = desc;
            startBtn.innerText = "INTENTAR DE NUEVO";
            screen.style.display = 'flex';
        }

        function gameWin() {
            gameActive = false;
            clearInterval(gameInterval);
            screenTitle.innerText = "¡VICTORIA ELECTORAL!";
            screenTitle.style.color = "#4CAF50";
            screenDesc.innerText = "¡Felicidades! Has alcanzado el 100% de los votos y ganaste la presidencia en La Gran Carrera.";
            startBtn.innerText = "JUGAR OTRA VEZ";
            screen.style.display = 'flex';
        }
    </script>
</body>
</html>
