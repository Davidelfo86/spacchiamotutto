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
    if (q < 2) return "B";
    if (q <= 3.5) return "M";
    return "A";
  });

  let report = `🧠 ANALISI INTELLIGENTE\n----------------------\n`;
  report += `📊 Pattern quote: ${patternLabels.join("-")}\n`;
  report += `🧮 Somma quote: ${sommaQuote.toFixed(2)}\n\n`;

  let trisSuggerita = [];
  let quoteStatistiche = {};
  let patternSimili = [], quoteSimili = [];
  let cavalliGlobale = {}, cavalliCorsia = {}, combinazioniTris = {}, trisCluster = {};

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

    g.nomi.forEach((n, i) => {
      if (!nomi.includes(n)) return; // <-- solo cavalli attuali
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

  const minQuota = Math.min(...nuoveQuote);
  const maxQuota = Math.max(...nuoveQuote);
  const favoritoIdx = nuoveQuote.indexOf(minQuota);
  const sfavoritoIdx = nuoveQuote.indexOf(maxQuota);

  report += `🏇 Cavallo favorito: ${nomi[favoritoIdx]} (Corsia ${favoritoIdx + 1}, Quota ${minQuota})\n`;
  report += `🐢 Cavallo sfavorito: ${nomi[sfavoritoIdx]} (Corsia ${sfavoritoIdx + 1}, Quota ${maxQuota})\n`;

  const storicoSfavorito = gare.filter(g => g.nomi.includes(nomi[sfavoritoIdx]));
  let sorprese = 0;
  storicoSfavorito.forEach(g => {
    g.tris.forEach(t => {
      const corsie = t.combinazione.split(",");
      const idx = g.nomi.findIndex(n => n === nomi[sfavoritoIdx]);
      if (idx !== -1 && corsie[0] === String(idx + 1)) sorprese++;
    });
  });
  if (sorprese > 0) {
    report += `🎯 Sorpresa: ${sorprese} vittorie del cavallo sfavorito storico!\n`;
  }

  report += `\n📌 Cavalli ricorrenti (storico della gara attuale):\n`;
  Object.entries(cavalliGlobale).forEach(([nome, stats]) => {
    if (stats.tot >= minGareAnalisi) {
      const perc = (stats.podio / stats.tot) * 100;
      const avg = stats.quote.reduce((a, b) => a + b, 0) / stats.quote.length;
      report += `→ ${nome}: ${perc.toFixed(1)}% podio su ${stats.tot} gare | Quota media: ${avg.toFixed(2)}\n`;
      trisSuggerita.push({ nome, perc, corsie: stats.corsie });
    }
  });

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

  report += `\n📌 Cluster di tris vincenti ricorrenti:\n`;
  Object.entries(trisCluster).filter(([k, v]) => v > 1).forEach(([k, v]) => {
    report += `→ Combinazione ${k.replace(/-/g, ",")}: ${v} volte\n`;
  });

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
  if (!('webkitSpeechRecognition' in window)) {
    alert("Il tuo browser non supporta la dettatura vocale (funziona solo su Chrome).");
    return;
  }

  const nomiMancanti = [];
  const quoteMancanti = [];
  for (let i = 1; i <= 6; i++) {
    const nomeInput = document.getElementById(`nome1_${i}`);
    const quotaInput = document.getElementById(`quota1_${i}`);
    if (nomeInput.value.trim() === "" || quotaInput.value.trim() === "") {
      nomiMancanti.push(nomeInput);
      quoteMancanti.push(quotaInput);
    }
  }

  if (nomiMancanti.length === 0) {
    alert("Hai già inserito tutti e 6 i cavalli. Pulisci la tabella o modifica manualmente.");
    return;
  }

  const recognition = new webkitSpeechRecognition();
  recognition.lang = "it-IT";
  recognition.continuous = false;
  recognition.interimResults = false;

  recognition.onstart = () =>
    alert(`🎙️ Inizia a parlare: nome + quota (es. 'Marcello 4.5') – Cavalli mancanti: ${nomiMancanti.length}`);

  recognition.onerror = (event) => alert("Errore nella dettatura: " + event.error);

  recognition.onresult = function(event) {
    const transcript = event.results[0][0].transcript;
    const parole = transcript.trim().split(/\s+/);

    const nomi = [];
    const quote = [];

    for (let i = 0; i < parole.length - 1; i++) {
      const possibileQuota = parseFloat(parole[i + 1].replace(",", "."));
      if (!isNaN(possibileQuota)) {
        nomi.push(parole[i]);
        quote.push(possibileQuota);
        i++; // salta la quota già usata
      }
    }

    const daInserire = Math.min(nomi.length, nomiMancanti.length);

    if (daInserire === 0) {
      alert("⚠️ Nessuna combinazione nome + quota valida riconosciuta.");
      return;
    }

    for (let i = 0; i < daInserire; i++) {
      nomiMancanti[i].value = nomi[i];
      quoteMancanti[i].value = quote[i];
    }

    alert(`✅ Inseriti ${daInserire} cavalli. Ne mancano ancora ${6 - (6 - nomiMancanti.length + daInserire)}.`);
  };

  recognition.start();
}
function startAIWatcher() {
  let ultimaFirma = "";

  // Analisi iniziale immediata (se i dati sono già compilati)
  const iniziale = getGaraData(1);
  if (!iniziale.nomi.includes("") && !iniziale.quote.includes("")) {
    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const reportAI = analisiAIAvanzata(iniziale.nomi, iniziale.quote, gare);
    mostraReport(reportAI.reportTesto);
    ultimaFirma = iniziale.nomi.join("|") + "::" + iniziale.quote.join("|");
  }

  // Avvia il watcher ogni secondo
  setInterval(() => {
    const { nomi, quote } = getGaraData(1);

    // Se non sono completi i dati, esci
    if (nomi.includes("") || quote.includes("")) return;

    // Firma attuale della gara (per evitare analisi duplicate)
    const firmaAttuale = nomi.join("|") + "::" + quote.join("|");

    // Se la firma è uguale alla precedente, non rifare analisi
    if (firmaAttuale === ultimaFirma) return;

    // Nuova analisi
    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const reportAI = analisiAIAvanzata(nomi, quote, gare);
    mostraReport(reportAI.reportTesto);

    ultimaFirma = firmaAttuale;
  }, 1000); // ogni 1 secondo
}
</script>
</body>
</html>
