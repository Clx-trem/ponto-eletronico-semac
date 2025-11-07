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
  /* pequenas melhorias para o modal de usuários */
  #usuariosList { max-height:220px; overflow:auto; margin-bottom:10px; border:1px solid #eef2f6; border-radius:6px; padding:8px; background:#fafafa; }
  .usr-row{display:flex;justify-content:space-between;align-items:center;padding:6px 8px;border-bottom:1px solid #f1f5f9}
  /* ícone de engrenagem estilo simples */
  .gear { width:36px;height:36px;border-radius:8px;display:flex;align-items:center;justify-content:center;cursor:pointer;background:transparent;border:none;color:#fff;font-size:18px }
  /* acessos list */
  #acessosList { max-height:300px; overflow:auto; border:1px solid #eef2f6; border-radius:6px; padding:8px; background:#fff; }
  .acc-row{padding:8px;border-bottom:1px solid #f1f5f9;font-size:13px}
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
      <!-- botão novo: gerenciar acessos (apenas admin verá) com ícone de engrenagem -->
      <button class="gear secondary" id="gerenciarAcessosBtn" title="Gerenciar Logins" style="display:none">⚙️</button>
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

  <table id="colabTable">
    <thead><tr><th>#</th><th>ID</th><th>Nome</th><th>Cargo</th><th>Matrícula / E-mail</th><th>Turno</th><th>Ações</th></tr></thead>
    <tbody id="colabBody"></tbody>
  </table>

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

<!-- Modal Colaborador (mantido) -->
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

<!-- Modal Relatório por Colaborador (mantido) -->
<div id="relColabModal" class="modal hidden">
  <div class="modal-content" style="max-width:900px">
    <h3>Relatório por Colaborador (mês atual)</h3>
    <div id="relColabContent"></div>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button class="secondary" id="closeRelColab">Fechar</button>
    </div>
  </div>
</div>

<!-- Modal Usuarios (novo) -->
<div id="usuariosModal" class="modal hidden">
  <div class="modal-content" style="max-width:720px">
    <h3>Gerenciar Logins</h3>

    <div id="usuariosList"></div>

    <div style="display:flex;gap:8px;align-items:center;margin-top:8px">
      <input id="novoUsuario" placeholder="Usuário" style="padding:8px;border-radius:6px;border:1px solid #e5e7eb">
      <!-- senha com asteriscos -->
      <input id="novaSenha" placeholder="Senha" type="password" style="padding:8px;border-radius:6px;border:1px solid #e5e7eb">
      <button class="add" id="addUsuarioBtn">Adicionar</button>
      <button class="secondary" id="verAcessosBtn" style="margin-left:8px">Ver Acessos</button>
    </div>

    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button class="secondary" id="closeUsuarios">Fechar</button>
      <button class="danger" id="limparAcessosBtn">Limpar Logs Acessos</button>
    </div>
  </div>
</div>

<!-- Modal Acessos -->
<div id="acessosModal" class="modal hidden">
  <div class="modal-content" style="max-width:900px">
    <h3>Logs de Acessos</h3>
    <div id="acessosList">Carregando...</div>
    <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:12px">
      <button class="secondary" id="closeAcessos">Fechar</button>
    </div>
  </div>
</div>

<script type="module">
/* ---------- FIREBASE ---------- */
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
import {
  getFirestore, collection, getDocs, setDoc, doc, deleteDoc, onSnapshot, runTransaction, addDoc
} from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

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

/* ---------- Estado ---------- */
let colaboradores = [];
let pontos = [];
let colabEmEdicao = null;
let usuariosAcesso = []; // lista de logins cadastrados
let usuarioLogado = null;
let isAdmin = false;

/* UI elements */
const loginScreen = document.getElementById('loginScreen');
const mainApp = document.getElementById('mainApp');
const colabSelect = document.getElementById('colabSelect');
const filtroAtual = (() => { const d = new Date(); const m = String(d.getMonth()+1).padStart(2,'0'); return `${d.getFullYear()}-${m}`; })(); // "YYYY-MM"

/* credenciais fixas (admin principal) */
const LOGIN_USER = 'CLX';
const LOGIN_PASS = '02072007';

/* ---------- LOGIN (agora valida também os usuarios da coleção 'logins') ---------- */
async function buscarUsuariosAcesso() {
  const uSnap = await getDocs(collection(db, "logins"));
  usuariosAcesso = uSnap.docs.map(d => ({ id: d.id, ...d.data() }));
  return usuariosAcesso;
}

async function validarCredenciaisLocal(u,p) {
  // se for admin fixo -> admin
  if (u === LOGIN_USER && p === LOGIN_PASS) {
    return { ok: true, admin: true, usuario: LOGIN_USER };
  }
  // carregar usuarios da coleção
  const us = await buscarUsuariosAcesso();
  const found = us.find(x => (x.usuario || '').toString() === u && (x.senha || '').toString() === p);
  if (found) return { ok: true, admin: false, usuario: found.usuario };
  return { ok: false };
}

/* registra o acesso no Firestore (coleção 'acessos') */
async function registrarAcesso(usuario) {
  try {
    const now = new Date();
    const ua = navigator.userAgent || '';
    // id automático via addDoc
    await addDoc(collection(db, "acessos"), {
      usuario,
      horarioISO: now.toISOString(),
      userAgent: ua,
      ip: '' // ip não preenchido aqui (exigir serviço externo)
    });
  } catch (err) {
    console.error('Erro ao registrar acesso:', err);
  }
}

document.getElementById('loginBtn').onclick = async () => {
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  const res = await validarCredenciaisLocal(u,p);
  if (res.ok) {
    usuarioLogado = res.usuario;
    isAdmin = !!res.admin;
    // registrar evento de acesso
    registrarAcesso(usuarioLogado);
    // esconder login e mostrar app
    loginScreen.style.display = 'none';
    mainApp.classList.remove('hidden');
    if (document.getElementById('remember').checked) {
      localStorage.setItem('autenticado','1');
      localStorage.setItem('usuarioLogado', usuarioLogado);
      localStorage.setItem('isAdmin', isAdmin ? '1' : '0');
    }
    // mostrar botão Gerenciar Acessos apenas para admin
    document.getElementById('gerenciarAcessosBtn').style.display = isAdmin ? 'inline-block' : 'none';
    iniciarLeituras();
    // carregar usuarios para o admin ver
    if (isAdmin) carregarUsuariosUI();
  } else {
    document.getElementById('loginMsg').textContent = 'Usuário ou senha incorretos.';
  }
};

if (localStorage.getItem('autenticado') === '1') {
  usuarioLogado = localStorage.getItem('usuarioLogado') || null;
  isAdmin = localStorage.getItem('isAdmin') === '1';
  // registrarAcesso? não registrar automático ao recarregar a sessão armazenada (apenas em login novo)
  loginScreen.style.display = 'none';
  mainApp.classList.remove('hidden');
  document.getElementById('gerenciarAcessosBtn').style.display = isAdmin ? 'inline-block' : 'none';
  iniciarLeituras();
  if (isAdmin) carregarUsuariosUI();
}

document.getElementById('logoutBtn').onclick = () => { 
  localStorage.removeItem('autenticado'); 
  localStorage.removeItem('usuarioLogado'); 
  localStorage.removeItem('isAdmin');
  location.reload(); 
};

/* ---------- RELÓGIO ---------- */
setInterval(() => { document.getElementById('clock').textContent = new Date().toLocaleTimeString('pt-BR',{hour12:false}); }, 1000);

/* ---------- LEITURAS FIRESTORE ---------- */
async function iniciarLeituras(){
  document.getElementById('status').textContent = "Carregando...";
  const colSnap = await getDocs(collection(db, "colaboradores"));
  colaboradores = colSnap.docs.map(d => ({ id: d.id, ...d.data() }));
  const ptSnap = await getDocs(collection(db, "pontos"));
  pontos = ptSnap.docs.map(d => ({ id: d.id, ...d.data() }));
  renderAll();

  onSnapshot(collection(db,"colaboradores"), snap => {
    colaboradores = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    renderColaboradores(document.getElementById('search').value.toLowerCase());
    popularColabSelect();
    document.getElementById('status').textContent = "Online • Firebase";
  });

  onSnapshot(collection(db,"pontos"), snap => {
    pontos = snap.docs.map(d => ({ id: d.id, ...d.data() }));
    renderEntradasSaidas();
    calcularHoras();
    document.getElementById('status').textContent = "Online • Firebase";
  });

  // manter lista de usuarios de acesso atualizada (se admin)
  if (isAdmin) {
    onSnapshot(collection(db,"logins"), snap => {
      usuariosAcesso = snap.docs.map(d => ({ id: d.id, ...d.data() }));
      carregarUsuariosUI();
    });
  }
}

/* ---------- RENDER GERAL ---------- */
function renderAll(){
  renderColaboradores();
  renderEntradasSaidas();
  calcularHoras();
  popularColabSelect();
}

/* busca */
document.getElementById('search').addEventListener('input', () => {
  renderColaboradores(document.getElementById('search').value.toLowerCase());
});

/* ---------- Obter próximo ID via transação (contagem contínua) ---------- */
async function obterProximoIdNum() {
  const counterRef = doc(db, 'meta', 'counters');
  const next = await runTransaction(db, async (tx) => {
    const snap = await tx.get(counterRef);
    let last = 0;
    if (snap.exists()) {
      const data = snap.data();
      last = Number(data.lastId) || 0;
    }
    const novo = last + 1;
    tx.set(counterRef, { lastId: novo }, { merge: true });
    return novo;
  });
  return String(next);
}

/* ---------- RENDER COLABORADORES ---------- */
function renderColaboradores(filtro = '') {
  const body = document.getElementById('colabBody');
  if (!body) return;
  body.innerHTML = '';
  colaboradores
    .slice()
    .sort((a,b) => (a.nome||'').localeCompare(b.nome||''))
    .filter(c => (c.nome||'').toLowerCase().includes(filtro) || (c.cargo||'').toLowerCase().includes(filtro) || (c.matricula||'').toLowerCase().includes(filtro) || (c.email||'').toLowerCase().includes(filtro))
    .forEach((c,i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${i+1}</td>
        <td>${c.id}</td>
        <td>${c.nome||''}</td>
        <td>${c.cargo||''}</td>
        <td>${c.matricula||''} <span class="small">${c.email||''}</span></td>
        <td>${c.turno||''}</td>
        <td>
          <button class="add btnEntrada">Entrada</button>
          <button class="secondary btnSaida">Saída</button>
          <button class="secondary scanBtn">Scanner</button>
          <button class="secondary editBtn">Editar</button>
          <button class="danger delBtn">Excluir</button>
        </td>`;

      tr.querySelector('.btnEntrada').onclick = () => registrarPonto(c.id, 'Entrada');
      tr.querySelector('.btnSaida').onclick = () => registrarPonto(c.id, 'Saída');

      /* BOTÃO SCANNER INDIVIDUAL */
      tr.querySelector('.scanBtn').onclick = () => {
        iniciarScannerPara(c.matricula);
      };

      tr.querySelector('.editBtn').onclick = () => abrirModalEditar(c);
      tr.querySelector('.delBtn').onclick = () => removerColab(c.id);
      body.appendChild(tr);
    });
}

/* ---------- Modal Colaborador ---------- */
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
  nomeInput.value = c.nome||'';
  cargoInput.value = c.cargo||'';
  matriculaInput.value = c.matricula||'';
  emailInput.value = c.email||'';
  turnoInput.value = c.turno||'';
  colabModal.classList.remove('hidden');
}
function fecharModalColab(){ colabModal.classList.add('hidden'); }

document.getElementById('saveColab').onclick = async () => {
  const nome = nomeInput.value.trim();
  if (!nome) return alert('Informe o nome do colaborador');
  const obj = { nome, cargo:cargoInput.value.trim(), matricula:matriculaInput.value.trim(), email:emailInput.value.trim(), turno:turnoInput.value.trim() };

  if (colabEmEdicao && colabEmEdicao.id) {
    // edição mantém o mesmo id
    await setDoc(doc(db,"colaboradores",colabEmEdicao.id), {...colabEmEdicao, ...obj});
  } else {
    // novo: pega próximo id via transação (continua contando pra sempre)
    const newId = await obterProximoIdNum();
    await setDoc(doc(db,"colaboradores",newId), { id:newId, ...obj });
  }
  fecharModalColab();
};

/* ---------- Registrar ponto ---------- */
async function registrarPonto(idColab, tipo) {
  const c = colaboradores.find(x => x.id === idColab);
  if (!c) return alert("Colaborador não encontrado!");
  const now = new Date();
  const p = { id: Date.now().toString(), idColab, nome: c.nome, matricula: c.matricula, email: c.email, tipo, data: now.toLocaleDateString('pt-BR'), hora: now.toLocaleTimeString('pt-BR',{hour12:false}), horarioISO: now.toISOString() };
  pontos.push(p);
  renderEntradasSaidas();
  await setDoc(doc(db,"pontos",p.id), p);
}

/* ---------- Funções mês atual ---------- */
function pontosDoMesAtual(pArray) {
  const hoje = new Date();
  const ano = String(hoje.getFullYear());
  const mes = String(hoje.getMonth()+1).padStart(2,'0');
  return pArray.filter(p => {
    const [d,m,a] = p.data.split('/');
    return a === ano && m === mes;
  });
}

/* ---------- Entradas / Saídas (mês atual) ---------- */
function renderEntradasSaidas() {
  const entBody = document.getElementById('entradasBody');
  const saiBody = document.getElementById('saidasBody');
  entBody.innerHTML = ''; saiBody.innerHTML = '';

  const pts = pontosDoMesAtual(pontos);

  let eIdx=1, sIdx=1;
  pts.filter(p => p.tipo === 'Entrada').forEach((p) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${eIdx++}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
    tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
    entBody.appendChild(tr);
  });

  pts.filter(p => p.tipo === 'Saída').forEach((p) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${sIdx++}</td><td>${p.idColab}</td><td>${p.nome}</td><td>${p.data}</td><td>${p.hora}</td><td><button class="danger delP">Excluir</button></td>`;
    tr.querySelector('.delP').onclick = () => excluirPonto(p.id);
    saiBody.appendChild(tr);
  });

  calcularHoras();
}

/* ---------- Excluir ponto ---------- */
async function excluirPonto(id) {
  if (confirm("Excluir este ponto permanentemente?")) {
    pontos = pontos.filter(p => p.id !== id);
    renderEntradasSaidas();
    await deleteDoc(doc(db,"pontos",id));
  }
}

/* ---------- Remover colaborador ---------- */
async function removerColab(id) {
  if (confirm("Excluir colaborador permanentemente?")) {
    colaboradores = colaboradores.filter(c => c.id !== id);
    pontos = pontos.filter(p => p.idColab !== id);
    renderAll();
    await deleteDoc(doc(db,"colaboradores", id));
    const pts = await getDocs(collection(db,"pontos"));
    for (let d of pts.docs) if (d.data().idColab === id) await deleteDoc(doc(db,"pontos", d.id));
    // NÃO reiniciamos o contador aqui: o meta/counters mantém lastId
  }
}

/* ---------- Apagar todos colaboradores (mantém contador) ---------- */
document.getElementById('limparTodosColabsBtn').onclick = async () => {
  if (!confirm("Deseja apagar TODOS os colaboradores e seus pontos? Isto NÃO vai reiniciar o contador (IDs continuarão aumentando).")) return;
  // apagar colaboradores
  const col = await getDocs(collection(db,"colaboradores"));
  for (let d of col.docs) await deleteDoc(doc(db,"colaboradores", d.id));
  // apagar pontos
  const pts = await getDocs(collection(db,"pontos"));
  for (let d of pts.docs) await deleteDoc(doc(db,"pontos", d.id));
  colaboradores = []; pontos = [];
  renderAll();
  alert('Todos os colaboradores e pontos foram apagados. O contador continuará de onde parou.');
}

/* ---------- Limpar todos os pontos (mantém colaboradores e contador) ---------- */
document.getElementById('limparTodosPontosBtn').onclick = async () => {
  if (!confirm("Deseja realmente excluir todos os pontos?")) return;
  const col = await getDocs(collection(db,"pontos"));
  for (let d of col.docs) await deleteDoc(doc(db,"pontos", d.id));
  pontos = [];
  renderEntradasSaidas();
}

/* ---------- UTIL / Formatação de tempo (hh mm ss) ---------- */
function formatarHorasSegundos(totalSegundos) {
  totalSegundos = Math.max(0, Math.round(totalSegundos)); // evitar negativos e garantir inteiro
  const horas = Math.floor(totalSegundos / 3600);
  const minutos = Math.floor((totalSegundos % 3600) / 60);
  const segundos = totalSegundos % 60;
  return `${horas}h ${minutos}m ${segundos}s`;
}

/* ---------- Calcular horas (mês atual) - agora com segundos ---------- */
function calcularHoras() {
  const horasBody = document.getElementById('horasBody');
  const totalHorasCell = document.getElementById('totalHoras');
  horasBody.innerHTML = '';
  let dados = {}; // dados[nome][data] = lista pontos
  let totalGeralSegundos = 0;

  const pts = pontosDoMesAtual(pontos);

  pts.forEach(p => {
    if (!dados[p.nome]) dados[p.nome] = {};
    if (!dados[p.nome][p.data]) dados[p.nome][p.data] = [];
    dados[p.nome][p.data].push(p);
  });

  Object.keys(dados).forEach(nome => {
    Object.keys(dados[nome]).forEach(data => {
      // ordenar por horarioISO
      let reg = dados[nome][data].slice().sort((a,b) => new Date(a.horarioISO) - new Date(b.horarioISO));
      let entrada = null;
      let totalSegundosPorDia = 0;
      reg.forEach(r => {
        if (r.tipo === 'Entrada') {
          entrada = new Date(r.horarioISO);
        } else if (r.tipo === 'Saída' && entrada) {
          const saida = new Date(r.horarioISO);
          const diffSeg = Math.round((saida - entrada) / 1000);
          if (diffSeg > 0) totalSegundosPorDia += diffSeg;
          entrada = null;
        }
      });
      totalGeralSegundos += totalSegundosPorDia;
      const tempoFormatado = formatarHorasSegundos(totalSegundosPorDia);
      const tr = document.createElement('tr');
      tr.innerHTML = `<td>${nome}</td><td>${data}</td><td>${tempoFormatado}</td>`;
      horasBody.appendChild(tr);
    });
  });

  totalHorasCell.textContent = formatarHorasSegundos(totalGeralSegundos);
}

/* ---------- Exportar Excel (mês atual) + aba de Resumo com hh mm ss (apenas quem bateou SAÍDA) ---------- */
document.getElementById('baixarBtn').onclick = () => {
  const ptsMes = pontosDoMesAtual(pontos);
  const entradas = [['#','ID Colab','Nome','Data','Hora']];
  const saidas = [['#','ID Colab','Nome','Data','Hora']];

  // preparar resumo: totalSegundos por colaborador (somente pares entrada->saida)
  const resumoSegundos = {}; // { nome: totalSegundos }
  // Para evitar confusão de indices, vamos agrupar por colaborador e data e processar em ordem
  const byPersonDate = {};
  ptsMes.forEach(p => {
    const key = `${p.nome}||${p.data}`;
    if (!byPersonDate[key]) byPersonDate[key] = [];
    byPersonDate[key].push(p);
  });

  // preencher Entradas / Saidas arrays e calcular resumo
  let eIdx = 1, sIdx = 1;
  // Entradas e Saídas na ordem original de ponto array filtrada por mês (mantém similar ao que usuário vê)
  ptsMes.forEach((p) => {
    if (p.tipo === 'Entrada') entradas.push([eIdx++, p.idColab, p.nome, p.data, p.hora]);
    if (p.tipo === 'Saída') saidas.push([sIdx++, p.idColab, p.nome, p.data, p.hora]);
  });

  // agora calcular resumo por pessoa (somente pares entrada->saída válidos)
  Object.keys(byPersonDate).forEach(k => {
    const arr = byPersonDate[k].slice().sort((a,b) => new Date(a.horarioISO) - new Date(b.horarioISO));
    let entrada = null;
    arr.forEach(item => {
      if (item.tipo === 'Entrada') entrada = new Date(item.horarioISO);
      else if (item.tipo === 'Saída' && entrada) {
        const saida = new Date(item.horarioISO);
        const diffSegundos = Math.round((saida - entrada) / 1000);
        if (diffSegundos > 0) {
          resumoSegundos[item.nome] = (resumoSegundos[item.nome] || 0) + diffSegundos;
        }
        entrada = null;
      }
    });
  });

  // filtrar resumo para incluir apenas quem teve pelo menos uma Saída
  const nomesComSaida = new Set(ptsMes.filter(p => p.tipo === 'Saída').map(p => p.nome));
  const wsResumo = [['Colaborador','Horas Trabalhadas (mês atual)']];
  Object.keys(resumoSegundos).forEach(nome => {
    if (nomesComSaida.has(nome)) {
      wsResumo.push([nome, formatarHorasSegundos(resumoSegundos[nome])]);
    }
  });

  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(entradas), 'Entradas');
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(saidas), 'Saídas');
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(wsResumo), 'Resumo de Horas');
  XLSX.writeFile(wb, `Pontos_${filtroAtual}.xlsx`);
};

