 <!DOCTYPE html>
<html lang="pt-br">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Milho & Espetinho - Cloud Firebase</title>
  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      font-family: 'Segoe UI', Roboto, sans-serif;
    }

    body {
      background: #d0e8d0;
      height: 100vh;
      overflow: hidden;
    }

    .app {
      display: flex;
      height: 100vh;
    }

    /* SIDEBAR */
    .sidebar {
      width: 280px;
      background: #1b4d1b;
      color: white;
      display: flex;
      flex-direction: column;
      transition: width 0.3s;
      box-shadow: 2px 0 10px rgba(0,0,0,0.2);
      z-index: 10;
    }

    .sidebar.collapsed {
      width: 70px;
    }

    .sidebar-header {
      padding: 20px;
      font-size: 1.2rem;
      font-weight: bold;
      display: flex;
      align-items: center;
      justify-content: space-between;
      border-bottom: 1px solid #2e7d32;
    }
    .sidebar-header h2 {
      font-size: 1.2rem;
      white-space: nowrap;
    }
    .toggle-btn {
      background: none;
      border: none;
      color: white;
      font-size: 1.5rem;
      cursor: pointer;
    }

    .menu {
      flex: 1;
      margin-top: 20px;
    }
    .menu-item {
      padding: 15px 20px;
      display: flex;
      align-items: center;
      gap: 12px;
      cursor: pointer;
      transition: 0.2s;
      color: #e0f2e0;
      font-size: 1rem;
    }
    .menu-item:hover {
      background: #2e7d32;
    }
    .menu-item.active {
      background: #2e7d32;
      border-left: 4px solid #ffc107;
    }
    .menu-item span:first-child {
      font-size: 1.3rem;
      width: 28px;
    }
    .menu-item .label {
      white-space: nowrap;
    }
    .sidebar.collapsed .label {
      display: none;
    }
    .sidebar.collapsed .menu-item {
      justify-content: center;
    }
    .sidebar-footer {
      padding: 20px;
      font-size: 0.7rem;
      text-align: center;
      border-top: 1px solid #2e7d32;
    }

    /* MAIN CONTENT */
    .main-content {
      flex: 1;
      overflow-y: auto;
      padding: 24px;
      background: #e8f5e9;
    }

    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      margin-bottom: 30px;
    }
    .card {
      background: white;
      border-radius: 24px;
      padding: 18px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.05);
      border-left: 6px solid #2e7d32;
    }
    .card h3 {
      font-size: 0.8rem;
      color: #1b5e20;
      margin-bottom: 8px;
    }
    .card .valor {
      font-size: 1.6rem;
      font-weight: bold;
    }
    .progress-bar {
      margin-top: 10px;
      background: #e0e0e0;
      border-radius: 20px;
      height: 8px;
    }
    .progress-fill {
      background: #4caf50;
      height: 100%;
      width: 0%;
      border-radius: 20px;
    }
    .form-row {
      display: flex;
      flex-wrap: wrap;
      gap: 15px;
      margin-bottom: 20px;
    }
    .form-group {
      flex: 1;
      min-width: 140px;
    }
    label {
      font-size: 0.7rem;
      font-weight: bold;
      color: #2e5c2e;
      display: block;
      margin-bottom: 5px;
    }
    input, select, button {
      width: 100%;
      padding: 10px 12px;
      border-radius: 20px;
      border: 1px solid #c8e6c9;
      background: white;
    }
    button {
      background: #2e7d32;
      color: white;
      font-weight: bold;
      border: none;
      cursor: pointer;
    }
    button:hover { background: #1b5e20; }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 15px;
      background: white;
      border-radius: 20px;
      overflow: hidden;
    }
    th, td {
      padding: 10px 8px;
      text-align: left;
      border-bottom: 1px solid #d0e8d0;
    }
    th {
      background: #e8f5e9;
      color: #1b5e20;
    }
    .badge-pendente { background: #ff8f00; color: white; padding: 2px 8px; border-radius: 20px; font-size: 0.7rem; }
    .badge-pago { background: #4caf50; color: white; padding: 2px 8px; border-radius: 20px; font-size: 0.7rem; }
    .insumo-box {
      background: #fff8e1;
      border-left: 4px solid #ff8f00;
      padding: 15px;
      border-radius: 20px;
      margin: 15px 0;
    }
    .page {
      display: none;
    }
    .page.active {
      display: block;
    }
    .loading {
      text-align: center;
      padding: 40px;
      color: #2e7d32;
    }
  </style>
</head>
<body>
<div class="app">
  <div class="sidebar" id="sidebar">
    <div class="sidebar-header">
      <h2>🌽 Milho & Espeto</h2>
      <button class="toggle-btn" id="toggleSidebar">☰</button>
    </div>
    <div class="menu">
      <div class="menu-item active" data-page="dashboard"><span>📊</span><span class="label">Dashboard</span></div>
      <div class="menu-item" data-page="lancamentos"><span>📆</span><span class="label">Lançamentos</span></div>
      <div class="menu-item" data-page="extrato"><span>📋</span><span class="label">Extrato & Resumo</span></div>
    </div>
    <div class="sidebar-footer"><span>💚 Dados salvos na nuvem</span></div>
  </div>

  <div class="main-content">
    <div id="dashboard-page" class="page active">
      <h1>📊 Dashboard - Visão Geral</h1>
      <div class="dashboard-grid">
        <div class="card"><h3>💵 CAIXA ESPÉCIE</h3><div class="valor" id="saldoEspecie">R$ 0,00</div></div>
        <div class="card"><h3>📱 CAIXA PIX</h3><div class="valor" id="saldoPix">R$ 0,00</div></div>
        <div class="card"><h3>📝 CONTAS A RECEBER</h3><div class="valor" id="totalReceber">R$ 0,00</div></div>
        <div class="card"><h3>💰 FUNDO TOTAL</h3><div class="valor" id="fundoTotal">R$ 0,00</div></div>
        <div class="card"><h3>💵 TOTAL INVESTIDO</h3><div class="valor" id="totalAportes">R$ 0,00</div></div>
        <div class="card"><h3>🏭 EQUIPAMENTOS: PAGO</h3><div class="valor" id="totalEquipPago">R$ 0,00</div></div>
        <div class="card"><h3>⚠️ EQUIPAMENTOS: A PAGAR</h3><div class="valor" id="totalEquipDevendo">R$ 0,00</div></div>
        <div class="card"><h3>📈 RESULTADO OPERACIONAL</h3><div class="valor" id="resultadoOp">R$ 0,00</div></div>
      </div>
      <div class="card">
        <h3>🎯 PAYBACK (recuperar investimento)</h3>
        <div class="valor" id="paybackPercent">0%</div>
        <div class="progress-bar"><div class="progress-fill" id="paybackFill"></div></div>
        <div id="paybackMsg"></div>
      </div>
      <div class="card" style="margin-top:20px">
        <h3>📈 Gráfico de Lucro Diário</h3>
        <canvas id="lucroChart" width="400" height="200" style="max-height:250px"></canvas>
      </div>
    </div>

    <div id="lancamentos-page" class="page">
      <h1>📆 Lançamentos</h1>
      <!-- APORTES -->
      <div class="card" style="margin-bottom:20px">
        <h3>💰 APORTES (dinheiro do seu bolso)</h3>
        <div class="form-row">
          <div class="form-group"><label>Data</label><input type="date" id="aporteData"></div>
          <div class="form-group"><label>Valor (R$)</label><input type="number" id="aporteValor" step="0.01"></div>
          <div class="form-group"><label>Destino</label><select id="aporteDestino"><option value="especie">Espécie</option><option value="pix">Pix</option></select></div>
          <div class="form-group"><label>Descrição</label><input type="text" id="aporteDesc" placeholder="Ex: Aporte inicial"></div>
          <div class="form-group"><button id="btnAddAporte">➕ Registrar</button></div>
        </div>
        <div id="listaAportes"></div>
      </div>

      <!-- EQUIPAMENTOS -->
      <div class="card" style="margin-bottom:20px">
        <h3>🏭 EQUIPAMENTOS / PARCELAS</h3>
        <div class="form-row">
          <div class="form-group"><label>Descrição</label><input type="text" id="equipDesc" placeholder="Ex: Geladeira"></div>
          <div class="form-group"><label>Valor total (R$)</label><input type="number" id="equipTotal" step="0.01"></div>
          <div class="form-group"><label>Valor já pago (R$)</label><input type="number" id="equipPago" step="0.01" value="0"></div>
          <div class="form-group"><label>Forma de pagamento</label><select id="equipPagamento"><option value="especie">Espécie</option><option value="pix">Pix</option></select></div>
          <div class="form-group"><button id="btnAddEquip">➕ Adicionar</button></div>
        </div>
        <div id="listaEquipamentos"></div>
      </div>

      <!-- CUSTOS FIXOS -->
      <div class="card" style="margin-bottom:20px">
        <h3>🏠 CUSTOS FIXOS (aluguel, água, luz, etc)</h3>
        <p style="font-size:0.8rem">Estes custos <strong>NÃO</strong> saem do caixa das vendas, saem do seu aporte.</p>
        <div class="form-row">
          <div class="form-group"><label>Data</label><input type="date" id="custoFixoData"></div>
          <div class="form-group"><label>Descrição</label><input type="text" id="custoFixoDesc" placeholder="Ex: Aluguel"></div>
          <div class="form-group"><label>Valor (R$)</label><input type="number" id="custoFixoValor" step="0.01"></div>
          <div class="form-group"><label>Pago com</label><select id="custoFixoOrigem"><option value="aporte">Aporte (do seu bolso)</option><option value="caixa">Caixa do negócio</option></select></div>
          <div class="form-group"><button id="btnAddCustoFixo">➕ Adicionar</button></div>
        </div>
        <div id="listaCustosFixos"></div>
      </div>

      <!-- BALANÇO DIÁRIO -->
      <div class="card" style="margin-bottom:20px">
        <h3>📆 BALANÇO DIÁRIO (Vendas e Insumos)</h3>
        <div class="form-row"><div class="form-group"><label>Data</label><input type="date" id="balancoData"></div></div>
        <div style="background:#e3f2fd; border-radius:16px; padding:15px; margin:10px 0">
          <h4>💰 FATURAMENTO DO DIA</h4>
          <div class="form-row">
            <div class="form-group"><label>Espécie (R$)</label><input type="number" id="vendaEspecie" step="0.01" value="0"></div>
            <div class="form-group"><label>Pix (R$)</label><input type="number" id="vendaPix" step="0.01" value="0"></div>
            <div class="form-group"><label>Fiado (R$)</label><input type="number" id="vendaFiado" step="0.01" value="0"></div>
          </div>
        </div>
        <div class="insumo-box">
          <h4>🥩 GASTOS COM INSUMOS (milho, carvão, carne)</h4>
          <div class="form-row">
            <div class="form-group"><label>Valor total (R$)</label><input type="number" id="insumoValor" step="0.01" value="0"></div>
            <div class="form-group"><label>Pagamento</label><select id="insumoPagamento"><option value="especie">Espécie</option><option value="pix">Pix</option></select></div>
            <div class="form-group"><label>Descrição</label><input type="text" id="insumoDesc" placeholder="Ex: carvão, carne"></div>
          </div>
        </div>
        <button id="btnFinalizarDia" style="background:#43a047">✅ FINALIZAR DIA</button>
      </div>
    </div>

    <div id="extrato-page" class="page">
      <h1>📋 Extrato e Resumo</h1>
      <div class="card">
        <div class="form-row">
          <div class="form-group"><label>Data inicial</label><input type="date" id="filtroInicio"></div>
          <div class="form-group"><label>Data final</label><input type="date" id="filtroFim"></div>
          <div class="form-group"><button id="btnFiltrar">🔍 Filtrar</button></div>
          <div class="form-group"><button id="btnLimparFiltro">🗑️ Limpar</button></div>
        </div>
        <div style="overflow-x:auto">
          <table id="tabelaExtrato">
            <thead><tr><th>Data</th><th>Faturamento</th><th>Insumos</th><th>Lucro Bruto</th><th>Custos Fixos (do caixa)</th><th>Lucro Líquido Dia</th><th>Ações</th></tr></thead>
            <tbody id="corpoExtrato"></tbody>
          </table>
        </div>
        <div style="margin-top:20px; text-align:right"><strong>Total Lucro no período: <span id="lucroPeriodo">R$ 0,00</span></strong></div>
        <button id="btnReset" style="background:#a5d6a7; color:#1c441c; margin-top:20px">🔄 Resetar todos os dados</button>
      </div>
    </div>
  </div>
</div>

<script>
  // FIREBASE CONFIG
  const firebaseConfig = {
    apiKey: "AIzaSyC0sfOc-_oRgT6QdXhCcFdANWUhTMyUP4w",
    authDomain: "controle-de-vendas-e819f.firebaseapp.com",
    projectId: "controle-de-vendas-e819f",
    storageBucket: "controle-de-vendas-e819f.firebasestorage.app",
    messagingSenderId: "1057991353567",
    appId: "1:1057991353567:web:5466718c454caa3972f56a",
    measurementId: "G-2D1YJ4P28F"
  };

  // Inicializar Firebase
  firebase.initializeApp(firebaseConfig);
  const db = firebase.firestore();

  // Coleções
  const aportesRef = db.collection("aportes");
  const equipamentosRef = db.collection("equipamentos");
  const custosFixosRef = db.collection("custosFixos");
  const balancosRef = db.collection("balancos");

  // Estado local (cache para cálculos rápidos)
  let aportes = [];
  let equipamentos = [];
  let custosFixos = [];
  let balancos = [];
  let saldoEspecie = 0;
  let saldoPix = 0;

  let lucroChart = null;

  // ========== FUNÇÕES DE CARREGAMENTO ==========
  async function carregarDados() {
    // Aportes
    const aportesSnap = await aportesRef.orderBy("data", "asc").get();
    aportes = aportesSnap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    // Equipamentos
    const equipSnap = await equipamentosRef.get();
    equipamentos = equipSnap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    // Custos Fixos
    const cfSnap = await custosFixosRef.orderBy("data", "asc").get();
    custosFixos = cfSnap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    // Balanços
    const balSnap = await balancosRef.orderBy("data", "asc").get();
    balancos = balSnap.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    
    recalcularSaldos();
    atualizarTudo();
  }

  function recalcularSaldos() {
    let novaEspecie = 0, novoPix = 0;
    aportes.forEach(a => {
      if(a.destino === "especie") novaEspecie += a.valor;
      else novoPix += a.valor;
    });
    balancos.forEach(b => {
      novaEspecie += (b.fatEspecie || 0);
      novoPix += (b.fatPix || 0);
      if(b.insumoPagamento === "especie") novaEspecie -= b.insumoValor;
      else novoPix -= b.insumoValor;
    });
    custosFixos.forEach(cf => {
      if(cf.origem === "caixa") {
        if(cf.pagamento === "especie") novaEspecie -= cf.valor;
        else novoPix -= cf.valor;
      }
    });
    saldoEspecie = novaEspecie;
    saldoPix = novoPix;
  }

  function getTotais() {
    let totalAportes = aportes.reduce((s,a)=>s+a.valor,0);
    let totalEquipPago = equipamentos.reduce((s,e)=>s+(e.pago||0),0);
    let totalEquipDevendo = equipamentos.reduce((s,e)=>s+(e.restante||0),0);
    let totalReceber = balancos.reduce((s,b)=>s+(b.fatFiado||0),0);
    let totalVendas = balancos.reduce((s,b)=>s+(b.fatTotal||0),0);
    let totalInsumos = balancos.reduce((s,b)=>s+(b.insumoValor||0),0);
    let totalCustosFixosDoCaixa = custosFixos.filter(c=>c.origem==="caixa").reduce((s,c)=>s+c.valor,0);
    let resultadoOp = totalVendas - totalInsumos - totalCustosFixosDoCaixa;
    let fundoTotal = saldoEspecie + saldoPix + totalReceber;
    let paybackPercent = totalAportes > 0 ? (resultadoOp / totalAportes) * 100 : 0;
    if(paybackPercent > 100) paybackPercent = 100;
    if(paybackPercent < 0) paybackPercent = 0;
    return { totalAportes, totalEquipPago, totalEquipDevendo, totalReceber, resultadoOp, fundoTotal, paybackPercent, saldoEspecie, saldoPix };
  }

  function atualizarDashboard() {
    const t = getTotais();
    document.getElementById("saldoEspecie").innerHTML = `R$ ${t.saldoEspecie.toFixed(2)}`;
    document.getElementById("saldoPix").innerHTML = `R$ ${t.saldoPix.toFixed(2)}`;
    document.getElementById("totalReceber").innerHTML = `R$ ${t.totalReceber.toFixed(2)}`;
    document.getElementById("fundoTotal").innerHTML = `R$ ${t.fundoTotal.toFixed(2)}`;
    document.getElementById("totalAportes").innerHTML = `R$ ${t.totalAportes.toFixed(2)}`;
    document.getElementById("totalEquipPago").innerHTML = `R$ ${t.totalEquipPago.toFixed(2)}`;
    document.getElementById("totalEquipDevendo").innerHTML = `R$ ${t.totalEquipDevendo.toFixed(2)}`;
    document.getElementById("resultadoOp").innerHTML = `R$ ${t.resultadoOp.toFixed(2)}`;
    document.getElementById("paybackPercent").innerHTML = `${t.paybackPercent.toFixed(1)}%`;
    document.getElementById("paybackFill").style.width = `${t.paybackPercent}%`;
    const msg = t.resultadoOp >= t.totalAportes ? "🎉 Payback atingido! Lucro real." : `📈 Faltam R$ ${(t.totalAportes - t.resultadoOp).toFixed(2)} para recuperar.`;
    document.getElementById("paybackMsg").innerHTML = msg;
    atualizarGrafico();
  }

  function atualizarGrafico() {
    const ctx = document.getElementById("lucroChart").getContext('2d');
    const ordenados = [...balancos].sort((a,b)=>a.data.localeCompare(b.data));
    const labels = ordenados.map(b=>b.data);
    const lucros = ordenados.map(b=>b.lucroBruto);
    if(lucroChart) lucroChart.destroy();
    lucroChart = new Chart(ctx, {
      type: 'line',
      data: { labels, datasets: [{ label: 'Lucro Bruto do dia (R$)', data: lucros, borderColor: '#2e7d32', fill: true }] }
    });
  }

  function renderizarAportes() {
    const div = document.getElementById("listaAportes");
    if(!aportes.length) { div.innerHTML = "<p>Nenhum aporte.</p>"; return; }
    let html = "<div style='display:flex;flex-wrap:wrap;gap:8px'>";
    aportes.forEach(a => {
      html += `<div style='background:#e8f5e9;border-radius:16px;padding:6px 12px'>${a.data} - R$ ${a.valor.toFixed(2)} - ${a.descricao} <button onclick="removerAporte('${a.id}')" style='background:#ffcdd2;padding:2px 8px'>✖️</button></div>`;
    });
    html += "</div>";
    div.innerHTML = html;
  }

  function renderizarEquipamentos() {
    const div = document.getElementById("listaEquipamentos");
    if(!equipamentos.length) { div.innerHTML = "<p>Nenhum equipamento.</p>"; return; }
    let html = "<table><tr><th>Item</th><th>Total</th><th>Já pago</th><th>A pagar</th><th>Ações</th></tr>";
    equipamentos.forEach(e => {
      html += `<tr><td>${e.descricao}</td><td>R$ ${e.total.toFixed(2)}</td><td>R$ ${e.pago.toFixed(2)}</td><td>R$ ${e.restante.toFixed(2)}</td>
               <td><button onclick="removerEquip('${e.id}')">🗑️</button></td></tr>`;
    });
    html += "</table>";
    div.innerHTML = html;
  }

  function renderizarCustosFixos() {
    const div = document.getElementById("listaCustosFixos");
    if(!custosFixos.length) { div.innerHTML = "<p>Nenhum custo fixo.</p>"; return; }
    let html = "<table><tr><th>Data</th><th>Descrição</th><th>Valor</th><th>Origem</th><th>Ações</th></tr>";
    custosFixos.forEach(c => {
      html += `<tr><td>${c.data}</td><td>${c.descricao}</td><td>R$ ${c.valor.toFixed(2)}</td><td>${c.origem === 'aporte' ? 'Aporte' : 'Caixa'}</td>
               <td><button onclick="removerCustoFixo('${c.id}')">🗑️</button></td></tr>`;
    });
    html += "</table>";
    div.innerHTML = html;
  }

  function renderizarExtrato(filtroInicio = "", filtroFim = "") {
    let lista = [...balancos];
    if(filtroInicio) lista = lista.filter(b=>b.data >= filtroInicio);
    if(filtroFim) lista = lista.filter(b=>b.data <= filtroFim);
    lista.sort((a,b)=>a.data.localeCompare(b.data));
    const tbody = document.getElementById("corpoExtrato");
    tbody.innerHTML = "";
    let lucroTotal = 0;
    lista.forEach(b => {
      const row = tbody.insertRow();
      row.insertCell(0).innerText = b.data;
      row.insertCell(1).innerText = `R$ ${b.fatTotal.toFixed(2)}`;
      row.insertCell(2).innerText = `R$ ${b.insumoValor.toFixed(2)}`;
      row.insertCell(3).innerText = `R$ ${b.lucroBruto.toFixed(2)}`;
      let custosFixosDia = custosFixos.filter(c=>c.origem==="caixa" && c.data === b.data).reduce((s,c)=>s+c.valor,0);
      row.insertCell(4).innerText = `R$ ${custosFixosDia.toFixed(2)}`;
      const lucroLiq = b.lucroBruto - custosFixosDia;
      row.insertCell(5).innerText = `R$ ${lucroLiq.toFixed(2)}`;
      if(lucroLiq < 0) row.cells[5].style.color = "#c62828";
      else row.cells[5].style.color = "#2e7d32";
      lucroTotal += lucroLiq;
      const btnCell = row.insertCell(6);
      const btn = document.createElement("button");
      btn.innerText = "🗑️";
      btn.style.background = "#ffcdd2";
      btn.onclick = () => { if(confirm("Remover este dia?")) removerBalancos(b.id); };
      btnCell.appendChild(btn);
    });
    document.getElementById("lucroPeriodo").innerHTML = `R$ ${lucroTotal.toFixed(2)}`;
  }

  // OPERAÇÕES NO FIREBASE
  async function adicionarAporte() {
    const data = document.getElementById("aporteData").value;
    const valor = parseFloat(document.getElementById("aporteValor").value);
    const destino = document.getElementById("aporteDestino").value;
    const descricao = document.getElementById("aporteDesc").value.trim() || "Aporte";
    if(!data || isNaN(valor) || valor<=0) { alert("Data e valor válidos"); return; }
    const docRef = await aportesRef.add({ data, valor, destino, descricao });
    aportes.push({ id: docRef.id, data, valor, destino, descricao });
    recalcularSaldos();
    atualizarTudo();
    document.getElementById("aporteValor").value = "";
    document.getElementById("aporteDesc").value = "";
  }

  async function adicionarEquipamento() {
    const descricao = document.getElementById("equipDesc").value.trim();
    const total = parseFloat(document.getElementById("equipTotal").value);
    let pago = parseFloat(document.getElementById("equipPago").value) || 0;
    const pagamento = document.getElementById("equipPagamento").value;
    if(!descricao || isNaN(total) || total<=0) { alert("Preencha corretamente"); return; }
    if(pago > total) pago = total;
    const restante = total - pago;
    const docRef = await equipamentosRef.add({ descricao, total, pago, restante, pagamento });
    equipamentos.push({ id: docRef.id, descricao, total, pago, restante, pagamento });
    atualizarTudo();
    document.getElementById("equipDesc").value = "";
    document.getElementById("equipTotal").value = "";
    document.getElementById("equipPago").value = "";
  }

  async function adicionarCustoFixo() {
    const data = document.getElementById("custoFixoData").value;
    const descricao = document.getElementById("custoFixoDesc").value.trim();
    const valor = parseFloat(document.getElementById("custoFixoValor").value);
    const origem = document.getElementById("custoFixoOrigem").value;
    if(!data || !descricao || isNaN(valor) || valor<=0) { alert("Preencha todos os campos"); return; }
    const docRef = await custosFixosRef.add({ data, descricao, valor, origem, pagamento: "especie" });
    custosFixos.push({ id: docRef.id, data, descricao, valor, origem, pagamento: "especie" });
    recalcularSaldos();
    atualizarTudo();
    document.getElementById("custoFixoDesc").value = "";
    document.getElementById("custoFixoValor").value = "";
  }

  async function finalizarDia() {
    const data = document.getElementById("balancoData").value;
    const fatEspecie = parseFloat(document.getElementById("vendaEspecie").value) || 0;
    const fatPix = parseFloat(document.getElementById("vendaPix").value) || 0;
    const fatFiado = parseFloat(document.getElementById("vendaFiado").value) || 0;
    const fatTotal = fatEspecie + fatPix + fatFiado;
    const insumoValor = parseFloat(document.getElementById("insumoValor").value) || 0;
    const insumoPagamento = document.getElementById("insumoPagamento").value;
    const insumoDesc = document.getElementById("insumoDesc").value;
    const lucroBruto = fatTotal - insumoValor;
    if(!data) { alert("Selecione a data"); return; }
    const existente = balancos.find(b=>b.data === data);
    if(existente) {
      if(!confirm(`Já existe lançamento para ${data}. Substituir?`)) return;
      await balancosRef.doc(existente.id).delete();
      balancos = balancos.filter(b=>b.id !== existente.id);
    }
    const docRef = await balancosRef.add({ data, fatEspecie, fatPix, fatFiado, fatTotal, insumoValor, insumoPagamento, insumoDesc, lucroBruto });
    balancos.push({ id: docRef.id, data, fatEspecie, fatPix, fatFiado, fatTotal, insumoValor, insumoPagamento, insumoDesc, lucroBruto });
    recalcularSaldos();
    atualizarTudo();
    alert(`✅ Dia ${data} registrado! Lucro Bruto: R$ ${lucroBruto.toFixed(2)}`);
    document.getElementById("vendaEspecie").value = "";
    document.getElementById("vendaPix").value = "";
    document.getElementById("vendaFiado").value = "";
    document.getElementById("insumoValor").value = "";
    document.getElementById("insumoDesc").value = "";
  }

  async function removerAporte(id) {
    await aportesRef.doc(id).delete();
    aportes = aportes.filter(a=>a.id !== id);
    recalcularSaldos();
    atualizarTudo();
  }
  async function removerEquip(id) {
    await equipamentosRef.doc(id).delete();
    equipamentos = equipamentos.filter(e=>e.id !== id);
    atualizarTudo();
  }
  async function removerCustoFixo(id) {
    await custosFixosRef.doc(id).delete();
    custosFixos = custosFixos.filter(c=>c.id !== id);
    recalcularSaldos();
    atualizarTudo();
  }
  async function removerBalancos(id) {
    await balancosRef.doc(id).delete();
    balancos = balancos.filter(b=>b.id !== id);
    recalcularSaldos();
    atualizarTudo();
  }

  async function resetarTudo() {
    if(confirm("⚠️ Isso apagará TODOS os dados do Firebase. Continuar?")) {
      const batches = [];
      batches.push(aportesRef.get().then(snap => snap.forEach(d => d.ref.delete())));
      batches.push(equipamentosRef.get().then(snap => snap.forEach(d => d.ref.delete())));
      batches.push(custosFixosRef.get().then(snap => snap.forEach(d => d.ref.delete())));
      batches.push(balancosRef.get().then(snap => snap.forEach(d => d.ref.delete())));
      await Promise.all(batches);
      aportes = []; equipamentos = []; custosFixos = []; balancos = [];
      recalcularSaldos();
      atualizarTudo();
    }
  }

  function atualizarTudo() {
    atualizarDashboard();
    renderizarAportes();
    renderizarEquipamentos();
    renderizarCustosFixos();
    renderizarExtrato(document.getElementById("filtroInicio").value, document.getElementById("filtroFim").value);
  }

  // Navegação e Sidebar
  function initMenu() {
    const menuItems = document.querySelectorAll('.menu-item');
    const pages = document.querySelectorAll('.page');
    menuItems.forEach(item => {
      item.addEventListener('click', () => {
        const pageId = item.getAttribute('data-page') + '-page';
        menuItems.forEach(i => i.classList.remove('active'));
        item.classList.add('active');
        pages.forEach(p => p.classList.remove('active'));
        document.getElementById(pageId).classList.add('active');
      });
    });
  }

  document.getElementById("toggleSidebar").addEventListener("click", () => {
    document.getElementById("sidebar").classList.toggle("collapsed");
  });

  function aplicarFiltro() {
    renderizarExtrato(document.getElementById("filtroInicio").value, document.getElementById("filtroFim").value);
  }
  function limparFiltro() {
    document.getElementById("filtroInicio").value = "";
    document.getElementById("filtroFim").value = "";
    renderizarExtrato("", "");
  }

  window.onload = async () => {
    initMenu();
    const hoje = new Date().toISOString().split('T')[0];
    document.getElementById("aporteData").value = hoje;
    document.getElementById("balancoData").value = hoje;
    document.getElementById("custoFixoData").value = hoje;
    document.getElementById("btnAddAporte").onclick = adicionarAporte;
    document.getElementById("btnAddEquip").onclick = adicionarEquipamento;
    document.getElementById("btnAddCustoFixo").onclick = adicionarCustoFixo;
    document.getElementById("btnFinalizarDia").onclick = finalizarDia;
    document.getElementById("btnReset").onclick = resetarTudo;
    document.getElementById("btnFiltrar").onclick = aplicarFiltro;
    document.getElementById("btnLimparFiltro").onclick = limparFiltro;
    await carregarDados();
  };
</script>
</body>
</html>
