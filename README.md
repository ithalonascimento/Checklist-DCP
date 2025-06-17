<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Checklist de Recebimento</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
  <style>
    * {
      box-sizing: border-box;
    }
    body {
      font-family: 'Roboto', sans-serif;
      padding: 16px;
      background-color: #f0f2f5;
      color: #333;
      max-width: 600px;
      margin: auto;
    }
    h2 {
      background-color: #d62828;
      color: white;
      padding: 16px;
      border-radius: 12px;
      text-align: center;
      font-size: 20px;
    }
    .section-title {
      font-weight: 700;
      font-size: 16px;
      margin-top: 24px;
      background-color: #e9ecef;
      padding: 10px 14px;
      border-radius: 8px;
      border-left: 5px solid #d62828;
    }
    .question {
      background: white;
      padding: 14px;
      border-radius: 12px;
      margin: 16px 0;
      box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }
    .question label {
      margin-bottom: 8px;
      display: block;
      font-weight: 500;
    }
    .options {
      display: flex;
      gap: 8px;
      margin-bottom: 10px;
    }
    .options button {
      flex: 1;
      padding: 10px;
      border: none;
      border-radius: 20px;
      font-weight: bold;
      cursor: pointer;
      transition: 0.3s;
    }
    .sim { background-color: #28a745; color: white; }
    .nao { background-color: #dc3545; color: white; }
    .na { background-color: #6c757d; color: white; }
    input[type="file"], textarea, input[type="text"], input[type="date"] {
      width: 100%;
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
      margin-top: 6px;
      margin-bottom: 12px;
      font-size: 14px;
    }
    textarea {
      resize: vertical;
    }
    button.finalizar {
      padding: 14px;
      width: 100%;
      background-color: #d62828;
      color: white;
      border: none;
      border-radius: 12px;
      font-size: 16px;
      font-weight: bold;
      margin-top: 30px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.2);
      cursor: pointer;
      transition: background-color 0.3s;
    }
    button.finalizar:hover {
      background-color: #a61e1e;
    }
    canvas.signature {
      border: 1px dashed #aaa;
      width: 100%;
      height: 150px;
      border-radius: 8px;
      margin-bottom: 10px;
      background: #fff;
    }
    .question button {
      font-size: 14px;
    }
  </style>
</head>
<body onbeforeunload="return confirm('Tem certeza que deseja atualizar a página? Você pode perder os dados preenchidos.')">
<h2>Recebimento</h2>

  <div class="section-title">Dados da Loja</div>
  <div class="question">
    <label>Número da Loja</label><input type="text" id="numeroLoja">
    <label>Data</label><input type="date" id="data">
    <label>Nome do Gerente</label><input type="text" id="gerente">
    <label>Prontuário</label><input type="text" id="prontuario">
  </div>

  <div class="section-title">Dados do Veículo</div>
  <div class="question">
    <label>Transportadora</label><input type="text" id="transportadora">
    <label>Placa do Veículo</label><input type="text" id="placaVeiculo">
    <label>Placa da Carreta</label><input type="text" id="placaCarreta">
  </div>

  <div class="section-title">Dados do Motorista</div>
  <div class="question">
    <label>Nome</label><input type="text" id="nomeMotorista">
    <label>CPF</label><input type="text" id="cpfMotorista">
    <label>CNH</label><input type="text" id="cnhMotorista">
  </div>

  <div class="section-title">Auditoria Veículo</div>
  <div id="perguntas"></div>

  <div class="section-title">Lacres</div>
  <div id="lacres"></div>

  <div class="section-title">Fotos Obrigatórias</div>
  <div id="fotosObrigatorias"></div>

  <div class="section-title">Observações</div>
  <div class="question">
    <label>Observação</label><textarea id="observacao" rows="4"></textarea>
    <label>Foto da Observação</label><input type="file" id="fotoObservacao" accept="image/*">
  </div>

  <div class="section-title">Assinaturas</div>
  <div class="question">
    <label>Assinatura do Gerente</label>
    <canvas id="assinaturaGerente" class="signature"></canvas>
    <button onclick="limparCanvas('assinaturaGerente')">Limpar Assinatura Gerente</button>
    <label>Assinatura do Motorista</label>
    <canvas id="assinaturaMotorista" class="signature"></canvas>
    <button onclick="limparCanvas('assinaturaMotorista')">Limpar Assinatura Motorista</button>
  </div>

  <button class="finalizar" onclick="gerarPDF()">Finalizar Checklist</button>

  <script>
    const perguntas = [
      "O Caminhão chegou com o Rastreador Ativo?",
      "Documentos do Motorista conferem com o COE?",
      "Descarregadores possuem ficha cadastral?",
      "Todos os Lacres estão apertados?",
      "Carroceria, assoalho tem vestígios de abertura?",
      "Itens de alto risco conferidos pelo COE?",
      "Lista do Gerenciamento de Risco conferida?"
    ];
    const lacres = [
      "Porta Traseira - Tranca do lado direito",
      "Porta Traseira - Tranca do lado esquerdo",
      "Pino da Dobradiça - lado direito",
      "Pino da Dobradiça - lado esquerdo",
      "Tranca/Adalbra central",
      "Porta Lateral - Tranca Externa",
      "Porta Lateral - Tranca Interna",
      "Porta de Refrigeração"
    ];
    const fotosObrigatorias = [
      "Foto da CNH do motorista",
      "Foto da CRVL do veículo",
      "Foto da Porta Traseira pegando a placa e os lacres",
      "Foto da Porta Lateral",
      "Foto do pino direito com lacre",
      "Foto do pino esquerdo com lacre",
      "Foto da Adalbra com lacre",
      "Fotos laterais externas do baú - lateral direita e lateral esquerda",
      "Foto visualizando toda a carga e o fundo do baú",
      "Foto de cima do baú",
      "Foto do baú e do teto do veículo após a descarga",
      "Foto das partes internas do veículo",
      "Foto do rastreador ligado",
      "Foto do alto risco com saco de ráfia"
    ];
    const respostas = {};
    const perguntasDiv = document.getElementById("perguntas");
    perguntas.forEach((p, i) => {
      const id = `pergunta_${i}`;
      perguntasDiv.innerHTML += `
        <div class="question">
          <label>${p}</label>
          <div class="options">
            <button type="button" onclick="responder('${id}', 'Sim', this)" class="sim">Sim</button>
            <button type="button" onclick="responder('${id}', 'Não', this)" class="nao">Não</button>
            <button type="button" onclick="responder('${id}', 'N/A', this)" class="na">N/A</button>
          </div>
          <input type="file" accept="image/*" id="${id}_foto">
        </div>`;
    });
    const lacresDiv = document.getElementById("lacres");
    lacres.forEach((p, i) => {
      const id = `lacre_${i}`;
      lacresDiv.innerHTML += `
        <div class="question">
          <label>${p}</label>
          <input type="text" placeholder="Numeração do Lacre" id="${id}_numero">
          <input type="file" accept="image/*" id="${id}_foto">
        </div>`;
    });
    const fotosDiv = document.getElementById("fotosObrigatorias");
    fotosObrigatorias.forEach((p, i) => {
      const id = `fotoObrigatoria_${i}`;
      fotosDiv.innerHTML += `
        <div class="question">
          <label>${p}</label>
          <input type="file" accept="image/*" id="${id}">
        </div>`;
    });

    function responder(id, valor, el) {
      respostas[id] = valor;
      const buttons = el.parentElement.querySelectorAll('button');
      buttons.forEach(btn => btn.style.opacity = '0.4');
      el.style.opacity = '1';
    }

    async function gerarPDF() {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF();
      let y = 10;

      function novaPagina() {
        doc.addPage();
        y = 10;
      }

      function verificarEspaco(altura) {
        if (y + altura > 280) novaPagina();
      }

      doc.setFontSize(12);
      const dados = [
        `Loja: ${document.getElementById('numeroLoja').value}`,
        `Gerente: ${document.getElementById('gerente').value} - Prontuário: ${document.getElementById('prontuario').value}`,
        `Data: ${document.getElementById('data').value}`,
        `Transportadora: ${document.getElementById('transportadora').value}`,
        `Placa Veículo: ${document.getElementById('placaVeiculo').value} | Carreta: ${document.getElementById('placaCarreta').value}`,
        `Motorista: ${document.getElementById('nomeMotorista').value} | CPF: ${document.getElementById('cpfMotorista').value} | CNH: ${document.getElementById('cnhMotorista').value}`
      ];
      for (const linha of dados) {
        verificarEspaco(10);
        doc.text(linha, 10, y);
        y += 8;
      }

      for (let i = 0; i < perguntas.length; i++) {
        const id = `pergunta_${i}`;
        verificarEspaco(10);
        doc.text(`${perguntas[i]}: ${respostas[id] || 'Não respondido'}`, 10, y);
        y += 8;

        const inputFoto = document.getElementById(`${id}_foto`);
        if (inputFoto.files[0]) {
          const imgData = await readFileAsDataURL(inputFoto.files[0]);
          verificarEspaco(40);
          doc.addImage(imgData, 'JPEG', 10, y, 50, 30);
          y += 35;
        }
      }

      for (let i = 0; i < lacres.length; i++) {
        const id = `lacre_${i}`;
        const numero = document.getElementById(`${id}_numero`).value || 'Não informado';
        verificarEspaco(10);
        doc.text(`${lacres[i]}: ${numero}`, 10, y);
        y += 8;

        const inputFoto = document.getElementById(`${id}_foto`);
        if (inputFoto.files[0]) {
          const imgData = await readFileAsDataURL(inputFoto.files[0]);
          verificarEspaco(40);
          doc.addImage(imgData, 'JPEG', 10, y, 50, 30);
          y += 35;
        }
      }

      for (let i = 0; i < fotosObrigatorias.length; i++) {
        const id = `fotoObrigatoria_${i}`;
        const inputFoto = document.getElementById(id);
        if (inputFoto.files[0]) {
          verificarEspaco(40);
          doc.text(fotosObrigatorias[i], 10, y);
          y += 8;
          const imgData = await readFileAsDataURL(inputFoto.files[0]);
          doc.addImage(imgData, 'JPEG', 10, y, 50, 30);
          y += 35;
        }
      }

      const obs = document.getElementById("observacao").value;
      if (obs) {
        verificarEspaco(20);
        doc.text(`Observação: ${obs}`, 10, y);
        y += 8;
      }
      const obsFoto = document.getElementById("fotoObservacao").files[0];
      if (obsFoto) {
        const imgData = await readFileAsDataURL(obsFoto);
        verificarEspaco(40);
        doc.addImage(imgData, 'JPEG', 10, y, 50, 30);
        y += 35;
      }

      verificarEspaco(50);
      doc.text("Assinatura do Gerente:", 10, y);
      const gerenteCanvas = document.getElementById("assinaturaGerente");
      const gerenteImg = gerenteCanvas.toDataURL("image/png");
      doc.addImage(gerenteImg, "PNG", 10, y + 5, 80, 30);
      y += 40;

      verificarEspaco(50);
      doc.text("Assinatura do Motorista:", 10, y);
      const motoristaCanvas = document.getElementById("assinaturaMotorista");
      const motoristaImg = motoristaCanvas.toDataURL("image/png");
      doc.addImage(motoristaImg, "PNG", 10, y + 5, 80, 30);

      doc.save("checklist_recebimento.pdf");
      alert("PDF baixado com sucesso!");
    }

    function readFileAsDataURL(file) {
      return new Promise(resolve => {
        const reader = new FileReader();
        reader.onload = e => resolve(e.target.result);
        reader.readAsDataURL(file);
      });
    }

    function limparCanvas(id) {
      const canvas = document.getElementById(id);
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }
function configurarCanvasAssinatura(id) {
  const canvas = document.getElementById(id);
  const ctx = canvas.getContext("2d");
  let desenhando = false;

  function desenhar(x, y) {
    ctx.lineWidth = 2;
    ctx.lineCap = "round";
    ctx.strokeStyle = "black";
    ctx.lineTo(x, y);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(x, y);
  }

  function getPosicao(event) {
    const rect = canvas.getBoundingClientRect();
    if (event.touches) {
      return {
        x: event.touches[0].clientX - rect.left,
        y: event.touches[0].clientY - rect.top
      };
    } else {
      return {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top
      };
    }
  }

  // Mouse
  canvas.addEventListener("mousedown", () => desenhando = true);
  canvas.addEventListener("mouseup", () => {
    desenhando = false;
    ctx.beginPath();
  });
  canvas.addEventListener("mouseout", () => {
    desenhando = false;
    ctx.beginPath();
  });
  canvas.addEventListener("mousemove", e => {
    if (!desenhando) return;
    const { x, y } = getPosicao(e);
    desenhar(x, y);
  });

  // Toque
  canvas.addEventListener("touchstart", e => {
    e.preventDefault();
    desenhando = true;
  }, { passive: false });
  canvas.addEventListener("touchend", e => {
    e.preventDefault();
    desenhando = false;
    ctx.beginPath();
  });
  canvas.addEventListener("touchcancel", e => {
    e.preventDefault();
    desenhando = false;
    ctx.beginPath();
  });
  canvas.addEventListener("touchmove", e => {
    e.preventDefault();
    if (!desenhando) return;
    const { x, y } = getPosicao(e);
    desenhar(x, y);
  });
}
    configurarCanvasAssinatura("assinaturaGerente");
    configurarCanvasAssinatura("assinaturaMotorista");

  </script>
</body>
</html>