/* ---------- Relatório Geral Semanal/Mensal (mês atual) - export já existente (mantive) ---------- */
document.getElementById('gerarRelatorioBtn').onclick = () => {
  const pts = pontosDoMesAtual(pontos).slice().sort((a,b)=> new Date(a.horarioISO) - new Date(b.horarioISO));
  const rel = {};
  function getMonday(d){ const date = new Date(d); const day = date.getDay(); const diff = date.getDate() - day + (day===0? -6:1); const m = new Date(date.setDate(diff)); m.setHours(0,0,0,0); return m; }
  const byPerson = {};
  pts.forEach(p => { if (!byPerson[p.nome]) byPerson[p.nome]=[]; byPerson[p.nome].push(p); });
  Object.keys(byPerson).forEach(nome => {
    rel[nome] = { semanal:{}, mensal:0 };
    const regs = byPerson[nome].slice().sort((a,b)=> new Date(a.horarioISO)-new Date(b.horarioISO));
    let entrada = null;
    regs.forEach(r => {
      const dt = new Date(r.horarioISO);
      if (r.tipo === 'Entrada') entrada = dt;
      else if (r.tipo === 'Saída' && entrada) {
        const h = (dt - entrada)/3600000;
        const monday = getMonday(entrada).toLocaleDateString('pt-BR');
        rel[nome].semanal[monday] = (rel[nome].semanal[monday] || 0) + h;
        rel[nome].mensal += h;
        entrada = null;
      }
    });
  });
  const ws = [['Funcionário','Semana (segunda)','Horas Semana','Total Mensal']];
  Object.keys(rel).forEach(nome => {
    const semanas = Object.keys(rel[nome].semanal).sort((a,b)=> {
      const pa = a.split('/').reverse().join('-'); const pb = b.split('/').reverse().join('-'); return new Date(pa)-new Date(pb);
    });
    if (semanas.length === 0) ws.push([nome,'-','-', `${Math.floor(rel[nome].mensal)}h ${Math.round((rel[nome].mensal-Math.floor(rel[nome].mensal))*60)}m`]);
    else {
      semanas.forEach(sk => {
        const h = rel[nome].semanal[sk];
        ws.push([nome, sk, `${Math.floor(h)}h ${Math.round((h-Math.floor(h))*60)}m`, '']);
      });
      ws.push([nome, 'Total Mês', '', `${Math.floor(rel[nome].mensal)}h ${Math.round((rel[nome].mensal-Math.floor(rel[nome].mensal))*60)}m`]);
    }
  });
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(ws), 'Relatorio_Horas');
  XLSX.writeFile(wb, `Relatorio_Horas_${filtroAtual}.xlsx`);
};

