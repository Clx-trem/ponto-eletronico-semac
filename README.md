<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Ponto Eletrônico - IDs contínuos</title>
<style>
  :root{
    --blue:#0b4f78; --green:#2e9b4f; --yellow:#ffb739; --red:#ef5350;
    --muted:#6b7280; --card:#ffffff; --bg:#f4f7fb;
  }
  body{font-family:Inter, system-ui, -apple-system, Arial, sans-serif;background:var(--bg);margin:0;color:#111}
  header{background:linear-gradient(90deg,var(--blue),#0f6b96);color:#fff;padding:12px 18px;display:flex;align-items:center;justify-content:space-between;gap:12px;flex-wrap:wrap}
  .logo{font-weight:700;font-size:18px}
  #clock{font-weight:700}
  .controls{display:flex;gap:8px;align-items:center}
  button{padding:8px 12px;border:none;border-radius:8px;cursor:pointer;font-weight:600}
  .add{background:var(--green);color:#fff}
  .secondary{background:#e5e7eb;color:#111}
  .download{background:var(--yellow);color:#111}
  .danger{background:var(--red);color:#fff}
  main{padding:20px;max-width:1100px;margin:20px auto}
  .search{width:100%;padding:10px;border-radius:8px;border:1px solid #d1d5db;margin-bottom:14px}
  table{width:100%;border-collapse:collapse;background:var(--card);border-radius:10px;overflow:hidden;box-shadow:0 6px 24px rgba(15,23,42,0.06);margin-bottom:18px}
  th,td{padding:10px;border-bottom:1px solid #eef2f6;text-align:left;font-size:14px}
  th{background:#fbfdfe;font-weight:700}
  tr:hover td{background:#fcfdff}
  .small{font-size:13px;color:var(--muted);margin-left:6px}
  .muted{color:var(--muted);font-size:13px}
  .modal{position:fixed;inset:0;background:rgba(0,0,0,.45);display:flex;align-items:center;justify-content:center;z-index:999}
  .modal-content{background:#fff;padding:18px;border-radius:10px;width:95%;max-width:720px;box-shadow:0 10px 40px rgba(2,6,23,0.12)}
  .hidden{display:none}
  .flex-row{display:flex;gap:8px;align-items:center}
  @media(max-width:720px){ header{flex-direction:column;align-items:flex-start} .controls{width:100%;justify-content:space-between} table{font-size:13px} }
</style>

<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" style="position:fixed;inset:0;background:var(--blue);display:flex;align-items:center;justify-content:center;z-index:9999">
  <div style="background:#fff;padding:26px;border-radius:10px;width:92%;max-width:360px;text-align:center">
    <h2 style="margin:0 0 8px 0;color:var(--blue)">Login do Sistema</h2>
    <input id="user" placeholder="Usuário" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="pass" type="password" placeholder="Senha" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <label style="font-size:13px"><input type="checkbox" id="remember"> Lembrar login</label><br>
    <button id="loginBtn" class="add" style="width:92%;margin-top:10px">Entrar</button>
    <p id="loginMsg" style="color:crimson;margin-top:8px;height:18px"></p>
  </div>
</div>

<header>
  <div style="display:flex;gap:12px;align-items:center">
    <div class="logo">Ponto Eletrônico</div>
    <div id="status" class="muted">Offline • Local Storage</div>
  </div>
  <div style="display:flex;gap:12px;align-items:center">
    <div id="clock">--:--:--</div>
    <div class="controls">
      <button class="download" id="baixarBtn">Baixar Planilhas (mês atual)</button>
      <button class="download" id="gerarRelatorioBtn">Relatório Horas (mês atual)</button>
      <button class="secondary" id="limparTodosPontosBtn">Limpar Pontos</button>
      <button class="secondary" id="limparTodosColabsBtn">Apagar Todos Colaboradores</button>
      <button class="secondary" id="logoutBtn">Sair</button>
    </div>
  </div>
</header>

<main id="mainApp" class="hidden">
  <div style="display:flex;gap:12px;align-items:center;margin-bottom:12px;">
    <label>Colaborador:
      <select id="colabSelect" style="padding:8px;border-radius:6px;border:1px solid #d1d5db"></select>
    </label>
    <button class="secondary" id="verRelatorioColabBtn">Ver Relatório Colaborador</button>
    <button class="download" id="exportRelatorioColabBtn">Exportar Relatório Colaborador</button>
  </div>

  <input id="search" class="search" placeholder="🔍 Pesquisar colaborador por nome, cargo, matrícula ou e-mail">

  <div style="display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:8px">
    <h3 style="margin:0">Colaboradores</h3>
    <div style="display:flex;gap:8px">
      <button class="add" id="addColabBtn">Adicionar Colaborador</button>
    </div>
  </div>

  <h3>Entradas Registradas (mês atual)</h3>
  <table id="entradasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="entradasBody"></tbody>
  </table>

  <h3>Saídas Registradas (mês atual)</h3>
  <table id="saidasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="saidasBody"></tbody>
  </table>

  <h3>Resumo de Horas Trabalhadas (mês atual)</h3>
  <table id="horasTable">
    <thead><tr><th>Funcionário</th><th>Data</th><th>Horas Trabalhadas</th></tr></thead>
    <tbody id="horasBody"></tbody>
    <tfoot><tr><td colspan="2"><b>Total Geral</b></td><td id="totalHoras">0h 0m 0s</td></tr></tfoot>
  </table>
</main>

<!-- MODAIS E SCRIPT COMPLETO -->
<script type="module">
// --- IMPORT FIREBASE ---
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";
const firebaseConfig = {
  apiKey: "AIzaSyCpBiFzqOod4K32cWMr5hfx13fw6LGcPVY",
  authDomain: "ponto-eletronico-f35f9.firebaseapp.com",
  projectId: "ponto-eletronico-f35f9",
  storageBucket: "ponto-eletronico-f35f9.firebasestorage.app",
  messagingSenderId: "208638350255",
  appId: "1:208638350255:web:63d016867a67575b5b0e0f"
};
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

// --- DADOS LOCAIS ---
let colaboradores = [];
let pontos = [];
let logins = [];
let userLogado = null;

// --- FUNÇÕES AUXILIARES ---
function agora() {
  const d = new Date();
  const data = d.toLocaleDateString('pt-BR');
  const hora = d.toLocaleTimeString('pt-BR'); // mantém segundos
  const iso = d.toISOString();
  return { data, hora, iso };
}

function pontosDoMesAtual(lista) {
  const hoje = new Date();
  return lista.filter(p=>{
    const [d,m,a] = p.data.split('/').map(Number);
    return m === (hoje.getMonth()+1) && a === hoje.getFullYear();
  });
}

// --- RENDERIZAR PONTOS AGRUPADOS POR DIA ---
function renderEntradasSaidas() {
  const entBody = document.getElementById('entradasBody');
  const saiBody = document.getElementById('saidasBody');
  entBody.innerHTML = ''; 
  saiBody.innerHTML = '';

  const pts = pontosDoMesAtual(pontos);

  const entradasPorDia = {};
  const saidasPorDia = {};

  pts.forEach(p => {
    if(p.tipo === 'Entrada'){
      if(!entradasPorDia[p.data]) entradasPorDia[p.data] = [];
      entradasPorDia[p.data].push(p);
    } else if(p.tipo === 'Saída'){
      if(!saidasPorDia[p.data]) saidasPorDia[p.data] = [];
      saidasPorDia[p.data].push(p);
    }
  });

  const datasEntradas = Object.keys(entradasPorDia).sort((a,b)=>{
    const [da,ma,aa] = a.split('/').map(Number);
    const [db,mb,ab] = b.split('/').map(Number);
    return new Date(aa,ma-1,da) - new Date(ab,mb-1,db);
  });
  const datasSaidas = Object.keys(saidasPorDia).sort((a,b)=>{
    const [da,ma,aa] = a.split('/').map(Number);
    const [db,mb,ab] = b.split('/').map(Number);
    return new Date(aa,ma-1,da) - new Date(ab,mb-1,db);
  });

  let eIdx = 1;
  datasEntradas.forEach(d => {
    const ptsDia = entradasPorDia[d].sort((a,b)=> new Date(a.horarioISO) - new Date(b.horarioISO));
    const trDia = document.createElement('tr');
    trDia.innerHTML = `<td colspan="6" style="background:#f0f0f0;font-weight:bold">📅 ${d}</td>`;
    entBody.appendChild(trDia);
    ptsDia.forEach(p=>{
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${eIdx++}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
      tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
      entBody.appendChild(tr);
    });
  });

  let sIdx = 1;
  datasSaidas.forEach(d => {
    const ptsDia = saidasPorDia[d].sort((a,b)=> new Date(a.horarioISO) - new Date(b.horarioISO));
    const trDia = document.createElement('tr');
    trDia.innerHTML = `<td colspan="6" style="background:#f0f0f0;font-weight:bold">📅 ${d}</td>`;
    saiBody.appendChild(trDia);
    ptsDia.forEach(p=>{
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${sIdx++}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
      tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
      saiBody.appendChild(tr);
    });
  });

  calcularHoras();
}

// --- EXCLUIR PONTO ---
function excluirPonto(id){
  pontos = pontos.filter(p=>p.id!==id);
  renderEntradasSaidas();
}

// --- CALCULAR HORAS ---
function calcularHoras(){
  // Aqui você coloca sua lógica de horas
}

// --- RELÓGIO ---
function updateClock(){
  const c = document.getElementById('clock');
  const d = new Date();
  c.textContent = d.toLocaleTimeString('pt-BR');
}
setInterval(updateClock,1000);

// --- LOGIN SIMPLIFICADO ---
document.getElementById('loginBtn').onclick = () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  const msg = document.getElementById('loginMsg');
  const encontrado = logins.find(l=>l.usuario===u && l.senha===p);
  if(encontrado){
    userLogado = encontrado;
    document.getElementById('loginScreen').classList.add('hidden');
    document.getElementById('mainApp').classList.remove('hidden');
    renderEntradasSaidas(); // chama para mostrar Entradas e Saídas agrupadas
    msg.textContent = '';
  } else msg.textContent = 'Usuário ou senha inválidos';
};

</script>

</body>
</html>
