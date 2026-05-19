<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NEXUS — Agente Autônomo</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:ital,wght@0,300;0,400;0,600;0,700;1,400&family=Rajdhani:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;}

:root {
  --bg: #03060a;
  --bg2: #060d14;
  --surface: #0a1520;
  --surface2: #0d1c2a;
  --border: #112233;
  --border2: #1a3344;
  --green: #00ff88;
  --green2: #00cc66;
  --cyan: #00d4ff;
  --amber: #ffaa00;
  --red: #ff3355;
  --purple: #8844ff;
  --text: #b8d4e8;
  --text-dim: #4a6880;
  --text-bright: #e8f4ff;
}

body {
  font-family: 'Rajdhani', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  overflow-x: hidden;
  position: relative;
}

/* Scanline effect */
body::before {
  content:'';
  position:fixed;
  inset:0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0,255,136,0.015) 2px,
    rgba(0,255,136,0.015) 4px
  );
  pointer-events:none;
  z-index:999;
}

/* Circuit grid */
body::after {
  content:'';
  position:fixed;
  inset:0;
  background-image:
    linear-gradient(rgba(0,212,255,0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0,212,255,0.04) 1px, transparent 1px);
  background-size: 32px 32px;
  pointer-events:none;
  z-index:0;
}

/* Glow orb */
.orb {
  position:fixed;
  width:50vw; height:50vw;
  border-radius:50%;
  pointer-events:none;
  z-index:0;
  filter:blur(80px);
  opacity:0.06;
}
.orb1 { top:-10%; left:-10%; background:var(--cyan); animation: orbFloat1 15s ease-in-out infinite alternate; }
.orb2 { bottom:-10%; right:-10%; background:var(--green); animation: orbFloat2 18s ease-in-out infinite alternate; }

@keyframes orbFloat1 { to { transform: translate(5%, 5%); } }
@keyframes orbFloat2 { to { transform: translate(-5%, -5%); } }

/* ===== HEADER ===== */
header {
  position:relative; z-index:10;
  display:flex; align-items:center; justify-content:space-between;
  padding: 12px 16px;
  border-bottom: 1px solid var(--border);
  background: rgba(3,6,10,0.9);
  backdrop-filter: blur(16px);
  flex-shrink:0;
}

.logo-wrap { display:flex; align-items:center; gap:10px; }

.logo-hex {
  width:36px; height:36px;
  background: linear-gradient(135deg, var(--green), var(--cyan));
  clip-path: polygon(50% 0%, 100% 25%, 100% 75%, 50% 100%, 0% 75%, 0% 25%);
  display:flex; align-items:center; justify-content:center;
  font-size:14px; font-weight:700; color:#000;
  animation: hexPulse 2s ease-in-out infinite;
  flex-shrink:0;
}

@keyframes hexPulse {
  0%,100% { filter: drop-shadow(0 0 6px rgba(0,255,136,0.5)); }
  50% { filter: drop-shadow(0 0 16px rgba(0,255,136,0.9)); }
}

.logo-title {
  font-size:20px; font-weight:700; letter-spacing:4px;
  color: var(--text-bright);
  text-shadow: 0 0 20px rgba(0,212,255,0.4);
}

.logo-sub {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--text-dim); letter-spacing:2px;
  text-transform:uppercase; margin-top:2px;
}

.status-dot {
  display:flex; align-items:center; gap:6px;
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--text-dim); letter-spacing:1px;
}

.dot {
  width:7px; height:7px; border-radius:50%;
  background:var(--text-dim);
  transition: background 0.3s, box-shadow 0.3s;
}
.dot.active { background:var(--green); box-shadow:0 0 8px var(--green); animation: dotBlink 1s ease-in-out infinite; }
.dot.working { background:var(--amber); box-shadow:0 0 8px var(--amber); animation: dotBlink 0.5s ease-in-out infinite; }
.dot.error { background:var(--red); box-shadow:0 0 8px var(--red); }

@keyframes dotBlink { 0%,100%{opacity:1} 50%{opacity:0.3} }

/* ===== SETUP SCREEN ===== */
#setup-screen {
  position:relative; z-index:10;
  flex:1; display:flex; flex-direction:column;
  align-items:center; justify-content:center;
  padding:24px 16px; gap:16px;
}

.setup-card {
  width:100%; max-width:460px;
  background: var(--surface);
  border: 1px solid var(--border2);
  border-radius:16px;
  padding:28px 22px;
  display:flex; flex-direction:column; gap:20px;
  box-shadow: 0 0 60px rgba(0,0,0,0.8), 0 0 30px rgba(0,212,255,0.05);
  position:relative; overflow:hidden;
}

