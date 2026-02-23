<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>IA-Prioryti PRO</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f9f9f9;
      margin: 0;
      padding: 0;
      color: #333;
    }

    header {
      background: #007bff;
      color: white;
      text-align: center;
      padding: 1rem;
    }

    section {
      background: white;
      max-width: 600px;
      margin: 1rem auto;
      padding: 1rem;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }

    textarea, input {
      width: 100%;
      padding: 0.5rem;
      margin-bottom: 0.5rem;
      border-radius: 4px;
      border: 1px solid #ccc;
    }

    button {
      padding: 0.5rem 1rem;
      margin: 0.25rem 0;
      background: #007bff;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }

    button:hover {
      background: #0056b3;
    }

    ul {
      padding-left: 1rem;
    }
  </style>
</head>
<body>
  <header>
    <h1>IA-Prioryti</h1>
    <p>Clasifica tus documentos por prioridad y gestiona tus planes</p>
  </header>

  <section class="user">
    <h2>Bienvenido</h2>
    <input type="text" id="username" placeholder="Ingresa tu nombre">
    <button id="login-btn">Iniciar sesión</button>
    <p id="welcome"></p>
  </section>

  <section class="plans">
    <h2>Planes</h2>
    <p>Prueba PRO por 7 días</p>
    <button id="free-plan-btn">Usar plan FREE</button>
    <button id="pro-plan-btn">Activar PRO</button>
    <input type="text" id="pro-code" placeholder="Ingresa tu código PRO">
    <p id="plan-info"></p>
    <p id="pro-countdown"></p>
  </section>

  <section class="classifier">
    <h2>Clasificador de documentos</h2>
    <textarea id="doc-input" placeholder="Pega aquí tu documento"></textarea>
    <button id="classify-btn">Clasificar</button>
    <p id="result"></p>
    <h3>Historial</h3>
    <ul id="history"></ul>
  </section>

  <script>
    // --- Variables ---
    let plan = 'FREE';
    const PRO_CODE = 'PRIORITY123';
    let proExpiry = null;

    // --- Usuarios ---
    const loginBtn = document.getElementById('login-btn');
    const usernameInput = document.getElementById('username');
    const welcomeText = document.getElementById('welcome');

    loginBtn.addEventListener('click', () => {
      const username = usernameInput.value.trim();
      if(!username) return alert('Ingresa tu nombre');
      localStorage.setItem('username', username);
      welcomeText.textContent = `Hola, ${username}!`;
      loadHistory();
      loadPlan();
    });

    // --- Planes ---
    const freeBtn = document.getElementById('free-plan-btn');
    const proBtn = document.getElementById('pro-plan-btn');
    const proCodeInput = document.getElementById('pro-code');
    const planInfo = document.getElementById('plan-info');
    const proCountdown = document.getElementById('pro-countdown');

    function loadPlan(){
      const storedPlan = localStorage.getItem('plan');
      if(storedPlan) plan = storedPlan;
      const expiry = localStorage.getItem('proExpiry');
      if(expiry) proExpiry = new Date(expiry);
      updatePlanUI();
    }

    function updatePlanUI(){
      planInfo.textContent = `Plan actual: ${plan}`;
      if(plan === 'PRO' && proExpiry){
        const now = new Date();
        const diff = proExpiry - now;
        if(diff > 0){
          const days = Math.floor(diff / (1000*60*60*24));
          const hours = Math.floor((diff % (1000*60*60*24))/(1000*60*60));
          const mins = Math.floor((diff % (1000*60*60))/(1000*60));
          proCountdown.textContent = `Prueba PRO activa: ${days}d ${hours}h ${mins}m restantes`;
        } else {
          plan = 'FREE';
          localStorage.setItem('plan', plan);
          proCountdown.textContent = '';
          planInfo.textContent = `Plan actual: ${plan}`;
        }
      }
    }

    freeBtn.addEventListener('click', () => {
      plan = 'FREE';
      localStorage.setItem('plan', plan);
      updatePlanUI();
      alert('Estás usando el plan FREE');
    });

    proBtn.addEventListener('click', () => {
      if(proCodeInput.value === PRO_CODE){
        plan = 'PRO';
        const now = new Date();
        proExpiry = new Date(now.getTime() + 7*24*60*60*1000); // 7 días
        localStorage.setItem('plan', plan);
        localStorage.setItem('proExpiry', proExpiry);
        updatePlanUI();
        alert('PRO activado por 7 días!');
      } else {
        alert('Código PRO incorrecto');
      }
    });

    // --- Clasificador ---
    const classifyBtn = document.getElementById('classify-btn');
    const docInput = document.getElementById('doc-input');
    const result = document.getElementById('result');
    const historyList = document.getElementById('history');

    function loadHistory(){
      const hist = JSON.parse(localStorage.getItem('history') || '[]');
      historyList.innerHTML = '';
      hist.forEach(h => {
        const li = document.createElement('li');
        li.textContent = `[${h.date}] "${h.text}" => ${h.priority}`;
        historyList.appendChild(li);
      });
    }

    classifyBtn.addEventListener('click', () => {
      const text = docInput.value.trim();
      if(!text) return alert('Ingresa un documento');
      let priority = 'Media';
      const t = text.toLowerCase();
      if(t.includes('urgente') || t.includes('importante')) priority = 'Alta';
      if(t.includes('opcional') || t.includes('bajo')) priority = 'Baja';
      const entry = {text, priority, date: new Date().toLocaleString()};
      const hist = JSON.parse(localStorage.getItem('history') || '[]');
      hist.unshift(entry);
      localStorage.setItem('history', JSON.stringify(hist));
      result.textContent = `Prioridad: ${priority}`;
      loadHistory();
    });

    // --- Al cargar la página ---
    window.addEventListener('load', () => {
      const username = localStorage.getItem('username');
      if(username){
        welcomeText.textContent = `Hola, ${username}!`;
      }
      loadPlan();
      loadHistory();
    });
  </script>
</body>
</html>
