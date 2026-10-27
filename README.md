<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Especial de Halloween con Sofi</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    body { background-color: #0f0a1c; color: #ffffff; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
    #game-container { width: 850px; height: 550px; background: #1e1333; border: 4px solid #ff7518; border-radius: 16px; box-shadow: 0 0 25px rgba(255, 117, 24, 0.4); position: relative; overflow: hidden; }
    .screen { display: none; width: 100%; height: 100%; padding: 20px; flex-direction: column; align-items: center; justify-content: center; }
    .screen.active { display: flex; }

    /* ==========================================================
       [ZONA DE ESTILOS] 1. MODIFICA AQUÍ EL DISEÑO DEL BOTÓN DE INICIO
       ========================================================== */
    .title-screen {
      text-align: center;
      background: radial-gradient(circle, #2a1b4e 0%, #120b24 100%);
    }
    .game-title {
      font-size: 3rem;
      color: #ff7518;
      text-shadow: 0 0 10px rgba(255, 117, 24, 0.6);
      margin-bottom: 10px;
    }
    .game-subtitle {
      font-size: 1.2rem;
      color: #d1c4e9;
      margin-bottom: 30px;
    }
    .btn-start-custom {
      background: linear-gradient(135deg, #ff7518, #ff5722); /* Color de fondo o degradado */
      color: #ffffff;                                      /* Color del texto */
      font-size: 1.4rem;                                   /* Tamaño de la letra */
      padding: 15px 40px;                                  /* Relleno / tamaño del botón */
      border: 2px solid #ffa726;                           /* Borde del botón */
      border-radius: 30px;                                 /* Curvatura de las esquinas */
      cursor: pointer;
      font-weight: bold;
      box-shadow: 0 4px 15px rgba(255, 117, 24, 0.4);    /* Sombreado */
      transition: transform 0.2s, background 0.2s;
    }
    .btn-start-custom:hover {
      transform: scale(1.08);                              /* Efecto al pasar el mouse por encima */
      background: linear-gradient(135deg, #ff8f00, #f4511e);
    }
    /* ========================================================== */

    .cinematic-box { background: rgba(0, 0, 0, 0.85); border: 2px solid #8a2be2; border-radius: 12px; padding: 30px; max-width: 650px; text-align: center; cursor: pointer; position: relative; }
    .cinematic-box h2 { color: #ff7518; margin-bottom: 15px; }
    .cinematic-box p { font-size: 1.2rem; line-height: 1.6; margin-bottom: 20px; }
    .click-hint { position: absolute; bottom: 10px; right: 15px; font-size: 0.85rem; color: #ff7518; animation: pulse 1.5s infinite; }
    @keyframes pulse { 0% { opacity: 0.4; } 50% { opacity: 1; } 100% { opacity: 0.4; } }
    .dressup-workspace { display: flex; gap: 30px; margin-top: 15px; }
    #avatar-preview { background: #2a1b4e; border-radius: 12px; border: 2px dashed #ff7518; }
    .controls-panel { display: flex; flex-direction: column; gap: 10px; width: 250px; }
    .controls-panel input, .controls-panel select { padding: 8px; border-radius: 6px; border: none; background: #392467; color: white; }
    button { padding: 10px 20px; font-size: 1rem; font-weight: bold; border: none; border-radius: 8px; cursor: pointer; transition: transform 0.1s, background 0.2s; }
    button:hover { transform: scale(1.05); }
    .btn-primary { background: #ff7518; color: white; }
    .btn-secondary { background: #8a2be2; color: white; }
    .hud { display: flex; justify-content: space-between; width: 100%; font-size: 1.3rem; font-weight: bold; color: #ff7518; margin-bottom: 10px; }
    #game-canvas { background: #120b24; border-radius: 10px; border: 2px solid #ff7518; }
    .end-container { display: flex; gap: 30px; margin-top: 15px; }
    .leaderboard-box { background: #2a1b4e; padding: 15px; border-radius: 10px; width: 250px; }
    .leaderboard-box ol { padding-left: 20px; margin-top: 10px; }
    #certificate-canvas { border-radius: 8px; margin-bottom: 10px; }
  </style>
</head>
<body>
  <div id="game-container">
    
    <!-- ==========================================================
       [ZONA HTML] 0. PANTALLA DE INICIO (NUEVA)
       ========================================================== -->
    <section id="screen-home" class="screen active title-screen">
      <h1 class="game-title">🎃 Noche de Dulces 🍬</h1>
      <p class="game-subtitle">Especial de Halloween con Sofi</p>
      <!-- Aquí puedes modificar el texto que dice el botón -->
      <button class="btn-start-custom" onclick="goToCinematic()">¡Empezar Aventura! 🚀</button>
    </section>

    <!-- PANTALLA 1: CINEMÁTICA INTERACTIVA -->
    <section id="screen-intro" class="screen">
      <div class="cinematic-box" onclick="handleCinematicClick()">
        <span class="click-hint">💬 Haz clic para continuar...</span>
        <h2 id="intro-speaker">Sofi</h2>
        <p id="intro-text">¡Hola! Bienvenidos al directo especial de Halloween 🎃.</p>
        <button id="btn-next-intro" class="btn-primary" style="display: none;" onclick="event.stopPropagation(); nextScreenFromIntro()">¡Ir al Vestidor! 🎃</button>
      </div>
    </section>

    <!-- PANTALLA 2: VESTIDOR -->
    <section id="screen-dressup" class="screen">
      <h2>¡Crea tu Disfraz!</h2>
      <div class="dressup-workspace">
        <canvas id="avatar-preview" width="300" height="350"></canvas>
        <div class="controls-panel">
          <label>Nombre del Jugador:</label>
          <input type="text" id="player-name-input" value="Viewer">

          <label>Tono de Piel:</label>
          <input type="color" id="skin-color" value="#ffdbac" onchange="updateAvatar()">

          <label>Disfraz Base:</label>
          <select id="select-outfit" onchange="updateAvatar()">
            <option value="bruja">Bruja</option>
            <option value="vampiro">Vampiro</option>
            <option value="fantasma">Fantasma</option>
            <option value="calabaza">Calabaza</option>
          </select>

          <label>Sombrero:</label>
          <select id="select-hat" onchange="updateAvatar()">
            <option value="ninguno">Ninguno</option>
            <option value="sombrero_bruja">Sombrero de Bruja</option>
            <option value="orejas_gato">Orejas de Gato</option>
          </select>

          <label>Expresión:</label>
          <select id="select-face" onchange="updateAvatar()">
            <option value="feliz">Feliz</option>
            <option value="travesura">Travesura</option>
            <option value="sorprendido">Sorprendido</option>
          </select>

          <button class="btn-primary" onclick="finishCustomization()">¡Listo! Continuar ▸</button>
        </div>
      </div>
    </section>

    <!-- PANTALLA 3: CINEMÁTICA INTERMEDIA -->
    <section id="screen-mid" class="screen">
      <div class="cinematic-box" onclick="handleMidClick()">
        <span class="click-hint">💬 Haz clic para continuar...</span>
        <h2 id="mid-speaker">Sofi</h2>
        <p id="mid-text">¡El disfraz quedó genial! Ahora sí estamos listos.</p>
        <button id="btn-next-mid" class="btn-primary" style="display: none;" onclick="event.stopPropagation(); startMinigameTransition()">¡Empezar a Recoger! 🎃</button>
      </div>
    </section>

    <!-- PANTALLA 4: MINIJUEGO -->
    <section id="screen-game" class="screen">
      <div class="hud">
        <div>⏳ Tiempo: <span id="hud-time">30</span>s</div>
        <div>🍬 Dulces: <span id="hud-score">0</span></div>
      </div>
      <canvas id="game-canvas" width="800" height="420"></canvas>
    </section>

    <!-- PANTALLA 5: RANKING Y CERTIFICADO -->
    <section id="screen-end" class="screen">
      <h2>🏆 ¡Fin del Recorrido de Halloween!</h2>
      <p style="margin-bottom: 10px;">Puntuación Final: <span id="final-score">0</span></p>

      <div class="end-container">
        <div class="leaderboard-box">
          <h3>Tabla de Posiciones</h3>
          <ol id="leaderboard-list"></ol>
        </div>

        <div class="certificate-box">
          <canvas id="certificate-canvas" width="450" height="280"></canvas>
          <div style="display: flex; gap: 10px; margin-top: 5px;">
            <button class="btn-primary" onclick="downloadCertificate()">📥 Descargar</button>
            <button class="btn-secondary" onclick="restartGame()">🔄 Repetir</button>
          </div>
        </div>
      </div>
    </section>

  </div>

  <script>
    const introSlides = [
      { speaker: "Sofi", text: "¡Hola! Bienvenidos al directo especial de Halloween 🎃." },
      { speaker: "Sofi", text: "Antes de salir a pedir dulces por la vecindad, ¡debemos arreglar a nuestro personaje!" },
      { speaker: "Sofi", text: "Haz clic en la pantalla para avanzar los diálogos y preparar tu disfraz." }
    ];

    const midSlides = [
      { speaker: "Sofi", text: "¡El disfraz quedó increíble! Ya casi salimos." },
      { speaker: "Sofi", text: "Instrucciones: Mueve tu personaje con las FLECHAS del teclado o arrastrando el MOUSE." },
      { speaker: "Sofi", text: "Atrapa dulces 🍬 y relojes ⏰ (para más tiempo). ¡Cuidado con las arañas 🕷️!" }
    ];

    let currentIntroIndex = 0;
    let currentMidIndex = 0;

    let playerCustomization = {
      name: "Viewer",
      skinColor: "#ffdbac",
      outfit: "bruja",
      hat: "sombrero_bruja",
      face: "feliz"
    };

    const gameConfig = {
      initialTime: 30,
      clockBonusTime: 8,
      playerSpeed: 8,
      itemSpawnInterval: 35,
      fallingSpeedMin: 2,
      fallingSpeedMax: 5
    };

    let gameTimer = 30;
    let gameScore = 0;
    let gameInterval = null;
    let animationFrameId = null;
    let isGameOver = false;

    const playerObj = { x: 360, y: 340, width: 70, height: 70 };
    let fallingItems = [];

    const ITEM_TYPES = [
      { type: "dulce1", icon: "🍬", points: 10, extraTime: 0, chance: 0.4 },
      { type: "dulce2", icon: "🍭", points: 15, extraTime: 0, chance: 0.3 },
      { type: "reloj",  icon: "⏰", points: 5,  extraTime: gameConfig.clockBonusTime, chance: 0.15 },
      { type: "arana",  icon: "🕷️", points: -15, extraTime: -3, chance: 0.15 }
    ];

    let leaderboard = JSON.parse(localStorage.getItem('halloween_leaderboard')) || [
      { name: "Sofi", score: 150 },
      { name: "ViewerPro", score: 120 },
      { name: "GhostHunter", score: 80 }
    ];

    const keys = {};
    window.addEventListener("keydown", e => keys[e.key] = true);
    window.addEventListener("keyup", e => keys[e.key] = false);

    // Función para saltar de la Pantalla de Inicio a la Cinemática 1
    function goToCinematic() {
      switchScreen("screen-intro");
    }

    function handleCinematicClick() {
      currentIntroIndex++;
      if (currentIntroIndex < introSlides.length) {
        document.getElementById("intro-speaker").innerText = introSlides[currentIntroIndex].speaker;
        document.getElementById("intro-text").innerText = introSlides[currentIntroIndex].text;
      } else {
        document.getElementById("intro-speaker").innerText = "¡Listo!";
        document.getElementById("intro-text").innerText = "Haz clic en el botón para comenzar a armar tu personaje.";
        document.querySelector("#screen-intro .click-hint").style.display = "none";
        document.getElementById("btn-next-intro").style.display = "inline-block";
      }
    }

    function nextScreenFromIntro() {
      switchScreen("screen-dressup");
      updateAvatar();
      currentIntroIndex = 0;
      document.querySelector("#screen-intro .click-hint").style.display = "block";
      document.getElementById("btn-next-intro").style.display = "none";
      document.getElementById("intro-speaker").innerText = introSlides[0].speaker;
      document.getElementById("intro-text").innerText = introSlides[0].text;
    }

    function handleMidClick() {
      currentMidIndex++;
      if (currentMidIndex < midSlides.length) {
        document.getElementById("mid-speaker").innerText = midSlides[currentMidIndex].speaker;
        document.getElementById("mid-text").innerText = midSlides[currentMidIndex].text;
      } else {
        document.getElementById("mid-speaker").innerText = "¡A Jugar!";
        document.getElementById("mid-text").innerText = "Haz clic en el botón para iniciar el minijuego.";
        document.querySelector("#screen-mid .click-hint").style.display = "none";
        document.getElementById("btn-next-mid").style.display = "inline-block";
      }
    }

    function startMinigameTransition() {
      switchScreen("screen-game");
      startGame();
      currentMidIndex = 0;
      document.querySelector("#screen-mid .click-hint").style.display = "block";
      document.getElementById("btn-next-mid").style.display = "none";
      document.getElementById("mid-speaker").innerText = midSlides[0].speaker;
      document.getElementById("mid-text").innerText = midSlides[0].text;
    }

    function switchScreen(screenId) {
      document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
      document.getElementById(screenId).classList.add("active");
    }

    function updateAvatar() {
      const canvas = document.getElementById("avatar-preview");
      const ctx = canvas.getContext("2d");
      
      playerCustomization.name = document.getElementById("player-name-input").value || "Viewer";
      playerCustomization.skinColor = document.getElementById("skin-color").value;
      playerCustomization.outfit = document.getElementById("select-outfit").value;
      playerCustomization.hat = document.getElementById("select-hat").value;
      playerCustomization.face = document.getElementById("select-face").value;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawCharacter(ctx, canvas.width / 2, canvas.height / 2 + 10, 1.3, playerCustomization);
    }

    function drawCharacter(ctx, x, y, scale, config) {
      ctx.save();
      ctx.translate(x, y);
      ctx.scale(scale, scale);

      ctx.fillStyle = config.skinColor;
      ctx.beginPath();
      ctx.arc(0, -40, 25, 0, Math.PI * 2);
      ctx.fill();

      if (config.outfit === "bruja") ctx.fillStyle = "#4a154b";
      else if (config.outfit === "vampiro") ctx.fillStyle = "#800020";
      else if (config.outfit === "fantasma") ctx.fillStyle = "#e0e0e0";
      else if (config.outfit === "calabaza") ctx.fillStyle = "#ff7518";

      ctx.fillRect(-20, -15, 40, 50);

      ctx.fillStyle = "#000";
      if (config.face === "feliz") {
        ctx.fillRect(-10, -45, 5, 5); ctx.fillRect(5, -45, 5, 5);
        ctx.beginPath(); ctx.arc(0, -35, 8, 0, Math.PI); ctx.stroke();
      } else if (config.face === "travesura") {
        ctx.fillRect(-10, -45, 6, 3); ctx.fillRect(5, -45, 6, 3);
        ctx.fillRect(-5, -35, 10, 3);
      } else {
        ctx.beginPath(); ctx.arc(-8, -45, 3, 0, Math.PI*2); ctx.arc(8, -45, 3, 0, Math.PI*2); ctx.fill();
        ctx.beginPath(); ctx.arc(0, -35, 5, 0, Math.PI*2); ctx.stroke();
      }

      if (config.hat === "sombrero_bruja") {
        ctx.fillStyle = "#111";
        ctx.beginPath(); ctx.moveTo(-35, -60); ctx.lineTo(35, -60); ctx.lineTo(0, -100); ctx.fill();
      } else if (config.hat === "orejas_gato") {
        ctx.fillStyle = "#111";
        ctx.beginPath(); ctx.moveTo(-20, -60); ctx.lineTo(-10, -80); ctx.lineTo(-5, -60); ctx.fill();
        ctx.beginPath(); ctx.moveTo(20, -60); ctx.lineTo(10, -80); ctx.lineTo(5, -60); ctx.fill();
      }

      ctx.restore();
    }

    function finishCustomization() {
      switchScreen("screen-mid");
    }

    function startGame() {
      gameTimer = gameConfig.initialTime;
      gameScore = 0;
      fallingItems = [];
      isGameOver = false;
      document.getElementById("hud-time").innerText = gameTimer;
      document.getElementById("hud-score").innerText = gameScore;

      const gameCanvas = document.getElementById("game-canvas");
      gameCanvas.onmousemove = (e) => {
        const rect = gameCanvas.getBoundingClientRect();
        playerObj.x = e.clientX - rect.left - playerObj.width / 2;
      };

      clearInterval(gameInterval);
      gameInterval = setInterval(() => {
        gameTimer--;
        document.getElementById("hud-time").innerText = gameTimer;
        if (gameTimer <= 0) endGame();
      }, 1000);

      let frameCount = 0;
      function gameLoop() {
        if (isGameOver) return;
        const ctx = gameCanvas.getContext("2d");
        ctx.clearRect(0, 0, gameCanvas.width, gameCanvas.height);

        if (keys["ArrowLeft"] && playerObj.x > 0) playerObj.x -= gameConfig.playerSpeed;
        if (keys["ArrowRight"] && playerObj.x < gameCanvas.width - playerObj.width) playerObj.x += gameConfig.playerSpeed;

        frameCount++;
        if (frameCount % gameConfig.itemSpawnInterval === 0) {
          const rand = Math.random();
          let cumulative = 0;
          let selected = ITEM_TYPES[0];
          for (let item of ITEM_TYPES) {
            cumulative += item.chance;
            if (rand <= cumulative) { selected = item; break; }
          }
          fallingItems.push({
            x: Math.random() * (gameCanvas.width - 40) + 20,
            y: -20,
            speed: Math.random() * (gameConfig.fallingSpeedMax - gameConfig.fallingSpeedMin) + gameConfig.fallingSpeedMin,
            icon: selected.icon,
            points: selected.points,
            extraTime: selected.extraTime
          });
        }

        for (let i = fallingItems.length - 1; i >= 0; i--) {
          let item = fallingItems[i];
          item.y += item.speed;

          ctx.font = "26px serif";
          ctx.fillText(item.icon, item.x, item.y);

          if (
            item.y >= playerObj.y - 20 &&
            item.y <= playerObj.y + playerObj.height &&
            item.x >= playerObj.x - 15 &&
            item.x <= playerObj.x + playerObj.width + 15
          ) {
            gameScore += item.points;
            if (gameScore < 0) gameScore = 0;
            if (item.extraTime !== 0) {
              gameTimer += item.extraTime;
              if (gameTimer < 0) gameTimer = 0;
              document.getElementById("hud-time").innerText = gameTimer;
            }
            document.getElementById("hud-score").innerText = gameScore;
            fallingItems.splice(i, 1);
            continue;
          }

          if (item.y > gameCanvas.height + 20) {
            fallingItems.splice(i, 1);
          }
        }

        drawCharacter(ctx, playerObj.x + playerObj.width / 2, playerObj.y + 30, 0.7, playerCustomization);
        animationFrameId = requestAnimationFrame(gameLoop);
      }
      gameLoop();
    }

    function endGame() {
      isGameOver = true;
      clearInterval(gameInterval);
      cancelAnimationFrame(animationFrameId);

      document.getElementById("final-score").innerText = gameScore;
      leaderboard.push({ name: playerCustomization.name, score: gameScore });
      leaderboard.sort((a, b) => b.score - a.score);
      leaderboard = leaderboard.slice(0, 5);
      localStorage.setItem('halloween_leaderboard', JSON.stringify(leaderboard));

      const list = document.getElementById("leaderboard-list");
      list.innerHTML = "";
      leaderboard.forEach(entry => {
        const li = document.createElement("li");
        li.innerText = `${entry.name}: ${entry.score} pts`;
        list.appendChild(li);
      });

      const certCanvas = document.getElementById("certificate-canvas");
      const ctx = certCanvas.getContext("2d");
      ctx.fillStyle = "#1e1333";
      ctx.fillRect(0, 0, certCanvas.width, certCanvas.height);
      ctx.strokeStyle = "#ff7518";
      ctx.lineWidth = 4;
      ctx.strokeRect(8, 8, certCanvas.width - 16, certCanvas.height - 16);

      ctx.fillStyle = "#ff7518";
      ctx.font = "bold 16px sans-serif";
      ctx.textAlign = "center";
      ctx.fillText("CERTIFICADO OFICIAL DE HALLOWEEN", certCanvas.width / 2, 40);

      ctx.fillStyle = "#ffffff";
      ctx.font = "14px sans-serif";
      ctx.fillText("Otorgado a:", certCanvas.width / 2, 70);

      ctx.fillStyle = "#8a2be2";
      ctx.font = "bold 20px sans-serif";
      ctx.fillText(playerCustomization.name, certCanvas.width / 2, 98);

      ctx.fillStyle = "#ffffff";
      ctx.font = "13px sans-serif";
      ctx.fillText(`Por completar la búsqueda con ${gameScore} puntos.`, certCanvas.width / 2, 125);

      drawCharacter(ctx, certCanvas.width / 2, 195, 0.9, playerCustomization);
      switchScreen("screen-end");
    }

    function downloadCertificate() {
      const certCanvas = document.getElementById("certificate-canvas");
      const link = document.createElement("a");
      link.download = `Certificado_${playerCustomization.name}.png`;
      link.href = certCanvas.toDataURL("image/png");
      link.click();
    }

    function restartGame() {
      switchScreen("screen-home");
    }
  </script>
</body>
</html>
