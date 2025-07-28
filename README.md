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
    <button onclick="startVoiceInput()">🎙️ Detta cavalli</button>
  </div>
</div>

<button onclick="cancellaTutto()">Cancella Tutto (Storage)</button>
<button onclick="mostraTutteGare()">Mostra tutte le gare salvate</button>
<button onclick="exportBackup()">Backup dati (JSON)</button>
<button onclick="exportCSV()">Esporta in CSV</button>
<button onclick="document.getElementById('importFile').click()">Importa backup (JSON)</button>
<button onclick="eliminaGaraPopup()">🗑️ Elimina Gara</button>
<button onclick="eliminaTrisSingolaPopup()">➖ Elimina una Tris</button>
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
        <td><input type="text" id="nome1_${i}" class="nome" name="cavallo${i}" autocomplete="off" /></td>
<td><input type="number" step="0.01" id="quota1_${i}" class="quota" name="quota${i}" /></td>
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
function capitalize(str) {
  if (!str) return "";
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}
function getGaraData(numeroGara) {
  const nomi = [];
  const quote = [];

  document.querySelectorAll(`#gara${numeroGara} input[name^="cavallo"]`).forEach(input => {
    nomi.push(capitalize(input.value.trim()));
  });

  document.querySelectorAll(`#gara${numeroGara} input[name^="quota"]`).forEach(input => {
    quote.push(input.value.trim());
  });

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

  // Esegui analisi AI
  const reportAI = analisiAIAvanzata(nomi, quote, gare);

  // Chiedi conferma
  if (!confirm("Vuoi procedere con il salvataggio della gara dopo l’analisi AI?")) {
    mostraReport(reportAI.reportTesto); // mostra il riquadro AI se clicchi Annulla
    return;
  }

  mostraReport(reportAI.reportTesto); // opzionale anche qui

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
  const minGareAnalisi = 5;
  const patternLabels = nuoveQuote.map(q => {
    if (q <= 2.5) return "B";
    if (q <= 6.5) return "M";
    if (q <= 9.9) return "A";
    return "SA";
});

  let report = `🧠 ANALISI INTELLIGENTE\n----------------------\n`;
  report += `📊 Pattern quote: ${patternLabels.join("-")}\n`;
  report += `🧮 Somma quote: ${sommaQuote.toFixed(2)}\n\n`;

  let trisSuggerita = [];
  let quoteStatistiche = {};
  let patternSimili = [], quoteSimili = [];
  let cavalliGlobale = {}, cavalliCorsia = {}, combinazioniTris = {}, trisCluster = {};

  // Analisi gare storiche
  gare.forEach(g => {
    const q = g.quote.map(x => parseFloat(x));
    const patternGara = q.map(qv => qv < 2 ? "B" : qv <= 3.5 ? "M" : "A");
    const matchCount = patternGara.filter((v, i) => v === patternLabels[i]).length;
    if (matchCount >= 5) patternSimili.push(g);

    const simili = q.filter((val, i) => Math.abs(val - nuoveQuote[i]) <= 0.3).length;
    if (simili >= 5) quoteSimili.push(g);

    q.forEach((val, idx) => {
      const intervallo = (Math.round(val * 2) / 2).toFixed(1);
      quoteStatistiche[intervallo] = quoteStatistiche[intervallo] || { podi: 0, tot: 0 };
      if (g.tris.some(t => t.combinazione.split(",").includes(String(idx + 1)))) {
        quoteStatistiche[intervallo].podi++;
      }
      quoteStatistiche[intervallo].tot++;
    });

    g.nomi.forEach((n, i) => {
      if (!nomi.includes(n)) return; // Solo cavalli della gara attuale
      const corsia = i + 1;
      const q = parseFloat(g.quote[i]);

      cavalliGlobale[n] = cavalliGlobale[n] || { tot: 0, podio: 0, quote: [], corsie: [] };
      cavalliGlobale[n].tot++;
      cavalliGlobale[n].quote.push(q);
      cavalliGlobale[n].corsie.push(corsia);

      if (g.tris.some(t => t.combinazione.split(",").includes(String(corsia)))) {
        cavalliGlobale[n].podio++;
      }

      const k = `${n}_C${corsia}`;
      cavalliCorsia[k] = cavalliCorsia[k] || { tot: 0, podio: 0 };
      if (g.tris.some(t => t.combinazione.split(",").includes(String(corsia)))) {
        cavalliCorsia[k].podio++;
      }
    });

    g.tris.forEach(t => {
      const key = t.combinazione.split(",").sort().join("-");
      combinazioniTris[key] = (combinazioniTris[key] || 0) + 1;
      const clusterKey = t.combinazione.split(",").map(n => parseInt(n)).sort((a, b) => a - b).join("-");
      trisCluster[clusterKey] = (trisCluster[clusterKey] || 0) + 1;
    });
  });

  // Cavallo favorito e sfavorito
  const minQuota = Math.min(...nuoveQuote);
  const maxQuota = Math.max(...nuoveQuote);
  const favoritoIdx = nuoveQuote.indexOf(minQuota);
  const sfavoritoIdx = nuoveQuote.indexOf(maxQuota);

  report += `🏇 Cavallo favorito: ${nomi[favoritoIdx]} (Corsia ${favoritoIdx + 1}, Quota ${minQuota})\n`;
  report += `🐢 Cavallo sfavorito: ${nomi[sfavoritoIdx]} (Corsia ${sfavoritoIdx + 1}, Quota ${maxQuota})\n`;

  // Sorprese storiche del cavallo sfavorito
  const storicoSfavorito = gare.filter(g => g.nomi.includes(nomi[sfavoritoIdx]));
  let sorprese = 0;
  storicoSfavorito.forEach(g => {
    g.tris.forEach(t => {
      const corsie = t.combinazione.split(",");
      const idx = g.nomi.indexOf(nomi[sfavoritoIdx]);
      if (idx !== -1 && corsie[0] === String(idx + 1)) sorprese++;
    });
  });
  if (sorprese > 0) {
    report += `🎯 Sorpresa: ${sorprese} vittorie del cavallo sfavorito storico!\n`;
  }

  // Cavalli ricorrenti (solo quelli della gara attuale)
  report += `\n📌 Cavalli ricorrenti (storico della gara attuale):\n`;
  Object.entries(cavalliGlobale).forEach(([nome, stats]) => {
    if (stats.tot >= minGareAnalisi) {
      const perc = (stats.podio / stats.tot) * 100;
      const avg = stats.quote.reduce((a, b) => a + b, 0) / stats.quote.length;
      report += `→ ${nome}: ${perc.toFixed(1)}% podio su ${stats.tot} gare | Quota media: ${avg.toFixed(2)}\n`;
      trisSuggerita.push({ nome, perc, corsie: stats.corsie });
    }
  });

  // Cavalli quasi vincenti
  report += `\n📌 Cavalli “quasi vincenti” (presenti in gara):\n`;
  for (let [nome, stats] of Object.entries(cavalliGlobale)) {
    let primi = 0;
    gare.forEach(g => {
      const idx = g.nomi.indexOf(nome);
      if (idx !== -1) {
        g.tris.forEach(t => {
          if (t.combinazione.split(",")[0] === String(idx + 1)) primi++;
        });
      }
    });
    if (stats.tot >= minGareAnalisi && stats.podio > 0 && primi === 0) {
      report += `→ ${nome}: ${stats.podio} podi, 0 vittorie su ${stats.tot} gare!\n`;
    }
  }

  // Cluster tris ricorrenti
  report += `\n📌 Cluster di tris vincenti ricorrenti:\n`;
  Object.entries(trisCluster).filter(([k, v]) => v > 1).forEach(([k, v]) => {
    report += `→ Combinazione ${k.replace(/-/g, ",")}: ${v} volte\n`;
  });

  // Predizione tris AI
  report += `\n🤖 Predizione Tris AI (solo cavalli in gara):\n`;
  if (trisSuggerita.length >= 3) {
    const trisFinale = trisSuggerita.sort((a, b) => b.perc - a.perc).slice(0, 3);
    trisFinale.forEach((c, i) => {
      const corsia = c.corsie.length ? c.corsie[0] : "?";
      report += `#${i + 1} → ${c.nome} (Corsia tipica: ${corsia}) - ${c.perc.toFixed(1)}%\n`;
    });
  } else {
    report += `Non abbastanza dati per una predizione affidabile.\n`;
  }

  // Confronto quote vincitrici con gare storiche simili
  report += `\n📊 Confronto con gare storiche simili:\n`;
  let sorpresePattern = 0, sorpreseQuote = 0;
  patternSimili.forEach(g => {
    const q = g.quote.map(v => parseFloat(v));
    const vincente = parseInt(g.tris[0]?.combinazione.split(",")[0]) - 1;
    if (q[vincente] > 10) sorpresePattern++;
  });
  quoteSimili.forEach(g => {
    const q = g.quote.map(v => parseFloat(v));
    const vincente = parseInt(g.tris[0]?.combinazione.split(",")[0]) - 1;
    if (q[vincente] > 10) sorpreseQuote++;
  });

  if (patternSimili.length > 0) {
    const perc = ((sorpresePattern / patternSimili.length) * 100).toFixed(1);
    report += `→ ${patternSimili.length} gare con pattern simile. In ${sorpresePattern} ha vinto quota >10 (${perc}%)\n`;
  }
  if (quoteSimili.length > 0) {
    const perc = ((sorpreseQuote / quoteSimili.length) * 100).toFixed(1);
    report += `→ ${quoteSimili.length} gare con quote simili. In ${sorpreseQuote} ha vinto quota >10 (${perc}%)\n`;
  }
  if (!patternSimili.length && !quoteSimili.length) {
    report += `→ Nessuna gara simile trovata.\n`;
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

  let csv = "Gara;Corsia;Nome;Quota;Tris Vincente;Quota Tris\n";

  gare.forEach((g, idx) => {
    g.nomi.forEach((nome, i) => {
      g.tris.forEach(t => {
        csv += `${idx + 1};${i + 1};${nome};${g.quote[i]};${t.combinazione};${t.quota}\n`;
      });
    });
  });

  const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = `gare_export_${new Date().toISOString().slice(0, 10)}.csv`;
  link.click();
}
function eliminaGaraPopup() {
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  if (gare.length === 0) {
    alert("Nessuna gara salvata.");
    return;
  }

  const id = prompt(`Inserisci il numero ID della gara da eliminare (1-${gare.length}):`);
  if (!id || isNaN(id)) {
    alert("ID non valido.");
    return;
  }

  const index = parseInt(id) - 1;
  if (index < 0 || index >= gare.length) {
    alert("ID fuori intervallo.");
    return;
  }

  const gara = gare[index];
  const conferma = confirm(
    `Vuoi davvero eliminare la gara #${id}?\n\n` +
    gara.nomi.map((n, i) => `Corsia ${i + 1}: ${n} (Quota: ${gara.quote[i]})`).join("\n") +
    `\n\nTris:\n${gara.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n")}`
  );

  if (!conferma) return;

  gare.splice(index, 1);
  localStorage.setItem("gare", JSON.stringify(gare));
  alert(`Gara #${id} eliminata con successo.`);
  document.getElementById("report").textContent = "";
}
function eliminaTrisSingolaPopup() {
  const gare = JSON.parse(localStorage.getItem("gare") || "[]");
  if (gare.length === 0) {
    alert("Nessuna gara salvata.");
    return;
  }

  const id = prompt(`Inserisci il numero ID della gara da cui eliminare una tris (1-${gare.length}):`);
  if (!id || isNaN(id)) {
    alert("ID non valido.");
    return;
  }

  const index = parseInt(id) - 1;
  if (index < 0 || index >= gare.length) {
    alert("ID fuori intervallo.");
    return;
  }

  const gara = gare[index];
  if (gara.tris.length === 0) {
    alert("Questa gara non ha tris salvate.");
    return;
  }

  const listaTris = gara.tris.map((t, i) => `#${i + 1} → ${t.combinazione} (Quota: ${t.quota})`).join("\n");
  const scelta = prompt(
    `Tris salvate nella gara #${id}:\n${listaTris}\n\nInserisci il numero della tris da eliminare:`
  );

  const trisIndex = parseInt(scelta) - 1;
  if (isNaN(trisIndex) || trisIndex < 0 || trisIndex >= gara.tris.length) {
    alert("Indice tris non valido.");
    return;
  }

  const conferma = confirm(`Vuoi davvero eliminare la tris #${scelta}: ${gara.tris[trisIndex].combinazione}?`);
  if (!conferma) return;

  gara.tris.splice(trisIndex, 1);
  localStorage.setItem("gare", JSON.stringify(gare));

  alert("Tris eliminata con successo.");
  document.getElementById("report").textContent = "";
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
  startAIWatcher();
});
function startVoiceInput() {
  const recognition = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
  recognition.lang = "it-IT";
  recognition.interimResults = false;
  recognition.maxAlternatives = 1;

  recognition.onresult = function(event) {
    const result = event.results[0][0].transcript;

    // Accetta anche "punto" come separatore oltre a "virgola" e ","
    const righe = result.split(/virgola|punto|,|\./i);

    righe.forEach((riga, i) => {
      const parts = riga.trim().split(" ");
      if (parts.length >= 2) {
        const nome = capitalize(parts.slice(0, -1).join(" ").trim());
        const quota = parts[parts.length - 1].replace(",", ".").trim();
        const inputNome = document.querySelector(`#gara1 input[name="cavallo${i + 1}"]`);
        const inputQuota = document.querySelector(`#gara1 input[name="quota${i + 1}"]`);
        if (inputNome && inputQuota) {
          inputNome.value = nome;
          inputQuota.value = quota;
        }
      }
    });

    // Analisi AI automatica subito dopo la dettatura
    const dati = getGaraData(1);
    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const reportAI = analisiAIAvanzata(dati.nomi, dati.quote, gare);
    mostraReport(reportAI.reportTesto);
  };

  recognition.onerror = function(event) {
    alert("Errore nella dettatura vocale: " + event.error);
  };

  recognition.start();
}
function startAIWatcher() {
  let ultimaFirma = "";

  // Analisi iniziale se i dati sono già completi
  const iniziale = getGaraData(1);
  if (!iniziale.nomi.includes("") && !iniziale.quote.includes("")) {
    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const reportAI = analisiAIAvanzata(iniziale.nomi, iniziale.quote, gare);
    mostraReport(reportAI.reportTesto);
    ultimaFirma = iniziale.nomi.join("|") + "::" + iniziale.quote.join("|");
    aggiornaBadgeOrario();
  }

  // Tooltip su ogni input cavallo o quota (facoltativo, ma utile)
  document.querySelectorAll("#gara1 input").forEach(input => {
    input.title = "Modifica per aggiornare l'analisi AI";
  });

  // Badge orario AI
  let badge = document.getElementById("badgeAI");
  if (!badge) {
    badge = document.createElement("div");
    badge.id = "badgeAI";
    badge.style.position = "absolute";
    badge.style.top = "10px";
    badge.style.right = "10px";
    badge.style.padding = "6px 12px";
    badge.style.background = "#d0ebff";
    badge.style.border = "1px solid #339af0";
    badge.style.borderRadius = "8px";
    badge.style.fontSize = "12px";
    badge.style.fontFamily = "monospace";
    badge.style.zIndex = 999;
    document.body.appendChild(badge);
  }

  function aggiornaBadgeOrario() {
    const ora = new Date();
    const orario = ora.toLocaleTimeString("it-IT", { hour: '2-digit', minute: '2-digit', second: '2-digit' });
    badge.innerText = `AI aggiornata alle ${orario}`;
  }

  // Watcher ogni secondo
  setInterval(() => {
    const { nomi, quote } = getGaraData(1);
    if (nomi.includes("") || quote.includes("")) return;

    const firmaAttuale = nomi.join("|") + "::" + quote.join("|");
    if (firmaAttuale === ultimaFirma) return;

    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const reportAI = analisiAIAvanzata(nomi, quote, gare);
    mostraReport(reportAI.reportTesto);

    ultimaFirma = firmaAttuale;
    aggiornaBadgeOrario();
  }, 1000);
}
function mostraOrarioAggiornamento() {
  const now = new Date();
  const orario = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  const div = document.getElementById("ai-status");
  if (div) {
    div.textContent = `🔄 Analisi aggiornata alle ${orario}`;
  } else {
    const newDiv = document.createElement("div");
    newDiv.id = "ai-status";
    newDiv.style = "margin-top:5px; font-size: 0.9em; color: gray;";
    newDiv.textContent = `🔄 Analisi aggiornata alle ${orario}`;
    document.getElementById("report-ai")?.appendChild(newDiv);
  }
}