.setup-card::before {
  content:'';
  position:absolute; top:0; left:0; right:0; height:2px;
  background: linear-gradient(90deg, transparent, var(--cyan), var(--green), transparent);
}

.setup-header { display:flex; flex-direction:column; gap:6px; }

.setup-title {
  font-size:24px; font-weight:700; color:var(--text-bright);
  letter-spacing:1px; line-height:1.2;
}

.setup-title .hi { color:var(--green); }

.setup-desc {
  font-family:'JetBrains Mono', monospace;
  font-size:11px; color:var(--text-dim); line-height:1.7;
}

.setup-desc a { color:var(--cyan); text-decoration:none; }

.field-wrap { display:flex; flex-direction:column; gap:6px; }

.field-label {
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--text-dim);
  letter-spacing:2px; text-transform:uppercase;
  display:flex; align-items:center; gap:6px;
}

.field-label .tag {
  background:rgba(0,255,136,0.1); color:var(--green);
  border:1px solid rgba(0,255,136,0.2);
  padding:1px 6px; border-radius:3px; font-size:9px;
}

.nexus-input {
  background: var(--surface2);
  border: 1px solid var(--border2);
  border-radius:10px;
  padding:12px 14px;
  color:var(--text-bright);
  font-family:'JetBrains Mono', monospace;
  font-size:12px; outline:none; width:100%;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.nexus-input:focus {
  border-color:var(--cyan);
  box-shadow:0 0 0 3px rgba(0,212,255,0.08), 0 0 16px rgba(0,212,255,0.1);
}

.nexus-input::placeholder { color:var(--text-dim); }

/* Model selector */
.model-grid { display:grid; grid-template-columns:1fr 1fr; gap:8px; }

.model-btn {
  background:var(--surface2); border:1px solid var(--border);
  border-radius:10px; padding:10px 12px;
  color:var(--text-dim); font-family:'JetBrains Mono', monospace;
  font-size:10px; cursor:pointer; transition:all 0.2s;
  text-align:left; display:flex; flex-direction:column; gap:3px;
}
.model-btn:hover { border-color:var(--border2); color:var(--text); }
.model-btn.active { border-color:var(--green); background:rgba(0,255,136,0.06); color:var(--green); }
.model-name { font-size:11px; font-weight:600; color:inherit; }
.model-desc { font-size:9px; color:var(--text-dim); }
.model-btn.active .model-desc { color:rgba(0,255,136,0.6); }

.launch-btn {
  background: linear-gradient(135deg, #004422, #006633);
  border: 1px solid var(--green2);
  border-radius:10px; padding:14px;
  color:var(--green); font-family:'Rajdhani', sans-serif;
  font-size:16px; font-weight:700; cursor:pointer;
  transition:all 0.2s; letter-spacing:2px; text-transform:uppercase;
  position:relative; overflow:hidden;
}

.launch-btn::before {
  content:'';
  position:absolute; inset:0;
  background:linear-gradient(135deg, transparent, rgba(0,255,136,0.1), transparent);
  transform:translateX(-100%);
  transition:transform 0.4s;
}

.launch-btn:hover::before { transform:translateX(100%); }
.launch-btn:hover { box-shadow:0 0 24px rgba(0,255,136,0.3); }

.err-line {
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--red); display:none;
  padding:8px 12px; background:rgba(255,51,85,0.08);
  border:1px solid rgba(255,51,85,0.2); border-radius:6px;
}

/* ===== MAIN SCREEN ===== */
#main-screen {
  position:relative; z-index:10;
  flex:1; display:none; flex-direction:column; overflow:hidden;
}

/* Task input panel */
.task-panel {
  padding:14px 16px;
  border-bottom:1px solid var(--border);
  background:rgba(3,6,10,0.8);
  backdrop-filter:blur(12px);
  flex-shrink:0;
}

.task-panel-header {
  display:flex; align-items:center; justify-content:space-between;
  margin-bottom:10px;
}

.panel-title {
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--cyan); letter-spacing:2px; text-transform:uppercase;
  display:flex; align-items:center; gap:6px;
}

.panel-title::before {
  content:'▶'; font-size:8px;
}

.task-type-row {
  display:flex; gap:6px; margin-bottom:10px; flex-wrap:wrap;
}

.type-chip {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; padding:4px 10px; border-radius:4px;
  border:1px solid var(--border); color:var(--text-dim);
  cursor:pointer; transition:all 0.2s; letter-spacing:1px;
  background: var(--surface);
}
.type-chip:hover { border-color:var(--border2); color:var(--text); }
.type-chip.active { border-color:var(--amber); color:var(--amber); background:rgba(255,170,0,0.08); }

.task-input-row { display:flex; gap:8px; align-items:flex-end; }

