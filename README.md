<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Luz Verde, Luz Roja — Mini Juego</title>
  <style>
    :root {
      --bg: #0f1220;
      --panel: #161a2b;
      --accent: #7dd3fc;
      --win: #10b981;
      --lose: #ef4444;
      --text: #e5e7eb;
      --muted: #9ca3af;
      --track: #1f243b;
    }

    * { box-sizing: border-box; }
    html, body { height: 100%; margin: 0; background: radial-gradient(1000px 600px at 70% -20%, #1c2240 0%, var(--bg) 60%); font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, "Helvetica Neue", Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji"; color: var(--text); }

    .wrap { display: grid; place-items: center; min-height: 100%; padding: 24px; }

    .card { width: min(900px, 95vw); background: color-mix(in oklab, var(--panel) 92%, black 8%);
            border: 1px solid rgba(255,255,255,.06); border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,.35); overflow: hidden; }

    header { display: flex; align-items: center; justify-content: space-between; gap: 12px; padding: 16px 18px; border-bottom: 1px solid rgba(255,255,255,.06); background: linear-gradient(180deg, rgba(255,255,255,.03), rgba(255,255,255,0)); }
    header h1 { font-size: clamp(18px, 2.5vw, 22px); margin: 0; letter-spacing: .4px; font-weight: 700; }

    .badges { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
    .pill { padding: 6px 10px; border-radius: 999px; font-size: 12px; letter-spacing: .3px; background: #0b1222; border: 1px solid rgba(255,255,255,.08); color: var(--muted); }

    .state { font-weight: 800; padding: 6px 12px; border-radius: 999px; font-size: 12px; letter-spacing: .6px; text-transform: uppercase; }
    .state.green { background: rgba(16,185,129,.15); color: #34d399; border: 1px solid rgba(16,185,129,.35); }
    .state.red   { background: rgba(239,68,68,.15);  color: #f87171; border: 1px solid rgba(239,68,68,.35); }

    canvas { display: block; width: 100%; height: auto; background: linear-gradient(#0b1020 0 55%, #0a0f1a 55% 100%); }

    .panel { display: grid; grid-template-columns: 1fr auto; gap: 12px; align-items: center; padding: 14px 18px; border-top: 1px solid rgba(255,255,255,.06); background: linear-gradient(180deg, rgba(255,255,255,.02), rgba(255,255,255,0)); }
    .meta { display: flex; gap: 18px; align-items: center; font-size: 13px; color: var(--muted); }
    .meta strong { color: var(--text); }

    .controls { display: flex; gap: 10px; }
    button { appearance: none; border: 1px solid rgba(255,255,255,.12); background: #0b1222; color: var(--text); padding: 10px 14px; border-radius: 12px; font-weight: 700; letter-spacing: .3px; cursor: pointer; transition: transform .06s ease, box-shadow .2s ease, border-color .2s ease; }
    button:hover { border-color: rgba(255,255,255,.25); box-shadow: 0 6px 20px rgba(0,0,0,.25); transform: translateY(-1px); }
    button:active { transform: translateY(0); }

    .move-btn { display: none; }

    @media (max-width: 700px) {
      .panel { grid-template-columns: 1fr; }
      .move-btn { display: inline-flex; }
    }

    .toast { position: absolute; inset: 0; display: grid; place-items: center; pointer-events: none; }
    .toast .bubble { padding: 14px 16px; border-radius: 12px; backdrop-filter: blur(8px); background: rgba(0,0,0,.4); border: 1px solid rgba(255,255,255,.08); font-size: 13px; color: var(--muted); }
    .sr-only { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0,0,0,0); white-space: nowrap; border: 0; }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <header>
        <h1>🟢 Luz Verde, 🔴 Luz Roja — Mini Juego</h1>
        <div class="badges">
          <span class="pill" id="statusPill">Preparado</span>
          <span class="state green" id="lightState">Luz Verde</span>
        </div>
      </header>

      <canvas id="game" width="900" height="380" aria-label="Lienzo del juego"></canvas>

      <div class="panel">
        <div class="meta">
          <div>Progreso: <strong><span id="progress">0</span>%</strong></div>
          <div>Rango de luz: <strong><span id="rng">—</span></strong></div>
          <div>Mejor tiempo: <strong><span id="best">—</span></strong></div>
        </div>
        <div class="controls">
          <button id="startBtn" title="Iniciar (Barra espaciadora)">Iniciar</button>
          <button id="moveBtn" class="move-btn" title="Mantener para avanzar">Mantener para avanzar</button>
          <button id="resetBtn" title="Reiniciar">Reiniciar</button>
        </div>
      </div>
    </div>
  </div>

  <div class="toast" aria-live="polite" aria-atomic="true">
    <div class="bubble" id="hint">Controles: mantén presionado <strong>W / Flecha ↑</strong> (o botón “Mantener para avanzar” en móvil). Suelta en 🔴.</div>
  </div>

  <span class="sr-only" id="sr-status" role="status"></span>

  <script>
    // --- Parámetros del juego ---
    const WORLD_LEN = 1500;       // distancia a la meta en px
    const PLAYER_SPEED = 140;     // px/seg al moverse
    const FRICTION = 6.0;         // freno cuando suelta la tecla
    const TOLERANCE = 12;         // px/seg de tolerancia en rojo (para no castigar micro-movimiento)

    const GREEN_RANGE = [1200, 2400]; // ms mínimo/máximo luz verde
    const RED_RANGE   = [900, 1700];  // ms mínimo/máximo luz roja

    // --- Estado ---
    let running = false, gameOver = false, won = false;
    let lightIsGreen = true; // inicia en verde
    let player = { x: 40, y: 300, v: 0 };
    let dollAngle = 0; // 0 mira atrás (seguro), 1 mira al jugador (riesgo)
    let tLast = 0, acc = 0;
    let bestTime = null; let runTime = 0;

    // --- DOM ---
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const progressEl = document.getElementById('progress');
    const rngEl = document.getElementById('rng');
    const statusPill = document.getElementById('statusPill');
    const lightStateEl = document.getElementById('lightState');
    const startBtn = document.getElementById('startBtn');
    const resetBtn = document.getElementById('resetBtn');
    const moveBtn = document.getElementById('moveBtn');
    const srStatus = document.getElementById('sr-status');

    // --- Utilidades ---
    const clamp = (v, a, b) => Math.max(a, Math.min(b, v));
    const randRange = ([a, b]) => Math.floor(a + Math.random() * (b - a));

    function announce(msg) { srStatus.textContent = msg; }

    // --- Máquina de estados de luz ---
    let lightTimer = 0, lightTTL = randRange(GREEN_RANGE);

    function toggleLight(force) {
      lightIsGreen = force ?? !lightIsGreen;
      dollAngle = lightIsGreen ? 0 : 1; // 0 = seguro (de espaldas), 1 = mirando
      lightTTL = lightIsGreen ? randRange(GREEN_RANGE) : randRange(RED_RANGE);
      lightStateEl.className = `state ${lightIsGreen ? 'green' : 'red'}`;
      lightStateEl.textContent = lightIsGreen ? 'Luz Verde' : 'Luz Roja';
      rngEl.textContent = `${lightTTL} ms`;
      announce(lightIsGreen ? 'Luz verde' : 'Luz roja');
    }

    // --- Entrada ---
    let movingInput = false;
    function handleDown(e) {
      if (['w','ArrowUp','W',' '].includes(e.key)) { movingInput = e.key !== ' '; if (e.key === ' ') startRun(); }
    }
    function handleUp(e) { if (['w','ArrowUp','W'].includes(e.key)) movingInput = false; }

    window.addEventListener('keydown', handleDown);
    window.addEventListener('keyup', handleUp);

    // Móvil: botón mantener
    moveBtn.addEventListener('pointerdown', () => movingInput = true);
    moveBtn.addEventListener('pointerup',   () => movingInput = false);
    moveBtn.addEventListener('pointerleave',() => movingInput = false);

    startBtn.addEventListener('click', startRun);
    resetBtn.addEventListener('click', reset);

    function startRun() {
      if (running) return; reset(false); running = true; tLast = performance.now(); requestAnimationFrame(loop);
      statusPill.textContent = 'En juego';
    }

    function reset(withLight = true) {
      running = false; gameOver = false; won = false; player.x = 40; player.v = 0; runTime = 0; statusPill.textContent = 'Preparado';
      if (withLight) toggleLight(true); // forzar a verde al reiniciar
    }

    // --- Lógica principal ---
    function step(dt) {
      // actualizar luz
      lightTimer += dt;
      if (lightTimer >= lightTTL) { lightTimer = 0; toggleLight(); }

      // física simple: aplicar input
      const targetV = movingInput ? PLAYER_SPEED : 0;
      const diff = targetV - player.v;
      // aceleración/frenado suave
      player.v += diff * clamp(dt * (movingInput ? 8 : FRICTION), 0, 1);

      // detectar trampa en rojo
      if (!lightIsGreen && Math.abs(player.v) > TOLERANCE && running && !gameOver && !won) {
        gameOver = true; running = false; statusPill.textContent = 'Eliminado';
        flash('#ef4444');
      }

      // integrar posición
      player.x += player.v * dt;
      player.x = clamp(player.x, 40, WORLD_LEN);

      // victoria
      if (player.x >= WORLD_LEN && !gameOver && running) {
        won = true; running = false; statusPill.textContent = '¡Ganaste!';
        if (!bestTime || runTime < bestTime) bestTime = runTime;
        flash('#10b981');
      }

      // progreso y tiempo
      const prog = Math.round(((player.x - 40) / (WORLD_LEN - 40)) * 100);
      progressEl.textContent = clamp(prog, 0, 100);
      runTime += running ? dt : 0;
      document.getElementById('best').textContent = bestTime ? `${bestTime.toFixed(2)} s` : '—';
    }

    function loop(now) {
      const dt = (now - tLast) / 1000; tLast = now;
      step(dt);
      draw();
      if (running) requestAnimationFrame(loop);
    }

    // --- Render ---
    function draw() {
      const w = canvas.width, h = canvas.height;
      ctx.clearRect(0,0,w,h);

      // Cielo y suelo ya vienen del background del canvas

      // Pista
      const trackY = 320, trackH = 40;
      ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--track');
      ctx.fillRect(0, trackY, w, trackH);

      // Meta (bandera)
      const camX = clamp(player.x - 160, 0, WORLD_LEN - w);
      const goalX = WORLD_LEN - camX;
      ctx.save();
      ctx.translate(-camX, 0);
      drawGoal(WORLD_LEN, trackY);

      // Marca de salida
      drawPost(40, trackY, '#64748b');

      // Muñeca (juez)
      drawDoll(200, 150, dollAngle);

      // Jugador
      drawPlayer(player.x, trackY);

      ctx.restore();
    }

    function drawGoal(x, trackY){
      const poleH = 220; const poleW = 10;
      ctx.fillStyle = '#94a3b8';
      ctx.fillRect(x, trackY - poleH, poleW, poleH);
      // bandera cuadriculada
      const flagW = 48, flagH = 32; const cell = 8;
      for (let i=0;i<flagW/cell;i++) for (let j=0;j<flagH/cell;j++){
        ctx.fillStyle = (i+j)%2===0 ? '#111827' : '#e5e7eb';
        ctx.fillRect(x + poleW + i*cell, trackY - poleH + 6 + j*cell, cell, cell);
      }
    }

    function drawPost(x, trackY, color){
      ctx.fillStyle = color; ctx.fillRect(x-4, trackY-200, 8, 200);
    }

    function drawDoll(x, y, look){
      // cuerpo
      ctx.fillStyle = '#f59e0b';
      ctx.beginPath(); ctx.arc(x, y, 28, 0, Math.PI*2); ctx.fill();
      // ojos (mirando jugador cuando look=1)
      ctx.fillStyle = '#0f172a';
      const dx = look ? 8 : -8;
      ctx.beginPath(); ctx.arc(x-8+dx*0.4, y-5, 4, 0, Math.PI*2); ctx.fill();
      ctx.beginPath(); ctx.arc(x+8+dx*0.6, y-5, 4, 0, Math.PI*2); ctx.fill();
      // cuello
      ctx.fillStyle = '#fbbf24'; ctx.fillRect(x-6, y+28, 12, 12);
      // torso
      ctx.fillStyle = '#22c55e'; ctx.fillRect(x-20, y+40, 40, 50);
    }

    function drawPlayer(x, trackY){
      const px = x, py = trackY - 10;
      // sombra
      ctx.fillStyle = 'rgba(0,0,0,.35)'; ctx.beginPath(); ctx.ellipse(px, py+14, 16, 6, 0, 0, Math.PI*2); ctx.fill();
      // cuerpo
      ctx.fillStyle = '#60a5fa'; ctx.beginPath(); ctx.arc(px, py-18, 12, 0, Math.PI*2); ctx.fill();
      ctx.fillStyle = '#a78bfa'; ctx.fillRect(px-9, py-8, 18, 20);
      // piernas anim simples
      const t = performance.now()/120; const step = Math.sin(t)*4*(player.v/PLAYER_SPEED);
      ctx.fillStyle = '#93c5fd';
      ctx.fillRect(px-7, py+12, 6, 10+step);
      ctx.fillRect(px+1, py+12, 6, 10-step);
    }

    function flash(color){
      const el = document.body;
      el.animate([
        { boxShadow: 'inset 0 0 0 0 rgba(0,0,0,0)' },
        { boxShadow: `inset 0 0 0 9999px ${color}22` },
        { boxShadow: 'inset 0 0 0 0 rgba(0,0,0,0)' }
      ], { duration: 600, easing: 'ease' });
    }

    // Inicialización
    toggleLight(true);
  </script>
</body>
</html>
