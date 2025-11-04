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
  #usuariosList { max-height:220px; overflow:auto; margin-bottom:10px; border:1px solid #eef2f6; border-radius:6px; padding:8px; background:#fafafa; }
  .usr-row{display:flex;justify-content:space-between;align-items:center;padding:6px 8px;border-bottom:1px solid #f1f5f9}
  .gear { width:36px;height:36px;border-radius:8px;display:flex;align-items:center;justify-content:center;cursor:pointer;background:transparent;border:none;color:#fff;font-size:18px }
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
      <button class="gear secondary" id="gerenciarAcessosBtn" title="Gerenciar Logins" style="display:none">⚙️</button>
      <button class="secondary" id="logoutBtn">Sair</button>
    </div>
  </div>
</header>
<main>
  <input type="text" id="searchInput" class="search" placeholder="Pesquisar colaborador...">

  <table id="colaboradoresTable">
    <thead>
      <tr>
        <th>ID</th>
        <th>Nome</th>
        <th>Departamento</th>
        <th>Pontos</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody id="colaboradoresBody">
      <!-- Linhas serão preenchidas pelo JS -->
    </tbody>
  </table>
</main>

<!-- MODAL DE PONTOS -->
<div id="modalPontos" class="modal hidden">
  <div class="modal-content">
    <h3>Gerenciar Pontos de <span id="modalNomeColab"></span></h3>
    <div id="pontosList"></div>
    <button class="danger" id="limparPontosColabBtn">Limpar Pontos</button>
    <button class="secondary" id="fecharModalPontosBtn">Fechar</button>
  </div>
</div>

<!-- MODAL DE ACESSOS -->
<div id="modalAcessos" class="modal hidden">
  <div class="modal-content">
    <h3>Gerenciar Acessos</h3>
    <div id="usuariosList"></div>
    <button class="add" id="addUsuarioBtn">Adicionar Usuário</button>
    <button class="secondary" id="fecharModalAcessosBtn">Fechar</button>
    <div id="acessosList"></div>
  </div>
</div>

<script>
let usuarios = JSON.parse(localStorage.getItem('usuarios')) || [
  {user:'admin', pass:'admin', role:'admin'},
  {user:'user', pass:'user', role:'user'}
];
let logado = null;

// Função de login
document.getElementById('loginBtn').addEventListener('click', ()=>{
  const u = document.getElementById('user').value.trim();
  const p = document.getElementById('pass').value.trim();
  const found = usuarios.find(x => x.user === u && x.pass === p);
  if(found){
    logado = found;
    document.getElementById('loginScreen').classList.add('hidden');
    document.getElementById('gerenciarAcessosBtn').style.display = (found.role==='admin') ? 'inline-flex' : 'none';
    document.getElementById('status').textContent = `Online • ${found.user} (${found.role})`;
  } else {
    document.getElementById('loginMsg').textContent = 'Usuário ou senha inválidos';
  }
});

document.getElementById('logoutBtn').addEventListener('click', ()=>{
  logado = null;
  document.getElementById('loginScreen').classList.remove('hidden');
  document.getElementById('gerenciarAcessosBtn').style.display = 'none';
  document.getElementById('status').textContent = 'Offline • Local Storage';
});

// Função de relógio
function atualizarRelogio(){
  const now = new Date();
  document.getElementById('clock').textContent = now.toLocaleTimeString();
}
setInterval(atualizarRelogio,1000);
atualizarRelogio();
<script>
// --- DADOS DE COLABORADORES ---
let colaboradores = JSON.parse(localStorage.getItem('colaboradores')) || [
  {id:1, nome:'João', depto:'Vendas', pontos:10},
  {id:2, nome:'Maria', depto:'Marketing', pontos:15}
];

// --- FUNÇÃO PARA ATUALIZAR TABELA ---
function atualizarTabela(){
  const tbody = document.getElementById('colaboradoresBody');
  tbody.innerHTML = '';
  colaboradores.forEach(c => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${c.id}</td>
      <td>${c.nome}</td>
      <td>${c.depto}</td>
      <td>${c.pontos}</td>
      <td>
        <button onclick="abrirModalPontos(${c.id})">Pontos</button>
      </td>
    `;
    tbody.appendChild(tr);
  });
}
atualizarTabela();

// --- MODAL DE PONTOS ---
function abrirModalPontos(id){
  if(!logado) return alert('Faça login primeiro!');
  const colab = colaboradores.find(c => c.id === id);
  document.getElementById('modalNomeColab').textContent = colab.nome;
  document.getElementById('pontosList').innerHTML = `
    <p>Pontos atuais: ${colab.pontos}</p>
    <button onclick="adicionarPonto(${id})">Adicionar Ponto</button>
  `;
  document.getElementById('modalPontos').classList.remove('hidden');
}

function adicionarPonto(id){
  if(logado.role !== 'admin') return alert('Apenas administradores podem adicionar pontos.');
  const colab = colaboradores.find(c => c.id === id);
  colab.pontos += 1;
  salvarColaboradores();
  abrirModalPontos(id);
  atualizarTabela();
}

document.getElementById('limparPontosColabBtn').addEventListener('click', ()=>{
  if(logado.role !== 'admin') return alert('Apenas administradores podem limpar pontos.');
  const nome = document.getElementById('modalNomeColab').textContent;
  const colab = colaboradores.find(c => c.nome === nome);
  colab.pontos = 0;
  salvarColaboradores();
  abrirModalPontos(colab.id);
  atualizarTabela();
});

document.getElementById('fecharModalPontosBtn').addEventListener('click', ()=>{
  document.getElementById('modalPontos').classList.add('hidden');
});

// --- PESQUISA ---
document.getElementById('searchInput').addEventListener('input', function(){
  const termo = this.value.toLowerCase();
  const rows = document.querySelectorAll('#colaboradoresBody tr');
  rows.forEach(row => {
    const nome = row.cells[1].textContent.toLowerCase();
    row.style.display = nome.includes(termo) ? '' : 'none';
  });
});

// --- SALVAR NO LOCALSTORAGE ---
function salvarColaboradores(){
  localStorage.setItem('colaboradores', JSON.stringify(colaboradores));
}

// --- DOWNLOAD CSV ---
document.getElementById('downloadBtn').addEventListener('click', ()=>{
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem baixar os dados.');
  let csv = 'ID,Nome,Departamento,Pontos\n';
  colaboradores.forEach(c => {
    csv += `${c.id},${c.nome},${c.depto},${c.pontos}\n`;
  });
  const blob = new Blob([csv], {type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'colaboradores.csv';
  a.click();
  URL.revokeObjectURL(url);
});
</script>
<script>
// --- USUÁRIOS E LOGIN ---
let usuarios = JSON.parse(localStorage.getItem('usuarios')) || [
  {username:'admin', password:'123', role:'admin'},
  {username:'user', password:'123', role:'user'}
];

let logado = null;

// --- LOGIN ---
document.getElementById('loginBtn').addEventListener('click', ()=>{
  const user = document.getElementById('username').value;
  const pass = document.getElementById('password').value;
  const found = usuarios.find(u => u.username === user && u.password === pass);
  if(found){
    logado = found;
    document.getElementById('loginForm').classList.add('hidden');
    document.getElementById('welcomeUser').textContent = `Bem-vindo, ${logado.username}!`;
    document.getElementById('welcomeUser').classList.remove('hidden');
    document.getElementById('adminControls').style.display = logado.role === 'admin' ? 'block' : 'none';
  } else {
    alert('Usuário ou senha incorretos!');
  }
});

// --- LOGOUT ---
document.getElementById('logoutBtn').addEventListener('click', ()=>{
  logado = null;
  document.getElementById('loginForm').classList.remove('hidden');
  document.getElementById('welcomeUser').classList.add('hidden');
});

// --- GERENCIAR USUÁRIOS (SOMENTE ADMIN) ---
document.getElementById('manageUsersBtn').addEventListener('click', ()=>{
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem gerenciar usuários.');
  const tbody = document.getElementById('usersBody');
  tbody.innerHTML = '';
  usuarios.forEach((u, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${u.username}</td>
      <td>${u.role}</td>
      <td>
        <button onclick="removerUsuario(${i})">Remover</button>
      </td>
    `;
    tbody.appendChild(tr);
  });
  document.getElementById('modalUsers').classList.remove('hidden');
});