/* ---------- Relatório por colaborador (visualizar/exportar) ---------- */
function getMonday(d) {
  const date = new Date(d); const day = date.getDay(); const diff = date.getDate() - day + (day === 0 ? -6 : 1); const monday = new Date(date.setDate(diff)); monday.setHours(0,0,0,0); return monday;
}
function gerarRelatorioPorColaborador(nome) {
  const rel = { semanal:{}, mensal:0, nome };
  const regsAll = pontosDoMesAtual(pontos).filter(p => p.nome === nome).sort((a,b)=> new Date(a.horarioISO)-new Date(b.horarioISO));
  let entrada = null;
  regsAll.forEach(r => {
    const dt = new Date(r.horarioISO);
    if (r.tipo === 'Entrada') entrada = dt;
    else if (r.tipo === 'Saída' && entrada) {
      const hours = (dt - entrada)/3600000;
      const monday = getMonday(entrada).toLocaleDateString('pt-BR');
      rel.semanal[monday] = (rel.semanal[monday]||0) + hours;
      rel.mensal += hours;
      entrada = null;
    }
  });
  return rel;
}
document.getElementById('verRelatorioColabBtn').onclick = () => {
  const nome = colabSelect.value;
  if (!nome) return alert('Selecione um colaborador');
  const rel = gerarRelatorioPorColaborador(nome);
  let html = `<p><b>Colaborador:</b> ${rel.nome}</p>`;
  html += `<table style="width:100%;border-collapse:collapse"><thead><tr style="background:#f3f4f6"><th>Semana (segunda)</th><th>Horas</th></tr></thead><tbody>`; 
  const semanas = Object.keys(rel.semanal).sort((a,b)=> { const pa=a.split('/').reverse().join('-'); const pb=b.split('/').reverse().join('-'); return new Date(pa)-new Date(pb); });
  if (semanas.length === 0) html += `<tr><td colspan="2">Sem registros no mês atual</td></tr>`;
  else semanas.forEach(sk => { const h = rel.semanal[sk]; html += `<tr><td>${sk}</td><td>${Math.floor(h)}h ${Math.round((h-Math.floor(h))*60)}m</td></tr>`; });
  html += `</tbody><tfoot><tr style="background:#fbfdfe"><td><b>Total mês</b></td><td><b>${Math.floor(rel.mensal)}h ${Math.round((rel.mensal-Math.floor(rel.mensal))*60)}m</b></td></tr></tfoot></table>`;
  document.getElementById('relColabContent').innerHTML = html;
  document.getElementById('relColabModal').classList.remove('hidden');
};
document.getElementById('closeRelColab').onclick = () => document.getElementById('relColabModal').classList.add('hidden');