#task-input {
  flex:1; background:var(--surface);
  border:1px solid var(--border2); border-radius:10px;
  padding:11px 14px; color:var(--text-bright);
  font-family:'Rajdhani', sans-serif; font-size:14px; font-weight:500;
  outline:none; resize:none; min-height:44px; max-height:100px;
  transition:border-color 0.2s, box-shadow 0.2s; line-height:1.4;
}

#task-input:focus {
  border-color:var(--amber);
  box-shadow:0 0 0 3px rgba(255,170,0,0.08);
}

#task-input::placeholder { color:var(--text-dim); font-weight:400; }

.execute-btn {
  height:44px; padding:0 16px;
  background:linear-gradient(135deg, #332200, #553300);
  border:1px solid var(--amber); border-radius:10px;
  color:var(--amber); font-family:'JetBrains Mono', monospace;
  font-size:11px; font-weight:700; cursor:pointer;
  transition:all 0.2s; letter-spacing:1px; white-space:nowrap;
  flex-shrink:0;
}

.execute-btn:hover { box-shadow:0 0 20px rgba(255,170,0,0.3); }
.execute-btn:disabled { opacity:0.4; cursor:not-allowed; }

/* ===== AGENT LOG ===== */
.agent-container {
  flex:1; overflow-y:auto; padding:14px 16px;
  display:flex; flex-direction:column; gap:12px;
  scroll-behavior:smooth;
}

.agent-container::-webkit-scrollbar { width:3px; }
.agent-container::-webkit-scrollbar-track { background:transparent; }
.agent-container::-webkit-scrollbar-thumb { background:var(--border2); border-radius:3px; }

/* Welcome state */
.welcome-state {
  flex:1; display:flex; flex-direction:column;
  align-items:center; justify-content:center;
  gap:14px; opacity:0.5; padding:32px;
  text-align:center;
}

.welcome-hex {
  font-size:48px;
  animation:float 4s ease-in-out infinite;
  filter: drop-shadow(0 0 20px rgba(0,255,136,0.4));
}

@keyframes float { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-12px)} }

.welcome-text {
  font-family:'JetBrains Mono', monospace;
  font-size:11px; color:var(--text-dim);
  line-height:1.8; letter-spacing:1px;
}

/* Task block */
.task-block {
  background:var(--surface);
  border:1px solid var(--border2);
  border-radius:12px; overflow:hidden;
  animation: blockIn 0.4s ease forwards;
  opacity:0; transform:translateY(12px);
}

@keyframes blockIn { to { opacity:1; transform:translateY(0); } }

.task-block-header {
  padding:12px 14px;
  border-bottom:1px solid var(--border);
  display:flex; align-items:center; justify-content:space-between;
  background:rgba(255,170,0,0.04);
}

.task-block-title {
  font-size:13px; font-weight:600; color:var(--amber);
  display:flex; align-items:center; gap:8px;
}

.task-id {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--text-dim);
  background:var(--surface2); padding:2px 6px; border-radius:3px;
}

.task-status {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; padding:3px 8px; border-radius:4px;
  letter-spacing:1px; text-transform:uppercase;
}

.task-status.running { color:var(--amber); border:1px solid rgba(255,170,0,0.3); background:rgba(255,170,0,0.08); }
.task-status.done { color:var(--green); border:1px solid rgba(0,255,136,0.3); background:rgba(0,255,136,0.08); }
.task-status.error { color:var(--red); border:1px solid rgba(255,51,85,0.3); background:rgba(255,51,85,0.08); }

/* Steps */
.steps-list { padding:12px 14px; display:flex; flex-direction:column; gap:8px; }

.step {
  display:flex; gap:10px; align-items:flex-start;
  animation: stepIn 0.3s ease forwards; opacity:0;
}

@keyframes stepIn { to { opacity:1; } }

.step-icon {
  width:20px; height:20px; border-radius:4px;
  display:flex; align-items:center; justify-content:center;
  font-size:10px; flex-shrink:0; margin-top:1px;
  font-family:'JetBrains Mono', monospace; font-weight:700;
}

.step-icon.pending { background:var(--surface2); color:var(--text-dim); border:1px solid var(--border); }
.step-icon.running { background:rgba(255,170,0,0.15); color:var(--amber); border:1px solid rgba(255,170,0,0.3); animation:iconPulse 0.8s infinite; }
.step-icon.done { background:rgba(0,255,136,0.15); color:var(--green); border:1px solid rgba(0,255,136,0.3); }
.step-icon.error { background:rgba(255,51,85,0.15); color:var(--red); border:1px solid rgba(255,51,85,0.3); }

@keyframes iconPulse { 0%,100%{opacity:1} 50%{opacity:0.5} }

.step-content { flex:1; min-width:0; }

