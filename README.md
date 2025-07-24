<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <title>Archivio Gare Cavalli</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
    }
    .gara-container {
      margin-bottom: 30px;
    }
    .gara {
      width: 100%;
      position: relative;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-bottom: 10px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 4px;
      text-align: center;
      position: relative;
    }
    input[type="text"], input[type="number"] {
      width: 90%;
      padding: 4px;
    }
    input.quota {
      width: 60px;
    }
    button {
      margin-top: 5px;
      margin-right: 10px;
    }
    .autocomplete-items {
      position: absolute;
      border: 1px solid #ccc;
      background-color: #fff;
      z-index: 99;
      max-height: 150px;
      overflow-y: auto;
      top: 100%;
      left: 0;
      right: 0;
    }
    .autocomplete-items div {
      padding: 5px;
      cursor: pointer;
    }
    .autocomplete-items div:hover {
      background-color: #f0f0f0;
    }
    #report {
      margin-top: 20px;
      background: #f9f9f9;
      border: 1px solid #ccc;
      padding: 10px;
      white-space: pre-wrap;
      font-family: monospace;
    }
    #importFile {
      display: none;
    }
  </style>
</head>
<body>
<h2>Archivio Gare Cavalli</h2>

<div class="gara-container">
  <div class="gara">
    <table id="gara1">
      <thead><tr><th>Corsia</th><th>Nome</th><th>Quota</th></tr></thead>
      <tbody id="body1"></tbody>
    </table>
    <button onclick="salvaGara(1)">Salva</button>
    <button onclick="pulisciTabella(1)">Pulisci Tabella</button>
    <button onclick="cercaGare()">Cerca</button>
  </div>
</div>

<button onclick="cancellaTutto()">Cancella Tutto (Storage)</button>
<button onclick="mostraTutteGare()">Mostra tutte le gare salvate</button>
<button onclick="exportBackup()">Backup dati (JSON)</button>
<button onclick="exportCSV()">Esporta in CSV</button>
<button onclick="document.getElementById('importFile').click()">Importa backup (JSON)</button>
<input type="file" id="importFile" accept=".json" onchange="importaBackup(event)">

<div id="report"></div>
<script>
const NUM_CORSIE = 6;

function inizializzaTabella() {
  const tbody = document.getElementById("body1");
  for (let i = 1; i <= NUM_CORSIE; i++) {
    tbody.innerHTML += `
      <tr>
        <td>${i}</td>
        <td><input type="text" id="nome1_${i}" class="nome" autocomplete="off" /></td>
        <td><input type="number" step="0.01" id="quota1_${i}" class="quota" /></td>
      </tr>
    `;
  }
}

setInterval(() => {
  const gare = localStorage.getItem("gare");
  if (gare) localStorage.setItem("backup_gare", gare);
}, 60000);