document.getElementById('exportRelatorioColabBtn').onclick = () => {
  const nome = colabSelect.value;
  if (!nome) return alert('Selecione um colaborador');
  const rel = gerarRelatorioPorColaborador(nome);
  const ws = [['Semana (segunda)','Horas Semana']];
  const semanas = Object.keys(rel.semanal).sort((a,b)=> { const pa=a.split('/').reverse().join('-'); const pb=b.split('/').reverse().join('-'); return new Date(pa)-new Date(pb); });
  semanas.forEach(sk => { const h = rel.semanal[sk]; ws.push([sk, `${Math.floor(h)}h ${Math.round((h-Math.floor(h))*60)}m`]); });
  ws.push(['Total mês', `${Math.floor(rel.mensal)}h ${Math.round((rel.mensal-Math.floor(rel.mensal))*60)}m`]);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, XLSX.utils.aoa_to_sheet(ws), 'Relatorio_Colaborador');
  XLSX.writeFile(wb, `Relatorio_${nome.replace(/\s+/g,'_')}_${filtroAtual}.xlsx`);
};
/* ---------- popular select ---------- */
function popularColabSelect() {
  colabSelect.innerHTML = '<option value="">-- selecione --</option>';
  colaboradores
    .slice()
    .sort((a,b) => (a.nome||'').localeCompare(b.nome||''))
    .forEach(c => {
      const opt = document.createElement('option');
      opt.value = c.nome || c.id;
      opt.textContent = `${c.id} - ${c.nome || c.id}`;
      colabSelect.appendChild(opt);
    });
}

