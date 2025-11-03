<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Ponto Eletrônico - Corporativo</title>
<style>
  :root{
    --blue:#0b4f78;
    --green:#2e9b4f;
    --yellow:#ffb739;
    --red:#ef5350;
    --muted:#6b7280;
    --card:#ffffff;
    --bg:#f4f7fb;
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
  .modal-content{background:#fff;padding:18px;border-radius:10px;width:95%;max-width:420px;box-shadow:0 10px 40px rgba(2,6,23,0.12)}
  .hidden{display:none}
  .flex-row{display:flex;gap:8px;align-items:center}
  @media(max-width:720px){ header{flex-direction:column;align-items:flex-start} .controls{width:100%;justify-content:space-between} table{font-size:13px} }
</style>

<!-- SheetJS for Excel export -->
<script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
</head>
<body>

<!-- LOGIN -->
<div id="loginScreen" style="position:fixed;inset:0;background:var(--blue);display:flex;align-items:center;justify-content:center;z-index:9999">
  <div style="background:#fff;padding:26px;border-radius:10px;width:92%;max-width:360px;text-align:center">
    <h2 style="margin:0 0 8px 0;color:var(--blue)">Login do Sistema</h2>
    <!-- campos visíveis para digitar (credenciais reais NÃO mostradas em texto) -->
    <input id="user" placeholder="Usuário" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="pass" type="password" placeholder="Senha" style="width:92%;padding:10px;margin:8px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <label style="font-size:13px"><input type="checkbox" id="remember"> Lembrar login</label><br>
    <button id="loginBtn" class="add" style="width:92%;margin-top:10px">Entrar</button>
    <p id="loginMsg" style="color:crimson;margin-top:8px;height:18px"></p>
    <!-- nota: removemos a dica de usuário/senha da interface -->
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
      <button class="download" id="baixarBtn">Baixar Planilhas</button>
      <button class="download" id="gerarRelatorioBtn">Relatório Horas</button>
      <button class="secondary" id="limparTodosBtn">Limpar Pontos</button>
      <button class="secondary" id="logoutBtn">Sair</button>
    </div>
  </div>
</header>

<main id="mainApp" class="hidden">
  <div style="display:flex;gap:12px;align-items:center;margin-bottom:12px;">
    <label>Filtrar mês:
      <input type="month" id="filtroMes">
    </label>
  </div>

  <input id="search" class="search" placeholder="🔍 Pesquisar colaborador por nome, cargo, matrícula ou e-mail">

  <div style="display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:8px">
    <h3 style="margin:0">Colaboradores</h3>
    <div style="display:flex;gap:8px">
      <button class="add" id="addColabBtn">Adicionar Colaborador</button>
    </div>
  </div>

  <table id="colabTable">
    <thead><tr><th>#</th><th>ID</th><th>Nome</th><th>Cargo</th><th>Matrícula / E-mail</th><th>Turno</th><th>Ações</th></tr></thead>
    <tbody id="colabBody"></tbody>
  </table>

  <h3>Entradas Registradas</h3>
  <table id="entradasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="entradasBody"></tbody>
  </table>

  <h3>Saídas Registradas</h3>
  <table id="saidasTable">
    <thead><tr><th>#</th><th>ID Colab</th><th>Nome</th><th>Data</th><th>Hora</th><th>Ações</th></tr></thead>
    <tbody id="saidasBody"></tbody>
  </table>

  <h3>Resumo de Horas Trabalhadas</h3>
  <table id="horasTable">
    <thead><tr><th>Funcionário</th><th>Data</th><th>Horas Trabalhadas</th></tr></thead>
    <tbody id="horasBody"></tbody>
    <tfoot><tr><td colspan="2"><b>Total Geral</b></td><td id="totalHoras">0</td></tr></tfoot>
  </table>
</main>

<div id="colabModal" class="modal hidden">
  <div class="modal-content">
    <h3 id="colabModalTitle">Adicionar Colaborador</h3>
    <input id="nomeInput" placeholder="Nome" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="cargoInput" placeholder="Cargo" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="matriculaInput" placeholder="Matrícula" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="emailInput" placeholder="E-mail" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <input id="turnoInput" placeholder="Turno" style="width:100%;padding:8px;margin:6px 0;border-radius:6px;border:1px solid #e5e7eb"><br>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:10px">
      <button class="secondary" id="cancelColab">Cancelar</button>
      <button class="add" id="saveColab">Salvar</button>
    </div>
  </div>
</div>

<script type="module">
/* -------- FIREBASE -------- */
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import { getFirestore, collection, getDocs, setDoc, doc, deleteDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyCpBiFzqOod4K32cWMr5hfx13fw6LGcPVY",
  authDomain: "ponto-eletronico-f35f9.firebaseapp.com",
  projectId: "ponto-eletronico-f35f9",
  storageBucket: "ponto-eletronico-f35f9.firebasestorage.app",
  messagingSenderId: "208638350255",
  appId: "1:208638350255:web:63d016867a67575b5e155a"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

/* -------- estado -------- */
let colaboradores = [];
let pontos = [];
let colabEmEdicao = null;

/* elementos */
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const filtroMesInput = document.getElementById('filtroMes');

/* -------- LOGIN -------- */
/* credenciais fixas (mantidas do seu código original) */
const LOGIN_USER = 'CLX';
const LOGIN_PASS = '02072007';

document.getElementById('loginBtn').onclick = async () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  if (u === LOGIN_USER && p === LOGIN_PASS) {
    loginScreen.style.display = 'none';
    mainApp.classList.remove('hidden');
    if (document.getElementById('remember').checked) localStorage.setItem('autenticado','1');
    iniciarLeituras();
  } else {
    document.getElementById('loginMsg').textContent = 'Usuário ou senha incorretos.';
  }
};

if (localStorage.getItem('autenticado') === '1') {
  loginScreen.style.display = 'none';
  mainApp.classList.remove('hidden');
  iniciarLeituras();
}

document.getElementById('logoutBtn').onclick = () => {
  localStorage.removeItem('autenticado');
  location.reload();
};

/* -------- RELÓGIO -------- */
setInterval(() => {
  document.getElementById('clock').textContent = new Date().toLocaleTimeString('pt-BR', { hour12: false });
}, 1000);

/* -------- INICIAR LEITURAS (Firestore) -------- */
async function iniciarLeituras(){
  document.getElementById('status').textContent = "Carregando...";
  const colSnap = await getDocs(collection(db, "colaboradores"));
  colaboradores = colSnap.docs.map(d => ({ id: d.id, ...d.data() }));
  const ptSnap = await getDocs(collection(db, "pontos"));
  pontos = ptSnap.docs.map(d => ({ id: d.id, ...d.data() }));
  renderAll();

  onSnapshot(collection(db, "colaboradores"), snap => {
    colaboradores = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    renderColaboradores(document.getElementById('search').value.toLowerCase());
    document.getElementById('status').textContent = "Online • Firebase";
  });

  onSnapshot(collection(db, "pontos"), snap => {
    pontos = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    renderEntradasSaidas();
    calcularHoras();
    document.getElementById('status').textContent = "Online • Firebase";
  });
}

/* -------- RENDER GERAL -------- */
function renderAll(){
  renderColaboradores();
  renderEntradasSaidas();
  calcularHoras();
}

document.getElementById('search').addEventListener('input', () => {
  renderColaboradores(document.getElementById('search').value.toLowerCase());
});

/* -------- RENDER COLABORADORES -------- */
function renderColaboradores(filtro = '') {
  const body = document.getElementById('colabBody');
  if (!body) return;
  body.innerHTML = '';

  colaboradores
    .filter(c =>
      (c.nome || '').toLowerCase().includes(filtro) ||
      (c.cargo || '').toLowerCase().includes(filtro) ||
      (c.matricula || '').toLowerCase().includes(filtro) ||
      (c.email || '').toLowerCase().includes(filtro)
    )
    .forEach((c, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${i + 1}</td>
        <td>${c.id}</td>
        <td>${c.nome || ''}</td>
        <td>${c.cargo || ''}</td>
        <td>${c.matricula || ''} <span class="small">${c.email || ''}</span></td>
        <td>${c.turno || ''}</td>
        <td>
          <button class="add btnEntrada">Entrada</button>
          <button class="secondary btnSaida">Saída</button>
          <button class="secondary editBtn">Editar</button>
          <button class="danger delBtn">Excluir</button>
        </td>`;
      tr.querySelector('.btnEntrada').onclick = () => registrarPonto(c.id, 'Entrada');
      tr.querySelector('.btnSaida').onclick = () => registrarPonto(c.id, 'Saída');
      tr.querySelector('.editBtn').onclick = () => abrirModalEditar(c);
      tr.querySelector('.delBtn').onclick = () => removerColab(c.id);
      body.appendChild(tr);
    });
}

/* -------- MODAL COLABORADOR -------- */
const colabModal = document.getElementById('colabModal');
const colabModalTitle = document.getElementById('colabModalTitle');
const nomeInput = document.getElementById('nomeInput');
const cargoInput = document.getElementById('cargoInput');
const matriculaInput = document.getElementById('matriculaInput');
const emailInput = document.getElementById('emailInput');
const turnoInput = document.getElementById('turnoInput');

document.getElementById('addColabBtn').onclick = () => abrirModalAdicionar();
document.getElementById('cancelColab').onclick = () => fecharModalColab();

function abrirModalAdicionar(){
  colabEmEdicao = null;
  colabModalTitle.textContent = 'Adicionar Colaborador';
  nomeInput.value = cargoInput.value = matriculaInput.value = emailInput.value = turnoInput.value = '';
  colabModal.classList.remove('hidden');
}
function abrirModalEditar(c){
  colabEmEdicao = c;
  colabModalTitle.textContent = 'Editar Colaborador';
  nomeInput.value = c.nome || '';
  cargoInput.value = c.cargo || '';
  matriculaInput.value = c.matricula || '';
  emailInput.value = c.email || '';
  turnoInput.value = c.turno || '';
  colabModal.classList.remove('hidden');
}
function fecharModalColab(){
  colabModal.classList.add('hidden');
}

document.getElementById('saveColab').onclick = async () => {
  const nome = nomeInput.value.trim();
  if (!nome) return alert('Informe o nome do colaborador');
  const obj = {
    nome,
    cargo: cargoInput.value.trim(),
    matricula: matriculaInput.value.trim(),
    email: emailInput.value.trim(),
    turno: turnoInput.value.trim()
  };
  if (colabEmEdicao && colabEmEdicao.id) {
    await setDoc(doc(db, "colaboradores", colabEmEdicao.id), { ...colabEmEdicao, ...obj });
  } else {
    const newId = Date.now().toString();
    await setDoc(doc(db, "colaboradores", newId), { id: newId, ...obj });
  }
  fecharModalColab();
};

/* -------- REGISTRAR PONTO -------- */
async function registrarPonto(idColab, tipo) {
  const c = colaboradores.find(x => x.id === idColab);
  if (!c) return alert("Colaborador não encontrado!");
  const now = new Date();
  const p = {
    id: Date.now().toString(),
    idColab,
    nome: c.nome,
    matricula: c.matricula,
    email: c.email,
    tipo,
    data: now.toLocaleDateString('pt-BR'),
    hora: now.toLocaleTimeString('pt-BR', { hour12: false }),
    horarioISO: now.toISOString()
  };
  pontos.push(p);
  renderEntradasSaidas();
  await setDoc(doc(db, "pontos", p.id), p);
}

/* -------- RENDER ENTRADAS / SAÍDAS (COM FILTRO MÊS) -------- */
function renderEntradasSaidas() {
  const entBody = document.getElementById('entradasBody');
  const saiBody = document.getElementById('saidasBody');
  entBody.innerHTML = '';
  saiBody.innerHTML = '';

  const filtroMes = filtroMesInput.value; // ex: "2025-11" or ""
  const pontosFiltrados = pontos.filter(p => {
    if (!filtroMes) return true;
    const [anoFiltro, mesFiltro] = filtroMes.split('-');
    const [d, m, a] = p.data.split('/');
    return a === anoFiltro && m === mesFiltro;
  });

  pontosFiltrados.filter(p => p.tipo === 'Entrada').forEach((p, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${i+1}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
    tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
    entBody.appendChild(tr);
  });

  pontosFiltrados.filter(p => p.tipo === 'Saída').forEach((p, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${i+1}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
    tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
    saiBody.appendChild(tr);
  });

  calcularHoras(); // recalcula com base no conjunto inteiro (calcularHoras usa pontos globais; mantemos)
}

/* -------- EXCLUIR PONTO -------- */
async function excluirPonto(id) {
  if (confirm("Excluir este ponto permanentemente?")) {
    pontos = pontos.filter(p => p.id !== id);
    renderEntradasSaidas();
    await deleteDoc(doc(db, "pontos", id));
  }
}

/* -------- REMOVER COLABORADOR (e seus pontos) -------- */
async function removerColab(id) {
  if (confirm("Excluir colaborador permanentemente?")) {
    colaboradores = colaboradores.filter(c => c.id !== id);
    pontos = pontos.filter(p => p.idColab !== id);
    renderAll();
    await deleteDoc(doc(db, "colaboradores", id));
    const pts = await getDocs(collection(db, "pontos"));
    for (let d of pts.docs) if (d.data().idColab === id) await deleteDoc(doc(db, "pontos", d.id));
  }
}

/* -------- LIMPAR TODOS OS PONTOS -------- */
document.getElementById('limparTodosBtn').onclick = async () => {
  if (confirm("Deseja realmente excluir todos os pontos?")) {
    pontos = [];
    renderEntradasSaidas();
    const col = await getDocs(collection(db, "pontos"));
    for (let docSnap of col.docs) await deleteDoc(doc(db, "pontos", docSnap.id));
  }
};

/* -------- CALCULAR HORAS (por dia; também calcula total geral) -------- */
function calcularHoras() {
  const horasBody = document.getElementById('horasBody');
  const totalHorasCell = document.getElementById('totalHoras');
  horasBody.innerHTML = '';
  let dados = {}, totalGeral = 0;

  // organizando por nome e data
  pontos.forEach(p => {
    if (!dados[p.nome]) dados[p.nome] = {};
    if (!dados[p.nome][p.data]) dados[p.nome][p.data] = [];
    dados[p.nome][p.data].push(p);
  });

  Object.keys(dados).forEach(nome => {
    Object.keys(dados[nome]).forEach(data => {
      let reg = dados[nome][data].sort((a,b) => new Date(a.horarioISO) - new Date(b.horarioISO));
      let entrada = null, total = 0;
      reg.forEach(r => {
        const hora = new Date(r.horarioISO);
        if (r.tipo === 'Entrada') entrada = hora;
        if (r.tipo === 'Saída' && entrada) {
          total += (hora - entrada) / 3600000;
          entrada = null;
        }
      });
      totalGeral += total;
      let h = Math.floor(total);
      let m = Math.round((total - h)*60);
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${nome}</td><td>${data}</td><td>${h}h ${m}m</td>`;
      horasBody.appendChild(tr);
    });
  });

  let gh = Math.floor(totalGeral);
  let gm = Math.round((totalGeral - gh)*60);
  totalHorasCell.textContent = `${gh}h ${gm}m`;
}