function exportBackup() {
  const data = localStorage.getItem("gare") || "[]";
  const blob = new Blob([data], { type: "application/json" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = `gare_backup_${new Date().toISOString().slice(0, 10)}.json`;
  link.click();
}

function importaBackup(event) {
  const file = event.target.files[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = function(e) {
    try {
      const dati = JSON.parse(e.target.result);
      if (Array.isArray(dati)) {
        localStorage.setItem("gare", JSON.stringify(dati));
        alert("Backup importato con successo.");
      } else {
        alert("Formato file non valido.");
      }
    } catch {
      alert("Errore nella lettura del file.");
    }
  };
  reader.readAsText(file);
}

function salvaGara(index) {
  const { nomi, quote } = getGaraData(index);
  if (nomi.includes("") || quote.includes("")) {
    alert("Compila tutti i campi prima di salvare.");
    return;
  }

  let gare = JSON.parse(localStorage.getItem("gare") || "[]");

  // Controllo gara identica
  const garaEsatta = gare.find(g => JSON.stringify(g.nomi) === JSON.stringify(nomi) && JSON.stringify(g.quote) === JSON.stringify(quote));
  if (garaEsatta) {
    alert("Questa gara esiste già.");
    return;
  }

  // Nuovo controllo: stessi nomi ma quote diverse
  const stessaCombinazioneNomi = gare.find(g => JSON.stringify(g.nomi) === JSON.stringify(nomi));
  if (stessaCombinazioneNomi) {
    const conferma = confirm("Esiste già una gara con gli stessi cavalli, ma quote diverse.\nVuoi salvarla comunque?");
    if (!conferma) return;
  }

  const reportAI = analisiAIAvanzata(nomi, quote, gare);
  mostraReport(reportAI.reportTesto);

  if (reportAI.alertMessage && !confirm(reportAI.alertMessage + "\n\nVuoi comunque salvare la gara?")) return;

  let tris = prompt("Inserisci tris vincente (es. 1,4,5):");
  if (!tris || tris.split(",").length !== 3) return alert("Formato tris non valido.");
  let quotaTris = prompt("Quota tris (es. 18.5):");
  if (!quotaTris || isNaN(parseFloat(quotaTris))) return alert("Quota non valida.");

  gare.push({ nomi, quote, tris: [{ combinazione: tris, quota: quotaTris }] });
  localStorage.setItem("gare", JSON.stringify(gare));
  alert("Gara salvata.");
}

function getGaraData(index) {
  const nomi = [], quote = [];
  for (let i = 1; i <= NUM_CORSIE; i++) {
    nomi.push(document.getElementById(`nome${index}_${i}`).value.trim());
    quote.push(document.getElementById(`quota${index}_${i}`).value.trim());
  }
  return { nomi, quote };
}

function mostraReport(testo) {
  document.getElementById("report").textContent = testo;
}

function analisiAIAvanzata(nomi, quote, gare) {
  const nuoveQuote = quote.map(q => parseFloat(q));
  const sommaQuote = nuoveQuote.reduce((a, b) => a + b, 0);
  const sogliaQuota = 0.2;
  const minGare = 3;
  const patternLabels = quote.map(q => {
    const val = parseFloat(q);
    if (val < 2) return "B";
    if (val <= 3.5) return "M";
    return "A";
  });

  let report = `🧠 ANALISI INTELLIGENTE\n----------------------\n`;
  report += `📊 Pattern quote: ${patternLabels.join("-")}\n`;
  report += `🧮 Somma quote: ${sommaQuote.toFixed(2)}\n`;

  let cavalliTotali = {}, cavalliCorsia = {};

  for (let i = 0; i < NUM_CORSIE; i++) {
    const nome = nomi[i];
    const quota = nuoveQuote[i];
    if (!nome || isNaN(quota)) continue;

    let tot = 0, podio = 0;

    gare.forEach(g => {
      for (let j = 0; j < NUM_CORSIE; j++) {
        if (g.nomi[j] === nome && Math.abs(parseFloat(g.quote[j]) - quota) <= sogliaQuota) {
          tot++;
          if (g.tris.some(t => t.combinazione.split(",").includes(String(j + 1)))) {
            podio++;
          }
        }
      }
    });

    if (tot >= minGare) {
      const perc = ((podio / tot) * 100).toFixed(1);
      report += `✔️ "${nome}" con quota ~${quota.toFixed(2)} → ${perc}% podio (${podio}/${tot})\n`;
    }

    cavalliTotali[nome] = cavalliTotali[nome] || { gare: 0, podi: 0 };
    cavalliTotali[nome].gare += tot;
    cavalliTotali[nome].podi += podio;

    cavalliCorsia[nome] = cavalliCorsia[nome] || {};
    cavalliCorsia[nome][i + 1] = cavalliCorsia[nome][i + 1] || { gare: 0, podi: 0 };
    cavalliCorsia[nome][i + 1].gare += tot;
    cavalliCorsia[nome][i + 1].podi += podio;
  }

  report += `\n📌 Analisi per cavallo globale:\n`;
  Object.keys(cavalliTotali).forEach(nome => {
    const { gare, podi } = cavalliTotali[nome];
    if (gare >= minGare) {
      report += `→ ${nome}: ${((podi / gare) * 100).toFixed(1)}% podio su ${gare} gare\n`;
    }
  });

  report += `\n📌 Analisi cavallo + corsia:\n`;
  Object.keys(cavalliCorsia).forEach(nome => {
    const dati = cavalliCorsia[nome];
    Object.keys(dati).forEach(corsia => {
      const { gare, podi } = dati[corsia];
      if (gare >= minGare) {
        report += `→ ${nome} in corsia ${corsia}: ${((podi / gare) * 100).toFixed(1)}% podio\n`;
      }
    });
  });

  report += `\n📌 Quote alte e somma quote:\n`;
  if (sommaQuote >= 18) {
    let cavalliAlti = nuoveQuote.map((q, i) => ({ nome: nomi[i], quota: q, idx: i + 1 }))
      .filter(c => c.quota >= 3.5);
    if (cavalliAlti.length > 0) {
      report += `Attenzione: somma alta. Quote >3.5:\n`;
      cavalliAlti.forEach(c => {
        report += `→ ${c.nome} (Corsia ${c.idx}, Quota ${c.quota})\n`;
      });
    }
  }

  return { reportTesto: report };
}

function setupAutocomplete() {
  const inputs = document.querySelectorAll("input.nome");
  const cavalli = new Set();
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  gare.forEach(g => g.nomi.forEach(n => cavalli.add(n)));

  inputs.forEach(input => {
    input.addEventListener("input", function() {
      closeLists();
      const val = this.value;
      if (!val) return;
      const list = document.createElement("div");
      list.setAttribute("class", "autocomplete-items");
      this.parentNode.appendChild(list);

      [...cavalli].forEach(nome => {
        if (nome.toLowerCase().startsWith(val.toLowerCase())) {
          const div = document.createElement("div");
          div.innerHTML = `<strong>${nome.substr(0, val.length)}</strong>${nome.substr(val.length)}<input type='hidden' value='${nome}'>`;
          div.addEventListener("click", () => {
            input.value = nome;
            closeLists();
          });
          list.appendChild(div);
        }
      });
    });
    input.addEventListener("blur", () => setTimeout(closeLists, 100));
  });

  function closeLists() {
    document.querySelectorAll(".autocomplete-items").forEach(el => el.remove());
  }
}

function cercaGare() {
  const nome = document.getElementById("nome1_1").value.trim().toLowerCase();
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  const risultati = gare.filter(g => g.nomi[0].toLowerCase() === nome);
  if (risultati.length === 0) return alert("Nessuna gara trovata con quel cavallo in corsia 1.");

  let index = 0;
  const win = window.open("", "Risultati Ricerca", "width=600,height=400");
  function mostraGara(i) {
    const g = risultati[i];
    win.document.body.innerHTML = `<h3>Gara ${i+1} di ${risultati.length}</h3><ul>
      ${g.nomi.map((n, idx) => `<li>Corsia ${idx+1}: ${n} (Quota: ${g.quote[idx]})</li>`).join("")}
      </ul><p><strong>Tris vincenti:</strong><br>${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("<br>")}</p>
      <button onclick="window.opener.prevGara()">&larr;</button>
      <button onclick="window.opener.nextGara()">&rarr;</button>`;
  }
  window.prevGara = () => { if (index > 0) index--; mostraGara(index); };
  window.nextGara = () => { if (index < risultati.length - 1) index++; mostraGara(index); };
  mostraGara(index);
}

function mostraTutteGare() {
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  if (gare.length === 0) return alert("Nessuna gara salvata.");
  const win = window.open("", "Gare Salvate", "width=600,height=600,scrollbars=yes");
  win.document.body.innerHTML = `<h2>${gare.length} Gare Salvate</h2>` + gare.map((g, idx) => `
    <h3>Gara ${idx + 1}</h3>
    <ul>${g.nomi.map((n, i) => `<li>Corsia ${i+1}: ${n} (Quota: ${g.quote[i]})</li>`).join("")}</ul>
    <p><strong>Tris:</strong><br>${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("<br>")}</p><hr>`).join("");
}

function cancellaTutto() {
  if (confirm("Sicuro di voler eliminare tutte le gare?")) {
    localStorage.removeItem("gare");
    alert("Gare eliminate.");
    document.getElementById("report").textContent = "";
  }
}

function exportCSV() {
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  if (gare.length === 0) return alert("Nessuna gara da esportare.");
  let csv = "Gara,Corsia,Nome,Quota,Tris,QuotaTris\n";
  gare.forEach((g, idx) => {
    g.nomi.forEach((nome, i) => {
      const tris = g.tris.map(t => t.combinazione).join(" | ");
      const qt = g.tris.map(t => t.quota).join(" | ");
      csv += `${idx + 1},${i + 1},${nome},${g.quote[i]},${tris},${qt}\n`;
    });
  });
  const blob = new Blob([csv], { type: "text/csv" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = `gare_export_${new Date().toISOString().slice(0, 10)}.csv`;
  link.click();
}

function pulisciTabella(index) {
  for (let i = 1; i <= NUM_CORSIE; i++) {
    document.getElementById(`nome${index}_${i}`).value = "";
    document.getElementById(`quota${index}_${i}`).value = "";
  }
}

window.addEventListener("DOMContentLoaded", () => {
  inizializzaTabella();
  setupAutocomplete();
});
</script>
</body>
</html>