/* ---------- inicialização UI ---------- */
function inicializarUI(){
  popularColabSelect();
}
inicializarUI();

/* =================== GESTÃO DE USUÁRIOS (Acessos) =================== */
/* Mostrar modal de usuarios (apenas admin) */
const gerenciarAcessosBtn = document.getElementById('gerenciarAcessosBtn');
const usuariosModal = document.getElementById('usuariosModal');
const usuariosList = document.getElementById('usuariosList');
const novoUsuarioInput = document.getElementById('novoUsuario');
const novaSenhaInput = document.getElementById('novaSenha');
const addUsuarioBtn = document.getElementById('addUsuarioBtn');
const closeUsuarios = document.getElementById('closeUsuarios');

const verAcessosBtn = document.getElementById('verAcessosBtn');
const acessosModal = document.getElementById('acessosModal');
const acessosList = document.getElementById('acessosList');
const closeAcessos = document.getElementById('closeAcessos');
const limparAcessosBtn = document.getElementById('limparAcessosBtn');

gerenciarAcessosBtn.onclick = () => {
  if (!isAdmin) return alert('Apenas o administrador pode gerenciar acessos.');
  usuariosModal.classList.remove('hidden');
  carregarUsuariosUI();
};
closeUsuarios.onclick = () => usuariosModal.classList.add('hidden');