/* -------- EXPORTAR EXCEL (com filtro por mês) -------- */
document.getElementById('baixarBtn').onclick = () => {
  const filtroMes = filtroMesInput.value; // ex: "2025-11"
  const entradas = [['#','ID Colab','Nome','Data','Hora']];
  const saidas = [['#','ID Colab','Nome','Data','Hora']];

  // usar índice separado para entradas/saídas para numeração correta
  let eIndex = 1, sIndex = 1;

  pontos.filter(p => {
    if (!filtroMes) return true;
    const [ano, mes] = filtroMes.split('-');
    const [d, m, a] = p.data.split('/');
    return a === ano && m === mes;
  }).forEach(p => {
    if (p.tipo === 'Entrada') { entradas.push([eIndex++, p.idColab, p.nome, p.data, p.hora]); }
    if (p.tipo === 'Saída') { saidas.push([sIndex++, p.idColab, p.nome, p.data, p.hora]); }
  });

  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(entradas), 'Entradas');
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(saidas), 'Saídas');
  XLSX.writeFile(wb, 'Pontos_Filtrados.xlsx');
};

/* -------- GERAR RELATÓRIO SEMANAL / MENSAL E EXPORTAR -------- */
document.getElementById('gerarRelatorioBtn').onclick = () => {
  const filtroMes = filtroMesInput.value; // ex: "2025-11" or ''
  // agruparemos por nome, por semana (início da semana: segunda-feira) e total mensal
  const relatorio = {}; // { nome: { semanal: {semanaInicioStr: hours}, mensal: hours } }

  // função pra obter data da segunda da semana da data passada
  function getMonday(d) {
    const date = new Date(d);
    const day = date.getDay();
    const diff = date.getDate() - day + (day === 0 ? -6 : 1);
    const monday = new Date(date.setDate(diff));
    monday.setHours(0,0,0,0);
    return monday;
  }

  // preparar cópia dos pontos ordenados por horarioISO por pessoa e data
  let pontosFiltrados = pontos.slice().sort((a,b)=> new Date(a.horarioISO) - new Date(b.horarioISO));

  pontosFiltrados = pontosFiltrados.filter(p => {
    if (!filtroMes) return true;
    const [ano, mes] = filtroMes.split('-');
    const [d, m, a] = p.data.split('/');
    return a === ano && m === mes;
  });

  // vamos agrupar entradas/saídas por pessoa e por dia para calcular pares entrada->saída
  // estratégia: para cada pessoa e dia, percorrer registros ordenados e parear entrada+saída
  const byPerson = {};
  pontosFiltrados.forEach(p => {
    if (!byPerson[p.nome]) byPerson[p.nome] = [];
    byPerson[p.nome].push(p);
  });

  Object.keys(byPerson).forEach(nome => {
    relatorio[nome] = { semanal: {}, mensal: 0 };
    const registros = byPerson[nome].slice().sort((a,b)=> new Date(a.horarioISO) - new Date(b.horarioISO));
    let entrada = null;
    registros.forEach(r => {
      const dt = new Date(r.horarioISO);
      if (r.tipo === 'Entrada') {
        entrada = dt;
      } else if (r.tipo === 'Saída' && entrada) {
        const hours = (dt - entrada) / 3600000;
        const monday = getMonday(entrada).toLocaleDateString('pt-BR');
        relatorio[nome].semanal[monday] = (relatorio[nome].semanal[monday] || 0) + hours;
        relatorio[nome].mensal += hours;
        entrada = null;
      }
    });
  });

  // montar planilha (AOA)
  const wsData = [['Funcionário','Semana (segunda)','Horas Semana','Total Mensal']];
  Object.keys(relatorio).forEach(nome => {
    const semanas = relatorio[nome].semanal;
    const semanasKeys = Object.keys(semanas).sort((a,b)=> {
      const pa = a.split('/').reverse().join('-');
      const pb = b.split('/').reverse().join('-');
      return new Date(pa) - new Date(pb);
    });
    if (semanasKeys.length === 0) {
      wsData.push([nome, '-', '-', `${Math.floor(relatorio[nome].mensal)}h ${Math.round((relatorio[nome].mensal - Math.floor(relatorio[nome].mensal))*60)}m`]);
    } else {
      semanasKeys.forEach(sk => {
        const h = semanas[sk];
        wsData.push([nome, sk, `${Math.floor(h)}h ${Math.round((h - Math.floor(h))*60)}m`, '']);
      });
      // linha resumo mensal
      wsData.push([nome, 'Total Mês', '', `${Math.floor(relatorio[nome].mensal)}h ${Math.round((relatorio[nome].mensal - Math.floor(relatorio[nome].mensal))*60)}m`]);
    }
  });

  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(wsData), 'Relatorio_Horas');
  XLSX.writeFile(wb, 'Relatorio_Horas.xlsx');
};

/* -------- PONTO DE SAÍDA AUTOMÁTICO 20:00 -------- */
function baterSaidaAutomatica() {
  const agora = new Date();
  const horaAtual = agora.getHours();
  const minutoAtual = agora.getMinutes();
  if (horaAtual === 20 && minutoAtual === 0) {
    colaboradores.forEach(c => {
      const pontosHoje = pontos.filter(p => p.idColab === c.id && p.data === agora.toLocaleDateString('pt-BR'));
      const temEntrada = pontosHoje.some(p => p.tipo === 'Entrada');
      const temSaida = pontosHoje.some(p => p.tipo === 'Saída');
      if (temEntrada && !temSaida) registrarPonto(c.id, 'Saída');
    });
  }
}
setInterval(baterSaidaAutomatica, 60000);

/* -------- Observadores de UI -------- */
filtroMesInput.addEventListener('change', () => {
  renderEntradasSaidas();
});
</script>
</body>
</html>
