<!doctype html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Controle de Ponto</title>

<style>
:root {
  --accent:#2196F3;
  --dark:#111;
}
*{box-sizing:border-box;margin:0;padding:0;font-family:Inter,Arial;}
body{
  background:#f5f5f5;
  padding:20px;
}
h1{
  text-align:center;
  margin-bottom:20px;
  color:var(--dark);
}
button{
  background:var(--accent);
  color:#fff;
  border:none;
  padding:10px 15px;
  border-radius:8px;
  cursor:pointer;
  margin:5px;
  transition:.2s;
}
button:hover{background:#1976D2;}
table{
  width:100%;
  border-collapse:collapse;
  margin-top:15px;
  background:#fff;
}
th,td{
  border:1px solid #ddd;
  padding:8px;
  text-align:center;
}
th{background:#2196F3;color:#fff;}
input,select{
  padding:8px;
  margin:5px;
  border:1px solid #ccc;
  border-radius:6px;
}
.container{
  background:#fff;
  border-radius:12px;
  padding:20px;
  box-shadow:0 0 10px rgba(0,0,0,.1);
  max-width:900px;
  margin:auto;
}
</style>

<!-- Bibliotecas -->
<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>

<body>
  <div class="container">
    <h1>Cadastro de Colaboradores e Controle de Ponto</h1>

    <div style="text-align:center;">
      <input id="idColab" placeholder="ID do colaborador">
      <input id="nomeColab" placeholder="Nome do colaborador">
      <button onclick="cadastrar()">Cadastrar</button>
      <button id="baixarBtn">Baixar Excel</button>
    </div>

    <h3 style="margin-top:20px;">Colaboradores</h3>
    <table id="tabelaColab">
      <thead>
        <tr><th>ID</th><th>Nome</th><th>Ações</th></tr>
      </thead>
      <tbody></tbody>
    </table>

    <h3 style="margin-top:20px;">Registros de Ponto</h3>
    <table id="tabelaPonto">
      <thead>
        <tr><th>ID</th><th>Nome</th><th>Data</th><th>Hora</th><th>Tipo</th></tr>
      </thead>
      <tbody></tbody>
    </table>

    <h3 style="margin-top:20px;text-align:center;">Gráfico de Horas Trabalhadas</h3>
    <canvas id="graficoHoras" style="max-width:800px;margin:0 auto;display:block;"></canvas>
  </div>

<script>
let colaboradores = [];
let pontos = [];

/* ========== CADASTRO ========== */
function cadastrar(){
  const id = document.getElementById('idColab').value.trim();
  const nome = document.getElementById('nomeColab').value.trim();
  if(!id || !nome) return alert('Preencha ID e Nome');

  if(colaboradores.find(c=>c.id===id)){
    alert('ID já cadastrado!');
    return;
  }
  colaboradores.push({id,nome});
  atualizarTabelaColab();
  document.getElementById('idColab').value='';
  document.getElementById('nomeColab').value='';
}

/* ========== TABELAS ========== */
function atualizarTabelaColab(){
  const tbody=document.querySelector('#tabelaColab tbody');
  tbody.innerHTML='';
  colaboradores.forEach(c=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${c.id}</td>
      <td>${c.nome}</td>
      <td>
        <button onclick="baterEntrada('${c.id}')">Entrada</button>
        <button onclick="baterSaida('${c.id}')">Saída</button>
      </td>`;
    tbody.appendChild(tr);
  });
}

function atualizarTabelaPonto(){
  const tbody=document.querySelector('#tabelaPonto tbody');
  tbody.innerHTML='';
  pontos.forEach(p=>{
    const tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${p.idColab}</td>
      <td>${p.nome}</td>
      <td>${p.data}</td>
      <td>${p.hora}</td>
      <td>${p.tipo}</td>`;
    tbody.appendChild(tr);
  });
}

/* ========== REGISTRO DE PONTO ========== */
function salvarPonto(tipo,idColab,nome){
  const agora=new Date();
  const data=agora.toLocaleDateString('pt-BR');
  const hora=agora.toLocaleTimeString('pt-BR');
  pontos.push({
    idColab,nome,data,hora,tipo,horarioISO:agora.toISOString()
  });
  atualizarTabelaPonto();
  atualizarGrafico();
}

function baterEntrada(id){
  const colab=colaboradores.find(c=>c.id===id);
  if(!colab)return alert('Colaborador não encontrado');
  salvarPonto('Entrada',colab.id,colab.nome);
}
function baterSaida(id){
  const colab=colaboradores.find(c=>c.id===id);
  if(!colab)return alert('Colaborador não encontrado');
  salvarPonto('Saída',colab.id,colab.nome);
}

/* ========== GRÁFICO ========== */
function atualizarGrafico(){
  const ctx=document.getElementById('graficoHoras').getContext('2d');
  const totais={};

  pontos.forEach(p=>{
    if(!totais[p.nome])totais[p.nome]={entrada:null,total:0};
    const hora=new Date(p.horarioISO);
    if(p.tipo==='Entrada'){
      totais[p.nome].entrada=hora;
    }else if(p.tipo==='Saída'&&totais[p.nome].entrada){
      const diff=(hora-totais[p.nome].entrada)/3600000;
      totais[p.nome].total+=diff;
      totais[p.nome].entrada=null;
    }
  });

  const nomes=Object.keys(totais);
  const horas=nomes.map(n=>totais[n].total.toFixed(2));

  if(window.graficoHoras)window.graficoHoras.destroy();

  window.graficoHoras=new Chart(ctx,{
    type:'bar',
    data:{
      labels:nomes,
      datasets:[{
        label:'Total de Horas Trabalhadas',
        data:horas,
        backgroundColor:'#2196F3'
      }]
    },
    options:{
      scales:{
        y:{beginAtZero:true,title:{display:true,text:'Horas'}},
        x:{title:{display:true,text:'Colaboradores'}}
      },
      plugins:{legend:{display:false}}
    }
  });
}

/* ========== GERAR EXCEL ========== */
document.getElementById('baixarBtn').onclick=()=>{
  const wb=XLSX.utils.book_new();

  const entradas=[['#','ID Colab','Nome','Data','Hora']];
  pontos.filter(p=>p.tipo==='Entrada').forEach((p,i)=>
    entradas.push([i+1,p.idColab,p.nome,p.data,p.hora])
  );
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(entradas),'Entradas');

  const saidas=[['#','ID Colab','Nome','Data','Hora']];
  pontos.filter(p=>p.tipo==='Saída').forEach((p,i)=>
    saidas.push([i+1,p.idColab,p.nome,p.data,p.hora])
  );
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(saidas),'Saídas');

  const horasSheet=[['Funcionário','Data','Horas Trabalhadas']];
  let dados={},totaisColab={},totalGeral=0;

  pontos.forEach(p=>{
    if(!dados[p.nome])dados[p.nome]={};
    if(!dados[p.nome][p.data])dados[p.nome][p.data]=[];
    dados[p.nome][p.data].push(p);
  });

  Object.keys(dados).forEach(nome=>{
    let totalColab=0;
    Object.keys(dados[nome]).forEach(data=>{
      let reg=dados[nome][data].sort((a,b)=>new Date(a.horarioISO)-new Date(b.horarioISO));
      let entrada=null,total=0;
      reg.forEach(r=>{
        const hora=new Date(r.horarioISO);
        if(r.tipo==='Entrada')entrada=hora;
        if(r.tipo==='Saída'&&entrada){
          total+=(hora-entrada)/3600000;
          entrada=null;
        }
      });
      totalColab+=total;
      horasSheet.push([nome,data,`${total.toFixed(2)} h`]);
    });
    horasSheet.push([`Total ${nome}`,'',`${totalColab.toFixed(2)} h`]);
    horasSheet.push([]);
    totaisColab[nome]=totalColab;
    totalGeral+=totalColab;
  });

  horasSheet.push(['TOTAL GERAL','',''+totalGeral.toFixed(2)+' h']);
  XLSX.utils.book_append_sheet(wb,XLSX.utils.aoa_to_sheet(horasSheet),'Horas Trabalhadas');

  const grafico=[['Funcionário','Total de Horas']];
  Object.keys(totaisColab).forEach(nome=>{
    grafico.push([nome,totaisColab[nome]]);
  });
  grafico.push([]);
  grafico.push(['OBS:','Para ver o gráfico, selecione os dados e insira gráfico de barras no Excel.']);
  const wsGrafico=XLSX.utils.aoa_to_sheet(grafico);
  XLSX.utils.book_append_sheet(wb,wsGrafico,'Gráfico de Horas');

  XLSX.writeFile(wb,'Pontos.xlsx');
};
</script>
</body>
</html>