/* carregar lista de usuarios na UI */
async function carregarUsuariosUI() {
  usuariosList.innerHTML = '';
  const uSnap = await getDocs(collection(db, "logins"));
  usuariosAcesso = uSnap.docs.map(d => ({ id: d.id, ...d.data() }));

  if (usuariosAcesso.length === 0) {
    usuariosList.innerHTML = '<div style="padding:8px;color:#6b7280">Nenhum usuário de acesso cadastrado.</div>';
  } else {
    usuariosAcesso.forEach(u => {
      const div = document.createElement('div');
      div.className = 'usr-row';
      div.innerHTML = `<div>${u.usuario}</div>
        <div style="display:flex;gap:6px">
          <button class="secondary editUserBtn" data-id="${u.id}">Editar</button>
          <button class="danger delUserBtn" data-id="${u.id}">Excluir</button>
        </div>`;
      usuariosList.appendChild(div);
    });

    // ligar eventos
    usuariosList.querySelectorAll('.delUserBtn').forEach(btn => {
      btn.onclick = async (ev) => {
        const id = ev.currentTarget.getAttribute('data-id');
        if (!confirm('Excluir este usuário de acesso?')) return;
        await deleteDoc(doc(db, "logins", id));
        carregarUsuariosUI();
      };
    });
    usuariosList.querySelectorAll('.editUserBtn').forEach(btn => {
      btn.onclick = (ev) => {
        const id = ev.currentTarget.getAttribute('data-id');
        const uobj = usuariosAcesso.find(x => x.id === id);
        if (!uobj) return;
        const novo = prompt('Novo nome de usuário:', uobj.usuario);
        if (novo === null) return;
        const novaSenha = prompt('Nova senha (vazio = manter):', '');
        (async () => {
          const ref = doc(db, "logins", id);
          const obj = { usuario: novo };
          if (novaSenha && novaSenha.trim().length > 0) obj.senha = novaSenha.trim();
          await setDoc(ref, obj, { merge: true });
          carregarUsuariosUI();
        })();
      };
    });
  }
}
/* adicionar novo usuario (apenas admin) */
addUsuarioBtn.onclick = async () => {
  if (!isAdmin) return alert('Apenas o administrador pode adicionar usuários.');
  const usuario = novoUsuarioInput.value.trim();
  const senha = novaSenhaInput.value.trim();
  if (!usuario || !senha) return alert('Informe usuário e senha.');
  // cria doc com id automático
  const newId = Date.now().toString();
  await setDoc(doc(db, "logins", newId), { usuario, senha });
  novoUsuarioInput.value = '';
  novaSenhaInput.value = '';
  carregarUsuariosUI();
};