.step-title {
  font-size:13px; font-weight:600; color:var(--text);
  margin-bottom:2px;
}

.step-title.running { color:var(--amber); }
.step-title.done { color:var(--text-bright); }

.step-detail {
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--text-dim); line-height:1.5;
}

/* Output block */
.output-block {
  margin:0 14px 14px;
  background: #020810;
  border:1px solid var(--border2);
  border-radius:10px; overflow:hidden;
}

.output-header {
  padding:8px 12px;
  border-bottom:1px solid var(--border);
  display:flex; align-items:center; justify-content:space-between;
  background:rgba(0,212,255,0.04);
}

.output-label {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--cyan); letter-spacing:2px; text-transform:uppercase;
  display:flex; align-items:center; gap:6px;
}

.copy-btn {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--text-dim);
  background:none; border:1px solid var(--border);
  border-radius:4px; padding:2px 8px; cursor:pointer;
  transition:all 0.2s; letter-spacing:1px;
}
.copy-btn:hover { color:var(--cyan); border-color:var(--cyan); }

.output-content {
  padding:14px; overflow-x:auto;
  font-family:'JetBrains Mono', monospace;
  font-size:11px; color:#a8d8b0; line-height:1.7;
  white-space:pre-wrap; word-break:break-word;
  max-height:360px; overflow-y:auto;
}

.output-content::-webkit-scrollbar { width:3px; height:3px; }
.output-content::-webkit-scrollbar-thumb { background:var(--border2); border-radius:3px; }

/* Thinking indicator */
.thinking {
  display:flex; align-items:center; gap:8px;
  padding:10px 14px;
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--text-dim);
}

.think-dots { display:flex; gap:4px; }
.think-dot {
  width:5px; height:5px; border-radius:50%;
  background:var(--amber); opacity:0.4;
  animation:thinkBlink 1.2s infinite;
}
.think-dot:nth-child(2) { animation-delay:0.2s; }
.think-dot:nth-child(3) { animation-delay:0.4s; }

@keyframes thinkBlink { 0%,100%{opacity:0.4;transform:scale(1)} 50%{opacity:1;transform:scale(1.3)} }

/* Bottom bar */
.bottom-bar {
  padding:8px 16px;
  border-top:1px solid var(--border);
  background:rgba(3,6,10,0.9);
  backdrop-filter:blur(12px);
  display:flex; align-items:center; justify-content:space-between;
  flex-shrink:0;
}

.bar-info {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--text-dim); letter-spacing:1px;
  display:flex; align-items:center; gap:12px;
}

.bar-stat { display:flex; align-items:center; gap:4px; }
.bar-stat .val { color:var(--text); }

.clear-all {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--text-dim);
  background:none; border:1px solid var(--border);
  border-radius:4px; padding:3px 8px; cursor:pointer;
  transition:all 0.2s; letter-spacing:1px;
}
.clear-all:hover { color:var(--red); border-color:var(--red); }

/* Progress bar */
.progress-bar {
  height:2px; background:var(--border);
  margin:0 14px 0; border-radius:1px; overflow:hidden;
}
.progress-fill {
  height:100%; background:linear-gradient(90deg, var(--cyan), var(--green));
  transition:width 0.5s ease;
  box-shadow: 0 0 8px var(--green);
}

/* Save files section */
.files-section {
  margin:0 14px 14px;
  display:flex; flex-direction:column; gap:6px;
}

.files-title {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--purple); letter-spacing:2px; text-transform:uppercase;
  display:flex; align-items:center; gap:6px;
}

.file-item {
  display:flex; align-items:center; justify-content:space-between;
  background:rgba(136,68,255,0.06); border:1px solid rgba(136,68,255,0.2);
  border-radius:6px; padding:8px 12px;
}

.file-name {
  font-family:'JetBrains Mono', monospace;
  font-size:10px; color:var(--purple);
}

.download-btn {
  font-family:'JetBrains Mono', monospace;
  font-size:9px; color:var(--purple);
  background:none; border:1px solid rgba(136,68,255,0.3);
  border-radius:4px; padding:3px 8px; cursor:pointer;
  transition:all 0.2s; letter-letters:1px;
}
.download-btn:hover { background:rgba(136,68,255,0.1); }
</style>
</head>
<body>

<div class="orb orb1"></div>
<div class="orb orb2"></div>

<!-- HEADER -->
<header>
  <div class="logo-wrap">
    <div class="logo-hex">N</div>
    <div>
      <div class="logo-title">NEXUS</div>
      <div class="logo-sub">Agente Autônomo · Gemini</div>
    </div>
  </div>
  <div class="status-dot">
    <div class="dot" id="status-dot"></div>
    <span id="status-text">OFFLINE</span>
  </div>