function removerUsuario(index){
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem remover usuários.');
  if(usuarios[index].username === 'admin') return alert('Não é possível remover o admin principal!');
  usuarios.splice(index,1);
  salvarUsuarios();
  document.getElementById('manageUsersBtn').click(); // Atualiza tabela
}

document.getElementById('fecharModalUsersBtn').addEventListener('click', ()=>{
  document.getElementById('modalUsers').classList.add('hidden');
});

// --- ADICIONAR USUÁRIO ---
document.getElementById('addUserBtn').addEventListener('click', ()=>{
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem adicionar usuários.');
  const username = prompt('Nome do usuário:');
  const password = prompt('Senha:');
  const role = prompt('Função (admin/user):');
  if(username && password && (role==='admin'||role==='user')){
    usuarios.push({username,password,role});
    salvarUsuarios();
    alert('Usuário adicionado com sucesso!');
  } else {
    alert('Dados inválidos!');
  }
});

// --- SALVAR USUÁRIOS ---
function salvarUsuarios(){
  localStorage.setItem('usuarios', JSON.stringify(usuarios));
}
</script>
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Gestão de Colaboradores e Pontos</title>
<style>
.hidden { display: none; }
table, th, td { border: 1px solid black; border-collapse: collapse; padding: 5px; }
</style>
</head>
<body>

