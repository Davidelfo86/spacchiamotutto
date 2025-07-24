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

// Inizializza tabella
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

// Backup automatico
setInterval(() => {
  const gare = localStorage.getItem("gare");
  if (gare) localStorage.setItem("backup_gare", gare);
}, 60000);

// Backup manuale
function exportBackup() {
  const data = localStorage.getItem("gare") || "[]";
  const blob = new Blob([data], { type: "application/json" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = `gare_backup_${new Date().toISOString().slice(0, 10)}.json`;
  link.click();
}

// Importa backup
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

function salvaGara(index) {
  const { nomi, quote } = getGaraData(index);
  if (nomi.includes("") || quote.includes("")) {
    alert("Compila tutti i campi prima di salvare.");
    return;
  }

  let gare = JSON.parse(localStorage.getItem("gare") || "[]");

  const quoteStr = JSON.stringify(quote.map(q => parseFloat(q).toFixed(2)));
  const nomiStr = JSON.stringify(nomi);

  // Quote uguali, cavalli diversi
  const gareStessaQuota = gare.filter(g => JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) === quoteStr && JSON.stringify(g.nomi) !== nomiStr);
  if (gareStessaQuota.length > 0) {
    let msg = `⚠️ Questa combinazione di quote è già presente in ${gareStessaQuota.length} gara/e con cavalli diversi.\n`;
    gareStessaQuota.forEach((g, i) => {
      msg += `\nGara ${i + 1} → Tris vincenti:\n${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n")}`;
    });
    alert(msg);
  }

  // Nomi uguali, quote diverse
  const gareStessiNomi = gare.filter(g => JSON.stringify(g.nomi) === nomiStr && JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) !== quoteStr);
  if (gareStessiNomi.length > 0) {
    let msg = `⚠️ Esiste già una gara con gli stessi cavalli ma quote differenti:\n`;
    gareStessiNomi.forEach((g, i) => {
      msg += `\nGara ${i + 1} → Quote: ${g.quote.join(", ")}\nTris:\n${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n")}`;
    });
    if (!confirm(msg + `\n\nVuoi salvare comunque?`)) return;
  }

  // Esegui analisi AI PRIMA di chiedere la tris
  const reportAI = analisiAIAvanzata(nomi, quote, gare);
  mostraReport(reportAI.reportTesto);

  // Chiedi se vuoi procedere
  if (!confirm("Vuoi procedere con il salvataggio della gara dopo l’analisi AI?")) return;

  // Gara identica già salvata
  const garaEsatta = gare.find(g => JSON.stringify(g.nomi) === nomiStr && JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) === quoteStr);
  if (garaEsatta) {
    let msg = `⚠️ Questa gara esiste già.\nTris salvate:\n`;
    msg += garaEsatta.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n");
    if (confirm(msg + `\n\nVuoi salvare comunque un'altra tris?`)) {
      let tris = prompt("Inserisci nuova tris vincente (es. 1,4,5):");
      if (!tris || tris.split(",").length !== 3) return alert("Formato tris non valido.");
      let quotaTris = prompt("Quota tris (es. 18.5):");
      if (!quotaTris || isNaN(parseFloat(quotaTris))) return alert("Quota non valida.");
      if (garaEsatta.tris.some(t => t.combinazione === tris && parseFloat(t.quota) === parseFloat(quotaTris))) {
        alert("✅ Abbiamo vinto allora!");
        return;
      }
      garaEsatta.tris.push({ combinazione: tris, quota: quotaTris });
      localStorage.setItem("gare", JSON.stringify(gare));
      alert("Nuova tris aggiunta.");
    }
    return;
  }

  // Gara nuova → chiedi tris e quota
  let tris = prompt("Inserisci tris vincente (es. 1,4,5):");
  if (!tris || tris.split(",").length !== 3) return alert("Formato tris non valido.");
  let quotaTris = prompt("Quota tris (es. 18.5):");
  if (!quotaTris || isNaN(parseFloat(quotaTris))) return alert("Quota non valida.");

  gare.push({ nomi, quote, tris: [{ combinazione: tris, quota: quotaTris }] });
  localStorage.setItem("gare", JSON.stringify(gare));
  alert("Gara salvata.");
}

