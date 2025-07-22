<!DOCTYPE html>
<html lang="it">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Archivio Gare Cavalli</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background-color: #f4f4f4;
    }

    h1,
    h2 {
      text-align: center;
    }

    .container {
      display: flex;
      justify-content: space-around;
      flex-wrap: wrap;
    }

    table,
    th,
    td {
      border: 1px solid black;
      border-collapse: collapse;
    }

    th,
    td {
      padding: 5px;
      text-align: center;
    }

    table {
      margin: 10px;
      background-color: white;
    }

    input[type="text"],
    input[type="number"] {
      width: 100%;
      padding: 5px;
      box-sizing: border-box;
      text-align: center;
    }

    button {
      margin: 5px;
      padding: 10px 15px;
    }

    #report {
      background: white;
      padding: 10px;
      margin-top: 20px;
    }
  </style>
</head>

<body>
  <h1>Archivio Gare Cavalli</h1>

  <div class="container">
    <div>
      <h2>Nuova Gara</h2>
      <table>
        <tr>
          <th>Corsia</th>
          <th>Nome Cavallo</th>
          <th>Quota</th>
        </tr>
        <tbody id="nuovaGara">
          <!-- Generato via JS -->
        </tbody>
      </table>
      <label for="trisVincente">Tris Vincente (es. 1-2-3):</label>
      <input type="text" id="trisVincente" inputmode="numeric" pattern="[0-9-]+" placeholder="Esempio: 1-2-3">
      <button onclick="salvaGara()">Salva Gara</button>
      <button onclick="pulisciTabelle()">Pulisci</button>
    </div>

    <div>
      <h2>Archivio Gare</h2>
      <table>
        <thead>
          <tr>
            <th>Gara</th>
            <th>Corsie</th>
            <th>Quote</th>
            <th>Tris</th>
            <th>Azioni</th>
          </tr>
        </thead>
        <tbody id="archivioGare">
        </tbody>
      </table>
    </div>
  </div>

  <div id="report"></div>

  <script>
    let gare = JSON.parse(localStorage.getItem('gare')) || [];

    function generaInputGara() {
      const tbody = document.getElementById("nuovaGara");
      tbody.innerHTML = "";
      for (let i = 1; i <= 6; i++) {
        const row = document.createElement("tr");
        row.innerHTML = `
          <td>${i}</td>
          <td><input type="text" id="nome${i}" placeholder="Nome Cavallo ${i}"></td>
          <td><input type="number" id="quota${i}" placeholder="Quota" inputmode="decimal"></td>
        `;
        tbody.appendChild(row);
      }
    }

    function aggiornaArchivio() {
      const tbody = document.getElementById("archivioGare");
      tbody.innerHTML = "";
      gare.forEach((gara, index) => {
        const row = document.createElement("tr");
        row.innerHTML = `
          <td>${index + 1}</td>
          <td>${gara.nomi.join("<br>")}</td>
          <td>${gara.quote.join("<br>")}</td>
          <td>${gara.tris.map(t => t.combinazione + ' (Quota: ' + t.quota + ')').join("<br>")}</td>
          <td><button onclick="cancellaGara(${index})">Elimina</button></td>
        `;
        tbody.appendChild(row);
      });
    }

    function salvaGara() {
      const nomi = [];
      const quote = [];
      for (let i = 1; i <= 6; i++) {
        const nome = document.getElementById(`nome${i}`).value.trim();
        const quota = parseFloat(document.getElementById(`quota${i}`).value);
        if (!nome || isNaN(quota)) {
          alert("Inserisci tutti i nomi e le quote.");
          return;
        }
        nomi.push(nome);
        quote.push(quota);
      }

      const trisInput = document.getElementById("trisVincente").value.trim();
      if (!trisInput.match(/^\d+-\d+-\d+$/)) {
        alert("Inserisci la Tris Vincente nel formato corretto (es. 1-2-3).”);
        return;
      }
      const nuovaTris = [{ combinazione: trisInput, quota: quote.join("+") }];

      const match = gare.find(g => g.quote.every((q, idx) => q === quote[idx]) && !g.nomi.every((n, i) => n === nomi[i]));
      if (match) {
        if (!confirm(`Attenzione: queste quote esistono già in un'altra gara con cavalli diversi.\nTris vincente suggerita: ${match.tris[0].combinazione} (Quota: ${match.tris[0].quota})\n\nVuoi comunque aggiungere la gara?`)) {
          return;
        }
      }

      const gara = { nomi, quote, tris: nuovaTris };
      gare.push(gara);
      localStorage.setItem('gare', JSON.stringify(gare));
      aggiornaArchivio();
      generaInputGara();
      document.getElementById("trisVincente").value = "";
      document.getElementById("report").innerHTML = generaReportAnalisiCavalli(nomi, quote, trisInput);
    }

    function cancellaGara(index) {
      if (confirm("Sei sicuro di voler eliminare questa gara?")) {
        gare.splice(index, 1);
        localStorage.setItem('gare', JSON.stringify(gare));
        aggiornaArchivio();
      }
    }

    function pulisciTabelle() {
      if (confirm("Vuoi cancellare tutte le gare?")) {
        gare = [];
        localStorage.removeItem('gare');
        aggiornaArchivio();
      }
    }

    function generaReportAnalisiCavalli(nomi, quote, trisInput) {
      let report = "<h3>Analisi AI Avanzata della Gara Inserita</h3>";
      const trisArray = trisInput.split("-").map(t => parseInt(t));
      const totaleGare = gare.length;
      if (totaleGare < 3) {
        return report + "<p>Ancora pochi dati per l'analisi (minimo 3 gare necessarie).</p>";
      }

      let cavalloTop3 = [];
      let quotaFaVincere = [];
      let coppieCavalli = [];

      for (let i = 0; i < 6; i++) {
        let conta = 0;
        gare.forEach(g => {
          const trisNumeri = g.tris[0].combinazione.split("-").map(x => parseInt(x));
          if (g.nomi[i] === nomi[i] && Math.abs(g.quote[i] - quote[i]) < 0.01 && trisNumeri.includes(i + 1)) {
            conta++;
          }
        });
        const perc = (conta / totaleGare) * 100;
        if (perc >= 60) {
          cavalloTop3.push(`➡️ Cavallo <b>${nomi[i]}</b> in corsia ${i + 1} con quota ${quote[i].toFixed(2)} è entrato nella Tris nel ${perc.toFixed(1)}% dei casi.`);
        }
      }

      for (let i = 0; i < 6; i++) {
        let conta = 0;
        for (let target = 1; target <= 6; target++) {
          let matchCount = 0;
          gare.forEach(g => {
            const trisNumeri = g.tris[0].combinazione.split("-").map(x => parseInt(x));
            if (Math.abs(g.quote[i] - quote[i]) <= 0.10 && trisNumeri.includes(target)) {
              matchCount++;
            }
          });
          const perc = (matchCount / totaleGare) * 100;
          if (perc >= 60) {
            quotaFaVincere.push(`🎯 La quota <b>${quote[i].toFixed(2)}</b> (±0.10) in corsia ${i + 1} ha fatto vincere spesso la corsia ${target} (${perc.toFixed(1)}%).`);
          }
        }
      }

      const nomiUnici = [...new Set(nomi)];
      for (let i = 0; i < nomiUnici.length; i++) {
        for (let j = i + 1; j < nomiUnici.length; j++) {
          let conta = 0;
          gare.forEach(g => {
            const presenti = g.nomi.includes(nomiUnici[i]) && g.nomi.includes(nomiUnici[j]);
            const trisNumeri = g.tris[0].combinazione.split("-").map(n => parseInt(n));
            if (presenti && trisNumeri.length >= 3) conta++;
          });
          const perc = (conta / totaleGare) * 100;
          if (perc >= 60) {
            coppieCavalli.push(`🤝 La coppia <b>${nomiUnici[i]}</b> e <b>${nomiUnici[j]}</b> compare in tris nel ${perc.toFixed(1)}% delle gare.`);
          }
        }
      }

      if (cavalloTop3.length > 0) {
        report += `<h4>✨ Pattern Cavalli e Quote</h4><ul><li>${cavalloTop3.join("</li><li>")}</li></ul>`;
      }
      if (quotaFaVincere.length > 0) {
        report += `<h4>📊 Quote che influenzano vincita</h4><ul><li>${quotaFaVincere.join("</li><li>")}</li></ul>`;
      }
      if (coppieCavalli.length > 0) {
        report += `<h4>👥 Coppie vincenti</h4><ul><li>${coppieCavalli.join("</li><li>")}</li></ul>`;
      }

      if (report === "<h3>Analisi AI Avanzata della Gara Inserita</h3>") {
        report += "<p>Nessun pattern rilevante sopra il 60%.</p>";
      }

      return report;
    }

    generaInputGara();
    aggiornaArchivio();
  </script>
</body>
</html>