<!-- LOGIN -->
<div id="loginForm">
  <h2>Login</h2>
  <input type="text" id="username" placeholder="Usuário">
  <input type="password" id="password" placeholder="Senha">
  <button id="loginBtn">Entrar</button>
</div>

<div id="welcomeUser" class="hidden">
  <h2>Bem-vindo!</h2>
  <button id="logoutBtn">Sair</button>
</div>

<div id="adminControls" style="display:none;">
  <button id="manageUsersBtn">Gerenciar Usuários</button>
  <button id="addUserBtn">Adicionar Usuário</button>
</div>

<!-- MODAL DE USUÁRIOS -->
<div id="modalUsers" class="hidden">
  <h3>Usuários</h3>
  <table>
    <thead>
      <tr><th>Usuário</th><th>Função</th><th>Ação</th></tr>
    </thead>
    <tbody id="usersBody"></tbody>
  </table>
  <button id="fecharModalUsersBtn">Fechar</button>
</div>

<!-- COLABORADORES -->
<h2>Cadastro de Colaboradores</h2>
<input type="text" id="nomeColab" placeholder="Nome">
<input type="text" id="cpfColab" placeholder="CPF">
<button id="addColabBtn">Adicionar Colaborador</button>

<table>
  <thead>
    <tr><th>Nome</th><th>CPF</th></tr>
  </thead>
  <tbody id="colabsBody"></tbody>
</table>

<!-- PONTOS -->
<h2>Pontos</h2>
<input type="text" id="nomePonto" placeholder="Nome do ponto">
<input type="text" id="descricaoPonto" placeholder="Descrição">
<button id="addPontoBtn">Adicionar Ponto</button>

<table>
  <thead>
    <tr><th>Nome</th><th>Descrição</th></tr>
  </thead>
  <tbody id="pontosBody"></tbody>
</table>

<!-- PESQUISA -->
<h2>Pesquisar Colaborador</h2>
<input type="text" id="searchColab" placeholder="Nome">
<button id="searchBtn">Pesquisar</button>
<div id="searchResults"></div>

<!-- DOWNLOAD CSV -->
<h2>Download CSV</h2>
<button id="downloadColabsBtn">Download Colaboradores</button>
<button id="downloadPontosBtn">Download Pontos</button>

<script>
// --- USUÁRIOS E LOGIN ---
let usuarios = JSON.parse(localStorage.getItem('usuarios')) || [
  {username:'admin', password:'123', role:'admin'},
  {username:'user', password:'123', role:'user'}
];
let logado = null;

// --- LOGIN ---
document.getElementById('loginBtn').addEventListener('click', ()=>{
  const user = document.getElementById('username').value;
  const pass = document.getElementById('password').value;
  const found = usuarios.find(u => u.username === user && u.password === pass);
  if(found){
    logado = found;
    document.getElementById('loginForm').classList.add('hidden');
    document.getElementById('welcomeUser').textContent = `Bem-vindo, ${logado.username}!`;
    document.getElementById('welcomeUser').classList.remove('hidden');
    document.getElementById('adminControls').style.display = logado.role === 'admin' ? 'block' : 'none';
  } else {
    alert('Usuário ou senha incorretos!');
  }
});