function analisiAIAvanzata(nomi, quote, gare) {
  const nuoveQuote = quote.map(q => parseFloat(q));
  const sommaQuote = nuoveQuote.reduce((a, b) => a + b, 0);
  const minGareAnalisi = 10;
  const patternLabels = nuoveQuote.map(q => {
    if (q < 2) return "B";
    if (q <= 3.5) return "M";
    return "A";
  });

  let report = `🧠 ANALISI INTELLIGENTE\n----------------------\n`;
  report += `📊 Pattern quote: ${patternLabels.join("-")}\n`;
  report += `🧮 Somma quote: ${sommaQuote.toFixed(2)}\n\n`;

  let cavalliTotali = {}, cavalliCorsia = {};
  let quoteStatistiche = {};
  let trisSuggerita = [];
  let quoteSimili = [];
  let patternSimili = [];

  // Quote storiche simili e pattern simili
  gare.forEach(g => {
    const q = g.quote.map(x => parseFloat(x));
    const patternGara = q.map(qv => qv < 2 ? "B" : qv <= 3.5 ? "M" : "A");
    const matchCount = patternGara.filter((v, i) => v === patternLabels[i]).length;
    if (matchCount >= 5) patternSimili.push(g);

    const simili = q.filter((val, i) => Math.abs(val - nuoveQuote[i]) <= 0.2).length;
    if (simili >= 5) quoteSimili.push(g);

    q.forEach((val, idx) => {
      const intervallo = (Math.round(val * 2) / 2).toFixed(1);
      quoteStatistiche[intervallo] = quoteStatistiche[intervallo] || { podi: 0, tot: 0 };
      if (g.tris.some(t => t.combinazione.split(",").includes(String(idx + 1)))) {
        quoteStatistiche[intervallo].podi++;
      }
      quoteStatistiche[intervallo].tot++;
    });
  });

  // Cavallo favorito e sfavorito
  let minQuota = Math.min(...nuoveQuote);
  let maxQuota = Math.max(...nuoveQuote);
  let favoritoIdx = nuoveQuote.indexOf(minQuota);
  let sfavoritoIdx = nuoveQuote.indexOf(maxQuota);

  report += `🏇 Cavallo favorito: ${nomi[favoritoIdx]} (Corsia ${favoritoIdx + 1}, Quota ${minQuota})\n`;
  report += `🐢 Cavallo sfavorito: ${nomi[sfavoritoIdx]} (Corsia ${sfavoritoIdx + 1}, Quota ${maxQuota})\n`;

  // Storico per cavallo favorito
  let storicoFavorito = gare.filter(g => {
    const q = g.quote.map(x => parseFloat(x));
    const min = Math.min(...q);
    return g.nomi[q.indexOf(min)] === nomi[favoritoIdx];
  });

  const favoritoPodio = storicoFavorito.filter(g =>
    g.tris.some(t => t.combinazione.split(",").includes(String(favoritoIdx + 1)))
  );

  if (storicoFavorito.length > 0) {
    report += `📊 Il favorito in gare storiche è andato a podio ${favoritoPodio.length} su ${storicoFavorito.length} volte (${((favoritoPodio.length / storicoFavorito.length) * 100).toFixed(1)}%)\n`;
  } else {
    report += `📊 Nessuna gara storica trovata per ${nomi[favoritoIdx]} con quota simile.\n`;
  }

  // Cavallo sfavorito: analisi su podio o vincente
  let storicoSfavorito = gare.filter(g => g.nomi.includes(nomi[sfavoritoIdx]));
  let podioSfavorito = 0;
  let vittorieSfavorito = 0;

  storicoSfavorito.forEach(g => {
    g.tris.forEach(t => {
      const corsie = t.combinazione.split(",");
      const idx = g.nomi.findIndex(n => n === nomi[sfavoritoIdx]);
      if (idx !== -1 && corsie.includes(String(idx + 1))) {
        podioSfavorito++;
        if (corsie[0] === String(idx + 1)) vittorieSfavorito++;
      }
    });
  });

  if (storicoSfavorito.length > 0) {
    report += `📊 Il cavallo sfavorito è andato a podio ${podioSfavorito} su ${storicoSfavorito.length} gare (${((podioSfavorito / storicoSfavorito.length) * 100).toFixed(1)}%)\n`;
    if (vittorieSfavorito > 0) {
      report += `🎉 Ha vinto ${vittorieSfavorito} volte! Possibile sorpresa.\n`;
    }
  }

  // Analisi per cavallo globale
  report += `\n📌 Analisi per cavallo globale:\n`;
  nomi.forEach((nome, i) => {
    let tot = 0, podio = 0;
    gare.forEach(g => {
      for (let j = 0; j < g.nomi.length; j++) {
        if (g.nomi[j] === nome && Math.abs(parseFloat(g.quote[j]) - nuoveQuote[i]) <= 0.2) {
          tot++;
          if (g.tris.some(t => t.combinazione.split(",").includes(String(j + 1)))) podio++;
        }
      }
    });
    if (tot >= minGareAnalisi) {
      report += `→ ${nome}: ${(podio / tot * 100).toFixed(1)}% podio su ${tot} gare\n`;
      trisSuggerita.push({ nome, corsia: i + 1, perc: podio / tot, tot });
    }
  });

  // Analisi cavallo + corsia
  report += `\n📌 Analisi cavallo + corsia:\n`;
  nomi.forEach((nome, i) => {
    let tot = 0, podio = 0;
    gare.forEach(g => {
      if (g.nomi[i] === nome && Math.abs(parseFloat(g.quote[i]) - nuoveQuote[i]) <= 0.2) {
        tot++;
        if (g.tris.some(t => t.combinazione.split(",").includes(String(i + 1)))) podio++;
      }
    });
    if (tot >= minGareAnalisi) {
      report += `→ ${nome} in corsia ${i + 1}: ${(podio / tot * 100).toFixed(1)}% podio\n`;
    }
  });

  // Tris consigliata
  report += `\n🎯 Tris consigliata:\n`;
  if (trisSuggerita.length >= 3) {
    const tris = trisSuggerita.sort((a, b) => b.perc - a.perc).slice(0, 3);
    tris.forEach(c => {
      report += `→ ${c.nome} (Corsia ${c.corsia}) - ${(c.perc * 100).toFixed(1)}% podio su ${c.tot} gare\n`;
    });
  } else {
    report += `Nessuna tris consigliata: servono almeno 3 cavalli con ${minGareAnalisi} gare analizzate.\n`;
  }

  // Quote spesso vincenti
  report += `\n📌 Quote spesso vincenti:\n`;
  Object.keys(quoteStatistiche).sort((a, b) => parseFloat(a) - parseFloat(b)).forEach(q => {
    const { podi, tot } = quoteStatistiche[q];
    if (tot >= 5) {
      report += `→ Quota ${q}: ${podi} su ${tot} podio (${(podi / tot * 100).toFixed(1)}%)\n`;
    }
  });

  // Quote alte
  if (sommaQuote >= 18) {
    report += `\n📌 Quote alte e somma quote:\nAttenzione: somma alta. Quote >3.5:\n`;
    nuoveQuote.forEach((q, i) => {
      if (q > 3.5) report += `→ ${nomi[i]} (Corsia ${i + 1}, Quota ${q})\n`;
    });
  }

  // Gare con quote simili
  if (quoteSimili.length > 0) {
    report += `\n📌 Gare storiche con quote simili: ${quoteSimili.length}\n`;
    quoteSimili.forEach((g, i) => {
      g.tris.forEach(t => {
        report += `🧩 Gara ${i + 1} → Tris: ${t.combinazione} (Quota: ${t.quota})\n`;
      });
    });
  }

  // Gare con pattern simile e sorprese
  if (patternSimili.length > 0) {
    let sorprese = 0;
    patternSimili.forEach(g => {
      g.tris.forEach(t => {
        const primaCorsia = parseInt(t.combinazione.split(",")[0]) - 1;
        const quotaVincente = parseFloat(g.quote[primaCorsia]);
        if (quotaVincente > 10) sorprese++;
      });
    });
    report += `\n📈 Gare con pattern quote simile: ${patternSimili.length}`;
    if (sorprese > 0) {
      report += ` → In ${sorprese} di queste ha vinto un cavallo con quota >10!\n`;
    } else {
      report += ` → Nessuna sorpresa rilevata in quelle gare.\n`;
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
