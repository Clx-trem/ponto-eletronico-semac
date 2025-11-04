// =======================================
// SISTEMA DE PONTO ELETRÔNICO COM FIREBASE
// =======================================

// Inicialização do Firebase (mantida igual)
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.13.0/firebase-app.js";
import {
  getFirestore, collection, getDocs, addDoc, deleteDoc, doc, updateDoc, query, orderBy
} from "https://www.gstatic.com/firebasejs/10.13.0/firebase-firestore.js";
import {
  getAuth, signInWithEmailAndPassword, onAuthStateChanged, signOut, createUserWithEmailAndPassword
} from "https://www.gstatic.com/firebasejs/10.13.0/firebase-auth.js";

const firebaseConfig = {
  apiKey: "SUA_API_KEY",
  authDomain: "SEU_DOMINIO.firebaseapp.com",
  projectId: "SEU_PROJETO",
  storageBucket: "SEU_BUCKET.appspot.com",
  messagingSenderId: "SEU_SENDER_ID",
  appId: "SUA_APP_ID"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
const auth = getAuth(app);

// =======================================
// VARIÁVEIS GLOBAIS
// =======================================
let colaboradores = [];
let isAdmin = false;

// =======================================
// LOGIN / LOGOUT
// =======================================
const loginForm = document.getElementById('loginForm');
loginForm.addEventListener('submit', async (e) => {
  e.preventDefault();
  const email = loginEmail.value;
  const senha = loginSenha.value;
  try {
    await signInWithEmailAndPassword(auth, email, senha);
  } catch (error) {
    alert('Erro ao fazer login: ' + error.message);
  }
});

document.getElementById('btnLogout').addEventListener('click', async () => {
  await signOut(auth);
});

onAuthStateChanged(auth, (user) => {
  if (user) {
    document.getElementById('loginContainer').style.display = 'none';
    document.getElementById('mainContainer').style.display = 'block';
    carregarColaboradores();
  } else {
    document.getElementById('loginContainer').style.display = 'block';
    document.getElementById('mainContainer').style.display = 'none';
  }
});

// =======================================
// CARREGAR COLABORADORES DO FIRESTORE
// =======================================
async function carregarColaboradores() {
  const snapshot = await getDocs(collection(db, 'colaboradores'));
  colaboradores = [];
  snapshot.forEach(doc => colaboradores.push({ id: doc.id, ...doc.data() }));
  renderColaboradores();
  atualizarSelectColab();
}

// =======================================
// RENDERIZAR TABELA DE COLABORADORES (AGORA EM ORDEM ALFABÉTICA)
// =======================================
function renderColaboradores() {
  const filtro = filtroColaboradores.value.toLowerCase();
  const tbody = document.getElementById('tbodyColaboradores');
  tbody.innerHTML = '';

  colaboradores
    .slice()
    .sort((a, b) => (a.nome || '').localeCompare(b.nome || '', 'pt-BR')) // 🔹 Ordena por nome
    .filter(c =>
      (c.nome || '').toLowerCase().includes(filtro) ||
      (c.cargo || '').toLowerCase().includes(filtro) ||
      (c.matricula || '').toLowerCase().includes(filtro) ||
      (c.email || '').toLowerCase().includes(filtro)
    )
    .forEach((c, i) => {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${c.matricula || ''}</td>
        <td>${c.nome || ''}</td>
        <td>${c.email || ''}</td>
        <td>${c.cargo || ''}</td>
        <td>
          <button class="btnEditar" data-index="${i}">Editar</button>
          <button class="btnExcluir" data-index="${i}">Excluir</button>
        </td>
      `;
      tbody.appendChild(tr);
    });

  document.querySelectorAll('.btnEditar').forEach(btn =>
    btn.addEventListener('click', e => editarColaborador(e.target.dataset.index))
  );
  document.querySelectorAll('.btnExcluir').forEach(btn =>
    btn.addEventListener('click', e => excluirColaborador(e.target.dataset.index))
  );
}

// =======================================
// ATUALIZAR SELECT DE COLABORADORES (TAMBÉM EM ORDEM ALFABÉTICA)
// =======================================
function atualizarSelectColab() {
  const select = document.getElementById('colabSelect');
  if (!select) return;
  select.innerHTML = '';

  const colaboradoresOrdenados = colaboradores
    .slice()
    .sort((a, b) => (a.nome || '').localeCompare(b.nome || '', 'pt-BR')); // 🔹 Ordena

  colaboradoresOrdenados.forEach(c => {
    const opt = document.createElement('option');
    opt.value = c.matricula;
    opt.textContent = c.nome;
    select.appendChild(opt);
  });
}

// =======================================
// CRUD DE COLABORADORES
// =======================================
async function salvarColaborador() {
  const nome = document.getElementById('colabNome').value;
  const email = document.getElementById('colabEmail').value;
  const cargo = document.getElementById('colabCargo').value;
  const matricula = document.getElementById('colabMatricula').value;

  if (!nome || !email) return alert('Preencha todos os campos!');

  await addDoc(collection(db, 'colaboradores'), {
    nome, email, cargo, matricula
  });

  document.getElementById('formColaborador').reset();
  carregarColaboradores();
}

async function excluirColaborador(index) {
  if (!confirm('Deseja realmente excluir este colaborador?')) return;
  const colab = colaboradores[index];
  await deleteDoc(doc(db, 'colaboradores', colab.id));
  carregarColaboradores();
}

async function editarColaborador(index) {
  const colab = colaboradores[index];
  document.getElementById('colabNome').value = colab.nome;
  document.getElementById('colabEmail').value = colab.email;
  document.getElementById('colabCargo').value = colab.cargo;
  document.getElementById('colabMatricula').value = colab.matricula;
}

// =======================================
// BOTÕES E EVENTOS
// =======================================
document.getElementById('btnSalvarColab').addEventListener('click', salvarColaborador);
document.getElementById('filtroColaboradores').addEventListener('input', renderColaboradores);

// =======================================
// GERENCIAMENTO DE ACESSOS (INÍCIO)
// =======================================
const gerenciarAcessosBtn = document.getElementById('gerenciarAcessosBtn');
gerenciarAcessosBtn.onclick = () => {
  if (!isAdmin) return alert('Acesso restrito a administradores.');
  // Aqui virá a lógica de gerenciamento de acessos...
};

// =======================================
// FIM DO ARQUIVO
// =======================================