</header>

<!-- SETUP -->
<div id="setup-screen">
  <div class="setup-card">
    <div class="setup-header">
      <div class="setup-title"><span class="hi">NEXUS</span> — Agente Autônomo</div>
      <div class="setup-desc">
        Sistema de execução autônoma de tarefas.<br>
        Powered by Google Gemini API.<br><br>
        Obtenha sua chave gratuita em<br>
        <a href="https://aistudio.google.com/app/apikey" target="_blank">→ aistudio.google.com/app/apikey</a>
      </div>
    </div>

    <div class="field-wrap">
      <div class="field-label">
        API Key Gemini <span class="tag">SALVA LOCALMENTE</span>
      </div>
      <input class="nexus-input" id="api-key-input" type="password"
        placeholder="AIzaSy..." autocomplete="off" spellcheck="false">
    </div>

    <div class="field-wrap">
      <div class="field-label">Modelo</div>
      <div class="model-grid">
        <button class="model-btn active" data-model="gemini-2.0-flash">
          <div class="model-name">Gemini 2.0 Flash</div>
          <div class="model-desc">Rápido · Recomendado</div>
        </button>
        <button class="model-btn" data-model="gemini-2.0-flash-lite">
          <div class="model-name">Gemini 2.0 Flash Lite</div>
          <div class="model-desc">Leve · Mais requisições</div>
        </button>
        <button class="model-btn" data-model="gemini-2.5-flash-preview-04-17">
          <div class="model-name">Gemini 2.5 Flash</div>
          <div class="model-desc">Avançado · Mais poderoso</div>
        </button>
        <button class="model-btn" data-model="gemini-2.5-pro-preview-05-06">
          <div class="model-name">Gemini 2.5 Pro</div>
          <div class="model-desc">Máximo · Mais lento</div>
        </button>
      </div>
    </div>

    <div class="err-line" id="err-line">⚠ Erro de conexão. Verifique sua API Key.</div>

    <button class="launch-btn" id="launch-btn">▶ INICIALIZAR NEXUS</button>
  </div>
</div>

<!-- MAIN -->
<div id="main-screen">

  <!-- Task panel -->
  <div class="task-panel">
    <div class="task-panel-header">
      <div class="panel-title">Nova Tarefa</div>
    </div>
    <div class="task-type-row">
      <div class="type-chip active" data-type="site">🌐 Site</div>
      <div class="type-chip" data-type="script">⚙️ Script</div>
      <div class="type-chip" data-type="bot">🤖 Bot</div>
      <div class="type-chip" data-type="automacao">⚡ Automação</div>
      <div class="type-chip" data-type="api">🔗 API</div>
      <div class="type-chip" data-type="outro">✦ Outro</div>
    </div>
    <div class="task-input-row">
      <textarea id="task-input" rows="1"
        placeholder="Descreva a tarefa com detalhes... Ex: Crie um site de portfólio completo com HTML, CSS e JS"></textarea>
      <button class="execute-btn" id="execute-btn">▶ EXECUTAR</button>
    </div>
  </div>

  <!-- Agent log -->
  <div class="agent-container" id="agent-container">
    <div class="welcome-state" id="welcome-state">
      <div class="welcome-hex">⬡</div>
      <div class="welcome-text">
        NEXUS ONLINE · AGUARDANDO TAREFA<br>
        Digite sua tarefa acima e pressione EXECUTAR<br>
        O agente trabalhará até concluir completamente
      </div>
    </div>
  </div>

  <!-- Bottom bar -->
  <div class="bottom-bar">
    <div class="bar-info">
      <div class="bar-stat">TAREFAS: <span class="val" id="stat-tasks">0</span></div>
      <div class="bar-stat">MODELO: <span class="val" id="stat-model">—</span></div>
    </div>
    <button class="clear-all" id="clear-all">↺ LIMPAR</button>
  </div>

</div>

<script>
// ===== STATE =====
let API_KEY = '';
let MODEL = 'gemini-2.0-flash';
let taskType = 'site';
let taskCount = 0;
let isRunning = false;

// ===== SETUP =====
document.querySelectorAll('.model-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.model-btn').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    MODEL = btn.dataset.model;
  });
});

document.querySelectorAll('.type-chip').forEach(chip => {
  chip.addEventListener('click', () => {
    document.querySelectorAll('.type-chip').forEach(c => c.classList.remove('active'));
    chip.classList.add('active');
    taskType = chip.dataset.type;
  });
});