// --- LOGOUT ---
document.getElementById('logoutBtn').addEventListener('click', ()=>{
  logado = null;
  document.getElementById('loginForm').classList.remove('hidden');
  document.getElementById('welcomeUser').classList.add('hidden');
});

// --- GERENCIAR USUÁRIOS (SOMENTE ADMIN) ---
document.getElementById('manageUsersBtn').addEventListener('click', ()=>{
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem gerenciar usuários.');
  const tbody = document.getElementById('usersBody');
  tbody.innerHTML = '';
  usuarios.forEach((u, i) => {
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td>${u.username}</td>
      <td>${u.role}</td>
      <td><button onclick="removerUsuario(${i})">Remover</button></td>
    `;
    tbody.appendChild(tr);
  });
  document.getElementById('modalUsers').classList.remove('hidden');
});

function removerUsuario(index){
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem remover usuários.');
  if(usuarios[index].username === 'admin') return alert('Não é possível remover o admin principal!');
  usuarios.splice(index,1);
  salvarUsuarios();
  document.getElementById('manageUsersBtn').click(); 
}

document.getElementById('fecharModalUsersBtn').addEventListener('click', ()=>{
  document.getElementById('modalUsers').classList.add('hidden');
});

// --- ADICIONAR USUÁRIO ---
document.getElementById('addUserBtn').addEventListener('click', ()=>{
  if(!logado || logado.role !== 'admin') return alert('Apenas administradores podem adicionar usuários.');
  const username = prompt('Nome do usuário:');
  const password = prompt('Senha:');
  const role = prompt('Função (admin/user):');
  if(username && password && (role==='admin'||role==='user')){
    usuarios.push({username,password,role});
    salvarUsuarios();
    alert('Usuário adicionado com sucesso!');
  } else {
    alert('Dados inválidos!');
  }
});

function salvarUsuarios(){
  localStorage.setItem('usuarios', JSON.stringify(usuarios));
}

// --- COLABORADORES ---
let colabs = JSON.parse(localStorage.getItem('colabs')) || [];
function renderColabs(){
  const tbody = document.getElementById('colabsBody');
  tbody.innerHTML = '';
  colabs.forEach(c=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${c.nome}</td><td>${c.cpf}</td>`;
    tbody.appendChild(tr);
  });
}
document.getElementById('addColabBtn').addEventListener('click', ()=>{
  const nome = document.getElementById('nomeColab').value;
  const cpf = document.getElementById('cpfColab').value;
  if(nome && cpf){
    colabs.push({nome, cpf});
    localStorage.setItem('colabs', JSON.stringify(colabs));
    renderColabs();
  }
});
renderColabs();

// --- PONTOS ---
let pontos = JSON.parse(localStorage.getItem('pontos')) || [];
function renderPontos(){
  const tbody = document.getElementById('pontosBody');
  tbody.innerHTML = '';
  pontos.forEach(p=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `<td>${p.nome}</td><td>${p.descricao}</td>`;
    tbody.appendChild(tr);
  });
}
document.getElementById('addPontoBtn').addEventListener('click', ()=>{
  const nome = document.getElementById('nomePonto').value;
  const desc = document.getElementById('descricaoPonto').value;
  if(nome && desc){
    pontos.push({nome, descricao: desc});
    localStorage.setItem('pontos', JSON.stringify(pontos));
    renderPontos();
  }
});
renderPontos();

// --- PESQUISA ---
document.getElementById('searchBtn').addEventListener('click', ()=>{
  const query = document.getElementById('searchColab').value.toLowerCase();
  const results = colabs.filter(c=>c.nome.toLowerCase().includes(query));
  const div = document.getElementById('searchResults');
  div.innerHTML = results.map(r=>`<p>${r.nome} - ${r.cpf}</p>`).join('');
});

// --- DOWNLOAD CSV ---
function downloadCSV(data, filename){
  const csv = data.map(r => Object.values(r).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = filename;
  a.click();
}
document.getElementById('downloadColabsBtn').addEventListener('click', ()=>downloadCSV(colabs,'colaboradores.csv'));
document.getElementById('downloadPontosBtn').addEventListener('click', ()=>downloadCSV(pontos,'pontos.csv'));

</script>

</body>
</html>