/* ---------- ACESSOS: visualizar e limpar ---------- */
verAcessosBtn.onclick = async () => {
  usuariosModal.classList.add('hidden');
  acessosModal.classList.remove('hidden');
  carregarAcessosUI();
  // inscrever atualização em tempo real
  onSnapshot(collection(db, "acessos"), snap => {
    carregarAcessosUI(); // atualiza
  });
};
closeAcessos.onclick = () => acessosModal.classList.add('hidden');

/* carrega lista de acessos (mais recentes primeiro) */
async function carregarAcessosUI() {
  const snap = await getDocs(collection(db, "acessos"));
  const docs = snap.docs.map(d => ({ id: d.id, ...d.data() })).sort((a,b)=> new Date(b.horarioISO) - new Date(a.horarioISO));
  if (docs.length === 0) {
    acessosList.innerHTML = '<div style="padding:8px;color:#6b7280">Nenhum acesso registrado.</div>';
    return;
  }
  acessosList.innerHTML = '';
  docs.forEach(d => {
    const dt = new Date(d.horarioISO);
    const dtStr = dt.toLocaleString('pt-BR', { hour12:false });
    const ua = (d.userAgent || '').slice(0,140);
    const div = document.createElement('div');
    div.className = 'acc-row';
    div.innerHTML = `<div><b>${d.usuario}</b> — ${dtStr}</div><div style="color:#6b7280;font-size:12px">${ua}</div>`;
    acessosList.appendChild(div);
  });
}
/* limpar logs de acessos (apenas admin) */
limparAcessosBtn.onclick = async () => {
  if (!isAdmin) return alert('Apenas admin pode limpar logs.');
  if (!confirm('Limpar todos os logs de acessos? Isso NÃO pode ser desfeito.')) return;
  const snap = await getDocs(collection(db, "acessos"));
  for (let d of snap.docs) {
    await deleteDoc(doc(db, "acessos", d.id));
  }
  alert('Logs de acessos limpos.');
  carregarAcessosUI();
};
  // === HISTÓRICO PARA ENTRAR/SAIR AUTOMÁTICO ===