document.getElementById('launch-btn').addEventListener('click', async () => {
  const key = document.getElementById('api-key-input').value.trim();
  const err = document.getElementById('err-line');

  if (!key || key.length < 20) {
    err.style.display = 'block';
    err.textContent = '⚠ Cole uma API Key válida (começa com AIza...)';
    return;
  }

  err.style.display = 'none';
  API_KEY = key;
  localStorage.setItem('nexus_api_key', key);
  localStorage.setItem('nexus_model', MODEL);

  document.getElementById('setup-screen').style.display = 'none';
  const main = document.getElementById('main-screen');
  main.style.display = 'flex';
  main.style.flexDirection = 'column';

  setStatus('active', 'ONLINE');
  document.getElementById('stat-model').textContent = MODEL.replace('gemini-', '').toUpperCase();
  document.getElementById('task-input').focus();
});

document.getElementById('api-key-input').addEventListener('keydown', e => {
  if (e.key === 'Enter') document.getElementById('launch-btn').click();
});

// Restore saved key
window.addEventListener('load', () => {
  const saved = localStorage.getItem('nexus_api_key');
  const savedModel = localStorage.getItem('nexus_model');
  if (saved) document.getElementById('api-key-input').value = saved;
  if (savedModel) {
    MODEL = savedModel;
    document.querySelectorAll('.model-btn').forEach(b => {
      b.classList.toggle('active', b.dataset.model === savedModel);
    });
  }
});

// ===== STATUS =====
function setStatus(state, text) {
  const dot = document.getElementById('status-dot');
  const txt = document.getElementById('status-text');
  dot.className = 'dot ' + state;
  txt.textContent = text;
}

// ===== EXECUTE =====
document.getElementById('execute-btn').addEventListener('click', runTask);

document.getElementById('task-input').addEventListener('keydown', e => {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); runTask(); }
});

document.getElementById('task-input').addEventListener('input', function() {
  this.style.height = 'auto';
  this.style.height = Math.min(this.scrollHeight, 100) + 'px';
});

document.getElementById('clear-all').addEventListener('click', () => {
  document.getElementById('agent-container').innerHTML = `
    <div class="welcome-state" id="welcome-state">
      <div class="welcome-hex">⬡</div>
      <div class="welcome-text">NEXUS ONLINE · AGUARDANDO TAREFA<br>Digite sua tarefa acima e pressione EXECUTAR<br>O agente trabalhará até concluir completamente</div>
    </div>`;
  taskCount = 0;
  document.getElementById('stat-tasks').textContent = '0';
});