function mostraBadgeNuoviCavalli(nomi, gare) {
  const storici = gare.flatMap(g => g.nomi);
  document.querySelectorAll("#tabella-gara-1 input.nome").forEach(input => {
    if (!input.value) return;
    const esiste = storici.includes(input.value.trim());
    input.style.border = esiste ? "" : "2px solid orange";
    input.title = esiste ? "" : "🆕 Cavallo mai visto prima nello storico!";
  });
}

function mostraTooltipQuote(quote, gare) {
  const quoteMap = {};

  gare.forEach(g => {
    g.quote.forEach((q, i) => {
      const val = parseFloat(q);
      if (!quoteMap[val]) quoteMap[val] = { podi: 0, tot: 0 };

      const corsiePodio = g.tris.flatMap(t => t.combinazione.split(",").map(n => parseInt(n)));
      if (corsiePodio.includes(i + 1)) quoteMap[val].podi++;
      quoteMap[val].tot++;
    });
  });

  document.querySelectorAll("#tabella-gara-1 input.quota").forEach(input => {
    const val = parseFloat(input.value);
    if (!val || !quoteMap[val]) return;

    const data = quoteMap[val];
    const perc = ((data.podi / data.tot) * 100).toFixed(1);
    input.title = `📊 Quota ${val} → ${perc}% podio su ${data.tot} casi`;
  });
}
</script>
</body>
</html>
