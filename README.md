<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Árbol de Flores Amarillas</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      background-color: #f4f3ed;
      font-family: 'Segoe UI', Georgia, serif;
      overflow: hidden;
      height: 100vh;
      width: 100vw;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    #container {
      position: relative;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
    }

    /* Botón inicial de flor */
    #start-btn {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      display: flex;
      flex-direction: column;
      align-items: center;
      cursor: pointer;
      z-index: 10;
      transition: opacity 0.5s ease;
    }

    #start-btn .flower-icon {
      width: 80px;
      height: 80px;
      border-radius: 50%;
      background: radial-gradient(circle, #3d2314 35%, #fbc02d 36%);
      box-shadow: 0 4px 15px rgba(0,0,0,0.15);
      position: relative;
      transition: transform 0.3s ease;
    }

    #start-btn:hover .flower-icon {
      transform: scale(1.1);
    }

    #start-btn span {
      margin-top: 10px;
      font-size: 14px;
      color: #333;
      font-weight: 500;
      background: #ffffff88;
      padding: 4px 12px;
      border-radius: 12px;
      border: 1px solid #ccc;
    }

    /* Canvas para la animación gráfica */
    canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }

    /* Contenedor del texto poético a la izquierda */
    #text-container {
      position: absolute;
      left: 8%;
      top: 15%;
      width: 400px;
      z-index: 5;
      color: #2c2c2c;
    }

    #text-container h2 {
      font-size: 1.4rem;
      letter-spacing: 1px;
      margin-bottom: 25px;
      font-weight: 700;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    #text-container p {
      font-size: 1.05rem;
      line-height: 1.8;
      font-weight: 600;
      letter-spacing: 1px;
      text-transform: uppercase;
      white-space: pre-line;
      min-height: 180px;
    }

    #text-container .footer-note {
      margin-top: 30px;
      font-size: 1rem;
      font-style: italic;
      text-transform: none;
      font-weight: 400;
      border-top: 1px solid #333;
      padding-top: 10px;
      display: inline-block;
    }

    .cursor {
      display: inline-block;
      width: 2px;
      height: 1em;
      background-color: #333;
      vertical-align: middle;
      animation: blink 0.7s infinite;
    }

    @keyframes blink {
      0%, 100% { opacity: 1; }
      50% { opacity: 0; }
    }
  </style>