async function runTask() {
  if (isRunning) return;
  const input = document.getElementById('task-input');
  const taskText = input.value.trim();
  if (!taskText) return;

  input.value = '';
  input.style.height = 'auto';
  isRunning = true;
  setStatus('working', 'EXECUTANDO');
  document.getElementById('execute-btn').disabled = true;

  const welcome = document.getElementById('welcome-state');
  if (welcome) welcome.remove();

  taskCount++;
  document.getElementById('stat-tasks').textContent = taskCount;

  const taskId = 'TASK-' + String(taskCount).padStart(3, '0');
  const container = document.getElementById('agent-container');

  // Create task block
  const block = document.createElement('div');
  block.className = 'task-block';
  block.id = taskId;
  block.innerHTML = `
    <div class="task-block-header">
      <div class="task-block-title">
        <span class="task-id">${taskId}</span>
        ${taskText.length > 60 ? taskText.substring(0,60)+'...' : taskText}
      </div>
      <div class="task-status running" id="${taskId}-status">EXECUTANDO</div>
    </div>
    <div class="steps-list" id="${taskId}-steps"></div>
    <div class="progress-bar"><div class="progress-fill" id="${taskId}-progress" style="width:0%"></div></div>
  `;
  container.appendChild(block);
  container.scrollTop = container.scrollHeight;

  try {
    // PHASE 1: Planning
    await addStep(taskId, 1, 'running', '⚡', 'Analisando tarefa e criando plano de execução...');
    setProgress(taskId, 10);

    const planPrompt = `Você é um agente autônomo especialista em programação. Analise esta tarefa e crie um plano de execução detalhado.

TAREFA: ${taskText}
TIPO: ${taskType}

Responda APENAS com JSON no formato:
{
  "titulo": "título curto da tarefa",
  "complexidade": "simples|media|complexa",
  "linguagens": ["lista de linguagens/tecnologias"],
  "etapas": [
    {"id": 1, "nome": "Nome da etapa", "descricao": "O que será feito"},
    {"id": 2, "nome": "Nome da etapa", "descricao": "O que será feito"}
  ],
  "arquivos": ["arquivo1.html", "arquivo2.js"]
}

Crie entre 3 e 6 etapas práticas e específicas. Retorne APENAS o JSON, sem markdown ou texto extra.`;

    const planData = await callGemini(planPrompt);
    let plan;

    try {
      const cleaned = planData.replace(/```json|```/g, '').trim();
      plan = JSON.parse(cleaned);
    } catch(e) {
      plan = {
        titulo: taskText.substring(0, 40),
        complexidade: 'media',
        linguagens: ['HTML', 'CSS', 'JavaScript'],
        etapas: [
          {id:1, nome:'Estrutura base', descricao:'Criando estrutura inicial'},
          {id:2, nome:'Implementação', descricao:'Desenvolvendo funcionalidades'},
          {id:3, nome:'Finalização', descricao:'Ajustes e otimizações'}
        ],
        arquivos: ['index.html']
      };
    }

    updateStep(taskId, 1, 'done', '✓', 'Plano criado', `${plan.etapas.length} etapas · ${plan.linguagens.join(', ')}`);
    setProgress(taskId, 20);

    // Add planned steps
    const stepsEl = document.getElementById(`${taskId}-steps`);
    plan.etapas.forEach((etapa, i) => {
      addStepEl(stepsEl, taskId, etapa.id + 1, 'pending', String(etapa.id + 1), etapa.nome, etapa.descricao);
    });

    // PHASE 2: Execute each step
    const totalSteps = plan.etapas.length;
    let allCode = '';
    const stepResults = [];

    for (let i = 0; i < plan.etapas.length; i++) {
      const etapa = plan.etapas[i];
      const stepNum = etapa.id + 1;
      const progressVal = 20 + ((i + 1) / totalSteps) * 60;

      updateStep(taskId, stepNum, 'running', '⚡', etapa.nome, 'Executando...');

      const stepPrompt = `Você é um agente de programação autônomo especialista. Execute esta etapa específica.

TAREFA COMPLETA: ${taskText}
TIPO: ${taskType}
ETAPA ATUAL (${i+1}/${totalSteps}): ${etapa.nome}
DESCRIÇÃO: ${etapa.descricao}
CONTEXTO ANTERIOR: ${stepResults.slice(-1)[0] || 'Início do projeto'}
LINGUAGENS: ${plan.linguagens.join(', ')}

Gere o código completo e funcional para esta etapa. Seja detalhado, profissional e inclua comentários.
Se for a última etapa, garanta que tudo está integrado e funcionando.`;

      const stepCode = await callGemini(stepPrompt);
      stepResults.push(`Etapa ${i+1} (${etapa.nome}): concluída`);
      allCode += `\n\n/* ===== ETAPA ${i+1}: ${etapa.nome.toUpperCase()} ===== */\n` + stepCode;

      updateStep(taskId, stepNum, 'done', '✓', etapa.nome, 'Concluído ✓');
      setProgress(taskId, progressVal);

      await sleep(300);
    }

    // PHASE 3: Final integration
    const finalStepNum = plan.etapas.length + 2;
    addStepEl(stepsEl, taskId, finalStepNum, 'running', '⚡', 'Integração Final', 'Consolidando todo o código...');
    setProgress(taskId, 85);

    const finalPrompt = `Você é um agente de programação sênior. Integre e consolide tudo em código final, completo e funcional.

TAREFA: ${taskText}
TIPO: ${taskType}
TECNOLOGIAS: ${plan.linguagens.join(', ')}

Crie o código FINAL, COMPLETO e FUNCIONAL. 
- Se for um site: HTML completo com CSS e JS embutidos em um único arquivo
- Se for um script: código completo e executável
- Se for um bot: código completo com instruções de uso
- Se for automação: código completo com comentários detalhados

O código deve ser PRODUCTION-READY, bem comentado e imediatamente utilizável.
Escreva APENAS o código, sem explicações extras antes ou depois.`;

    const finalCode = await callGemini(finalPrompt);

    updateStep(taskId, finalStepNum, 'done', '✓', 'Integração Final', 'Código final gerado ✓');
    setProgress(taskId, 95);

    // Show output
    const ext = getExtension(plan.linguagens[0], taskType);
    const fileName = sanitizeFileName(plan.titulo || taskText) + ext;

    block.insertAdjacentHTML('beforeend', `
      <div class="output-block">
        <div class="output-header">
          <div class="output-label">◈ OUTPUT — ${fileName}</div>
          <button class="copy-btn" onclick="copyCode(this)">COPIAR</button>
        </div>
        <div class="output-content" id="${taskId}-output">${escapeHtml(finalCode)}</div>
      </div>
      <div class="files-section">
        <div class="files-title">◈ Arquivos Gerados</div>
        <div class="file-item">
          <div class="file-name">📄 ${fileName}</div>
          <button class="download-btn" onclick="downloadFile('${taskId}-output', '${fileName}')">↓ BAIXAR</button>
        </div>
      </div>
    `);

    setProgress(taskId, 100);
    document.getElementById(`${taskId}-status`).className = 'task-status done';
    document.getElementById(`${taskId}-status`).textContent = 'CONCLUÍDO';
    setStatus('active', 'ONLINE');

  } catch(err) {
    document.getElementById(`${taskId}-status`).className = 'task-status error';
    document.getElementById(`${taskId}-status`).textContent = 'ERRO';

    const stepsEl = document.getElementById(`${taskId}-steps`);
    stepsEl.insertAdjacentHTML('beforeend', `
      <div class="step" style="opacity:1">
        <div class="step-icon error">✕</div>
        <div class="step-content">
          <div class="step-title" style="color:var(--red)">Erro na execução</div>
          <div class="step-detail">${err.message}</div>
        </div>
      </div>
    `);

    setStatus('error', 'ERRO');
    setTimeout(() => setStatus('active', 'ONLINE'), 3000);
  }

  isRunning = false;
  document.getElementById('execute-btn').disabled = false;
  document.getElementById('agent-container').scrollTop = 999999;
}