function getProximoTipo(id) {
    const chave = "ultimoTipo_" + id;
    const anterior = localStorage.getItem(chave);

    if (!anterior || anterior === "saida") {
        localStorage.setItem(chave, "entrada");
        return "entrada";
    } else {
        localStorage.setItem(chave, "saida");
        return "saida";
    }
}

let html5Qr = null;

document.getElementById("btnScanner").addEventListener("click", () => {
    document.getElementById("scannerModal").style.display = "flex";

    html5Qr = new Html5Qrcode("reader");

    html5Qr.start(
        { facingMode: "environment" },
        { fps: 10, qrbox: 250 },
        (textoLido) => {
            let idLido = textoLido.trim();
            let tipo = getProximoTipo(idLido);

            registrarPonto(idLido, tipo);
            fecharScanner();
        },
        (erro) => {}
    );
});

function fecharScanner() {
    document.getElementById("scannerModal").style.display = "none";
    if (html5Qr) {
        html5Qr.stop().then(() => { html5Qr.clear(); });
    }
}

</script>
<!-- MODAL DO SCANNER -->
<div id="scannerModal" 
 style="display:none; position: fixed; top:0; left:0; width:100%; height:100%;
        background: rgba(0,0,0,0.7); justify-content:center; align-items:center;">
    <div style="background:white; padding:15px; border-radius:8px;">
        <h3>Scanner de Código</h3>
        <div id="reader" style="width:300px; height:300px;"></div>
        <button onclick="fecharScanner()" style="margin-top:10px;">Fechar</button>
    </div>
</div>

</body>
</html>