</head>
<body>

  <div id="container">
    <!-- Botón inicial -->
    <div id="start-btn" onclick="startAnimation()">
      <div class="flower-icon"></div>
      <span>Click Aquí</span>
    </div>

    <!-- Canvas para renderizado de árbol y girasoles -->
    <canvas id="treeCanvas"></canvas>

    <!-- Texto romántico -->
    <div id="text-container">
      <h2 id="title"></h2>
      <p id="typed-text"></p>
      <div id="footer-text" class="footer-note"></div>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('treeCanvas');
    const ctx = canvas.getContext('2d');
    
    let width, height;
    function resize() {
      width = canvas.width = window.innerWidth;
      height = canvas.height = window.innerHeight;
    }
    window.addEventListener('resize', resize);
    resize();

    // Desplazamiento X del árbol (comienza al centro y se mueve a la derecha)
    let treeOffsetX = 0; 
    let targetOffsetX = 0;

    // Configuración de la animación
    let trunkProgress = 0;
    let branchProgress = 0;
    let flowerProgress = 0;
    let animState = 'idle'; // idle, trunk, branches, flowers, shift, typing

    // Dibujar el punto inicial rojo y la línea del suelo
    function drawBase() {
      ctx.clearRect(0, 0, width, height);

      const centerX = width / 2 + treeOffsetX;
      const groundY = height * 0.75;

      // Línea de tierra
      ctx.beginPath();
      ctx.moveTo(0, groundY);
      ctx.lineTo(width, groundY);
      ctx.strokeStyle = '#222';
      ctx.lineWidth = 1.5;
      ctx.stroke();

      // Punto rojo de origen
      if (animState !== 'idle') {
        ctx.beginPath();
        ctx.arc(centerX, groundY, 4, 0, Math.PI * 2);
        ctx.fillStyle = '#d32f2f';
        ctx.fill();
      }
    }

    // Dibujar un girasol individual
    function drawSunflower(x, y, radius) {
      ctx.save();
      ctx.translate(x, y);

      // Pétalos
      const petalCount = 12;
      ctx.fillStyle = '#fbc02d';
      for (let i = 0; i < petalCount; i++) {
        ctx.beginPath();
        ctx.rotate((Math.PI * 2) / petalCount);
        ctx.ellipse(0, radius * 0.6, radius * 0.3, radius * 0.6, 0, 0, Math.PI * 2);
        ctx.fill();
      }

      // Centro oscuro del girasol
      ctx.beginPath();
      ctx.arc(0, 0, radius * 0.45, 0, Math.PI * 2);
      ctx.fillStyle = '#3d2314';
      ctx.fill();

      ctx.restore();
    }

    // Puntos en forma de corazón para la copa del árbol
    const heartFlowers = [];
    function generateHeartPoints() {
      heartFlowers.length = 0;
      const totalFlowers = 320;
      
      for (let i = 0; i < totalFlowers; i++) {
        // Ecuación paramétrica de corazón
        const t = Math.random() * Math.PI * 2;
        const r = Math.sqrt(Math.random()); // Distribución uniforme interna
        
        let hx = 16 * Math.pow(Math.sin(t), 3);
        let hy = -(13 * Math.cos(t) - 5 * Math.cos(2*t) - 2 * Math.cos(3*t) - Math.cos(4*t));

        hx *= r * 11;
        hy *= r * 11;

        heartFlowers.push({
          relX: hx,
          relY: hy - 140, // Elevar sobre el tronco
          size: 7 + Math.random() * 5,
          delay: Math.random()
        });
      }
    }

    // Estructura fija de las ramas
    const branches = [
      { sx: 0, sy: -120, ex: -60, ey: -200, w: 10 },
      { sx: 0, sy: -140, ex: 65, ey: -210, w: 9 },
      { sx: -40, sy: -170, ex: -110, ey: -250, w: 6 },
      { sx: 40, sy: -180, ex: 105, ey: -260, w: 6 },
      { sx: 0, sy: -180, ex: -20, ey: -280, w: 7 },
      { sx: -20, sy: -220, ex: 30, ey: -290, w: 5 },
    ];

    // Bucle principal de animación
    function loop() {
      drawBase();

      const centerX = width / 2 + treeOffsetX;
      const groundY = height * 0.75;

      // 1. Dibujar Tronco
      if (trunkProgress > 0) {
        ctx.beginPath();
        ctx.moveTo(centerX, groundY);
        const currentTrunkHeight = 180 * trunkProgress;
        
        // Tronco grueso con ligera curvatura
        ctx.quadraticCurveTo(
          centerX - 10, groundY - currentTrunkHeight / 2, 
          centerX, groundY - currentTrunkHeight
        );
        ctx.strokeStyle = '#1b5e20';
        ctx.lineWidth = 22;
        ctx.lineCap = 'round';
        ctx.stroke();
      }

      // 2. Dibujar Ramas
      if (branchProgress > 0) {
        branches.forEach(b => {
          ctx.beginPath();
          ctx.moveTo(centerX + b.sx, groundY + b.sy);
          
          const curEx = b.sx + (b.ex - b.sx) * branchProgress;
          const curEy = b.sy + (b.ey - b.sy) * branchProgress;
          
          ctx.lineTo(centerX + curEx, groundY + curEy);
          ctx.strokeStyle = '#1b5e20';
          ctx.lineWidth = b.w;
          ctx.stroke();
        });
      }

      // 3. Dibujar Flores en Corazón
      if (flowerProgress > 0) {
        heartFlowers.forEach(f => {
          if (flowerProgress >= f.delay) {
            const scale = Math.min(1, (flowerProgress - f.delay) * 3);
            drawSunflower(centerX + f.relX, groundY + f.relY, f.size * scale);
          }
        });
      }

      // Animaciones de transición de estados
      if (animState === 'trunk') {
        trunkProgress += 0.02;
        if (trunkProgress >= 1) {
          trunkProgress = 1;
          animState = 'branches';
        }
      } else if (animState === 'branches') {
        branchProgress += 0.025;
        if (branchProgress >= 1) {
          branchProgress = 1;
          animState = 'flowers';
        }
      } else if (animState === 'flowers') {
        flowerProgress += 0.015;
        if (flowerProgress >= 1) {
          flowerProgress = 1;
          setTimeout(() => {
            animState = 'shift';
            targetOffsetX = width * 0.22; // Desplazar hacia la derecha
          }, 800);
        }
      } else if (animState === 'shift') {
        // Interpola movimiento suave
        treeOffsetX += (targetOffsetX - treeOffsetX) * 0.05;
        if (Math.abs(targetOffsetX - treeOffsetX) < 1) {
          treeOffsetX = targetOffsetX;
          animState = 'typing';
          startTypingEffect();
        }
      }

      requestAnimationFrame(loop);
    }

    // Iniciar secuencia al presionar el botón
    function startAnimation() {
      document.getElementById('start-btn').style.opacity = '0';
      setTimeout(() => {
        document.getElementById('start-btn').style.display = 'none';
      }, 500);

      generateHeartPoints();
      animState = 'trunk';
    }

    // Efecto de mecanografía paso a paso para el texto
    function startTypingEffect() {
      const titleEl = document.getElementById('title');
      const textEl = document.getElementById('typed-text');
      const footerEl = document.getElementById('footer-text');

      const titleText = "🌻 FELIZ DÍA DE LAS FLORES AMARILLAS 🌻";
      const bodyText = "CADA GIRASOL QUE VES\nAQUÍ ES UN LATIDO DE MI\nCORAZÓN.\nASÍ COMO EL SOL ILUMINA\nLOS CAMPOS, TÚ ILUMINAS\nMI VIDA.\nQUE ESTAS FLORES TE\nRECUERDEN LO ESPECIAL\nQUE ERES PARA MÍ.\n¡TE AMO!";
      const footerText = "Eres el sol que hace florecer cada uno de mis días.";

      // Escribir título
      let i = 0;
      titleEl.innerHTML = '<span class="cursor"></span>';
      
      function typeTitle() {
        if (i < titleText.length) {
          titleEl.innerHTML = titleText.substring(0, i + 1) + '<span class="cursor"></span>';
          i++;
          setTimeout(typeTitle, 40);
        } else {
          titleEl.innerHTML = titleText;
          typeBody();
        }
      }

      // Escribir cuerpo del mensaje
      let j = 0;
      function typeBody() {
        textEl.innerHTML = '<span class="cursor"></span>';
        function stepBody() {
          if (j < bodyText.length) {
            textEl.innerHTML = bodyText.substring(0, j + 1) + '<span class="cursor"></span>';
            j++;
            setTimeout(stepBody, 35);
          } else {
            textEl.innerHTML = bodyText;
            typeFooter();
          }
        }
        stepBody();
      }

      // Escribir nota final al pie
      let k = 0;
      function typeFooter() {
        footerEl.innerHTML = '<span class="cursor"></span>';
        function stepFooter() {
          if (k < footerText.length) {
            footerEl.innerHTML = footerText.substring(0, k + 1) + '<span class="cursor"></span>';
            k++;
            setTimeout(stepFooter, 30);
          } else {
            footerEl.innerHTML = footerText;
          }
        }
        stepFooter();
      }

      typeTitle();
    }

    // Iniciar bucle de renderizado
    loop();
  </script>
</body>
</html>