// ===== HELPERS =====

async function callGemini(prompt, retries = 3) {
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const res = await fetch(
        `https://generativelanguage.googleapis.com/v1beta/models/${MODEL}:generateContent?key=${API_KEY}`,
        {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            contents: [{ role: 'user', parts: [{ text: prompt }] }],
            generationConfig: { temperature: 0.7, maxOutputTokens: 8192 }
          })
        }
      );

      if (res.status === 429) {
        if (attempt < retries) {
          await sleep(5000 * attempt);
          continue;
        }
        throw new Error('Limite de requisições atingido. Aguarde alguns segundos e tente novamente.');
      }

      if (!res.ok) {
        const d = await res.json();
        throw new Error(d?.error?.message || `Erro HTTP ${res.status}`);
      }

      const data = await res.json();
      return data?.candidates?.[0]?.content?.parts?.[0]?.text || '';

    } catch(e) {
      if (attempt === retries) throw e;
      await sleep(2000 * attempt);
    }
  }
}

function addStep(taskId, num, state, icon, title, detail = '') {
  const stepsEl = document.getElementById(`${taskId}-steps`);
  addStepEl(stepsEl, taskId, num, state, icon, title, detail);
  return Promise.resolve();
}

function addStepEl(container, taskId, num, state, icon, title, detail) {
  const existing = document.getElementById(`${taskId}-step-${num}`);
  if (existing) { updateStep(taskId, num, state, icon, title, detail); return; }
  const div = document.createElement('div');
  div.className = 'step';
  div.id = `${taskId}-step-${num}`;
  div.innerHTML = `
    <div class="step-icon ${state}" id="${taskId}-icon-${num}">${icon}</div>
    <div class="step-content">
      <div class="step-title ${state}" id="${taskId}-title-${num}">${title}</div>
      <div class="step-detail" id="${taskId}-detail-${num}">${detail}</div>
    </div>
  `;
  container.appendChild(div);
}

function updateStep(taskId, num, state, icon, title, detail = '') {
  const iconEl = document.getElementById(`${taskId}-icon-${num}`);
  const titleEl = document.getElementById(`${taskId}-title-${num}`);
  const detailEl = document.getElementById(`${taskId}-detail-${num}`);
  if (iconEl) { iconEl.className = `step-icon ${state}`; iconEl.textContent = icon; }
  if (titleEl) { titleEl.className = `step-title ${state}`; titleEl.textContent = title; }
  if (detailEl && detail) detailEl.textContent = detail;
}

function setProgress(taskId, pct) {
  const bar = document.getElementById(`${taskId}-progress`);
  if (bar) bar.style.width = pct + '%';
}

function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

function escapeHtml(text) {
  return text.replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
}

function getExtension(lang, type) {
  const l = (lang || '').toLowerCase();
  if (l.includes('python')) return '.py';
  if (l.includes('javascript') || l.includes('node')) return '.js';
  if (l.includes('typescript')) return '.ts';
  if (l.includes('css')) return '.css';
  if (type === 'script') return '.py';
  return '.html';
}

function sanitizeFileName(name) {
  return name.toLowerCase().replace(/[^a-z0-9]/g, '-').replace(/-+/g, '-').substring(0, 40);
}

function copyCode(btn) {
  const output = btn.closest('.output-block').querySelector('.output-content');
  navigator.clipboard.writeText(output.textContent).then(() => {
    btn.textContent = 'COPIADO ✓';
    setTimeout(() => btn.textContent = 'COPIAR', 2000);
  });
}

function downloadFile(outputId, fileName) {
  const content = document.getElementById(outputId).textContent;
  const blob = new Blob([content], { type: 'text/plain' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = fileName;
  a.click();
}
</script>
</body>
</html> 
