<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Archivio Gare Cavalli</title>

  <!-- PWA essentials -->
  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#007bff">
  <link rel="icon" href="icons/icon-192.png">

  <!-- Mobile app compatibility (opzionale ma utile) -->
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

  <!-- Style -->
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

  <!-- Service Worker registration -->
  <script>
    if ('serviceWorker' in navigator) {
      window.addEventListener('load', () => {
        navigator.serviceWorker.register('service-worker.js')
          .then(reg => console.log('✅ Service Worker registrato:', reg.scope))
          .catch(err => console.error('❌ Errore nella registrazione SW:', err));
      });
    }
  </script>
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
const datiStoriciPerCorsia = {
  1: {
    vittorie: [2.1, 3.0, 4.5],
    podi: [2.0, 3.0, 4.0, 5.0]
  },
  2: {
    vittorie: [2.3, 3.2, 4.1],
    podi: [2.5, 3.5, 5.0]
  },
  3: {
    vittorie: [1.9, 2.6, 3.8],
    podi: [2.0, 3.2, 4.5]
  },
  4: {
    vittorie: [3.5, 4.2, 5.1],
    podi: [3.8, 4.5, 6.0]
  },
  5: {
    vittorie: [4.8, 5.5, 6.2],
    podi: [5.0, 5.5, 6.8]
  },
  6: {
    vittorie: [5.5, 6.2, 7.0],
    podi: [5.8, 6.5, 7.2]
  }
};

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
  const gareStessaQuota = gare.filter(g =>
    JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) === quoteStr &&
    JSON.stringify(g.nomi) !== nomiStr
  );
  if (gareStessaQuota.length > 0) {
    let msg = `⚠️ Questa combinazione di quote è già presente in ${gareStessaQuota.length} gara/e con cavalli diversi.\n`;
    gareStessaQuota.forEach((g, i) => {
      msg += `\nGara ${i + 1} → Tris vincenti:\n${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n")}`;
    });
    alert(msg);
  }

  // Nomi uguali, quote diverse
  const gareStessiNomi = gare.filter(g =>
    JSON.stringify(g.nomi) === nomiStr &&
    JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) !== quoteStr
  );
  if (gareStessiNomi.length > 0) {
    let msg = `⚠️ Esiste già una gara con gli stessi cavalli ma quote differenti:\n`;
    gareStessiNomi.forEach((g, i) => {
      msg += `\nGara ${i + 1} → Quote: ${g.quote.join(", ")}\nTris:\n${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n")}`;
    });
    if (!confirm(msg + `\n\nVuoi salvare comunque?`)) return;
  }

  let reportAI = null;

  try {
    const cavalli = nomi.map((nome, i) => ({
      nome,
      quota: parseFloat(quote[i]),
      corsia: i + 1
    }));
    reportAI = analisiAIAvanzata(cavalli, quote, gare);
    mostraReport(reportAI);

    if (!confirm("Vuoi procedere con il salvataggio della gara dopo l’analisi AI?")) {
      return;
    }
  } catch (err) {
    alert("❌ Errore durante l'analisi AI: " + err.message);
    console.error(err);
    return;
  }

  // Gara identica già salvata
  const garaEsatta = gare.find(g =>
    JSON.stringify(g.nomi) === nomiStr &&
    JSON.stringify(g.quote.map(q => parseFloat(q).toFixed(2))) === quoteStr
  );

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
  alert("✅ Gara salvata con successo.");
}

function analisiAIAvanzata(cavalli, trisVincentiStoriche, analisiStoriche) {
  let report = "";
  if (!cavalli || cavalli.length < 6) return "❌ Dati incompleti per l'analisi AI.";

  const quote = cavalli.map(c => parseFloat(c.quota));
  const sommaQuote = quote.reduce((a, b) => a + b, 0);
  const patternQuoteTotale = quote.map(q => {
    if (q <= 2.5) return "B";
    if (q <= 6.5) return "M";
    if (q <= 9.9) return "A";
    return "SA";
  });
  const patternString = patternQuoteTotale.join("-");

  report += `📊 Pattern quote: ${patternString}\n🧮 Somma quote: ${sommaQuote.toFixed(2)}`;

  // Favorito e sfavorito
  const minQuota = Math.min(...quote);
  const maxQuota = Math.max(...quote);
  const favorito = cavalli.find(c => Math.abs(c.quota - minQuota) < 0.01);
  const sfavorito = cavalli.find(c => Math.abs(c.quota - maxQuota) < 0.01);

  if (!favorito || !sfavorito) return "❌ Errore nell'identificazione di favorito/sfavorito.";

  function storicoPodio(c) {
    const corsia = c.corsia;
    const quota = parseFloat(c.quota);
    const key = `${corsia}|${quota}`;
    const data = analisiStoriche[key] || { podi: 0, gare: 0, vittorie: 0 };
    const percPodi = data.gare > 0 ? (data.podi / data.gare) * 100 : 0;
    const percVittorie = data.gare > 0 ? (data.vittorie / data.gare) * 100 : 0;
    return {
      podi: data.podi,
      gare: data.gare,
      vittorie: data.vittorie,
      percPodi: percPodi.toFixed(1),
      percVittorie: percVittorie.toFixed(1)
    };
  }

  const storicoFavorito = storicoPodio(favorito);
  const storicoSfavorito = storicoPodio(sfavorito);

  report += `\n\n🏇 Cavallo favorito: ${favorito.nome} (Corsia ${favorito.corsia}, Quota ${favorito.quota})`;
  report += `\n📊 Storico: ${storicoFavorito.podi} podi / ${storicoFavorito.vittorie} vittorie su ${storicoFavorito.gare} gare → ${storicoFavorito.percPodi}% podio, ${storicoFavorito.percVittorie}% vittorie`;

  report += `\n\n🐢 Cavallo sfavorito: ${sfavorito.nome} (Corsia ${sfavorito.corsia}, Quota ${sfavorito.quota})`;
  report += `\n📊 Storico: ${storicoSfavorito.podi} podi / ${storicoSfavorito.vittorie} vittorie su ${storicoSfavorito.gare} gare → ${storicoSfavorito.percPodi}% podio, ${storicoSfavorito.percVittorie}% vittorie`;

  // Analisi quote + corsia
  report += `\n\n📌 Quote e corsie dettagliate:\n`;
  cavalli.forEach(c => {
    const q = parseFloat(c.quota);
    const key = `${c.corsia}|${q}`;
    const storico = analisiStoriche[key] || { podi: 0, gare: 0, vittorie: 0 };
    const podiPerc = storico.gare > 0 ? ((storico.podi / storico.gare) * 100).toFixed(1) : 0;
    const vittPerc = storico.gare > 0 ? ((storico.vittorie / storico.gare) * 100).toFixed(1) : 0;
    const extra = analizzaQuotaECorsia(c.corsia, q);
    const podioTest = analisiPodioPerQuotaECorsia(c.corsia, q);
    report += `→ Corsia ${c.corsia}, Quota ${q}:\n   ${extra}\n   🎯 ${podioTest}\n   📊 Storico: ${storico.podi} podi / ${storico.vittorie} vittorie su ${storico.gare} gare (${podiPerc}% podio, ${vittPerc}% vittorie)\n`;
  });

  // Tris AI
  let classificazione = "";
  const tris = cavalli.slice().sort((a, b) => a.quota - b.quota).slice(0, 3);
  if (tris.length >= 3) {
    const sommaTris = tris.reduce((sum, c) => sum + c.quota, 0);
    const patternTris = tris.map(c => quotaPattern(c.quota));
    if (sommaTris <= 9) classificazione = "💎 favorita";
    else if (sommaTris <= 16) classificazione = "✅ bilanciata";
    else if (sommaTris <= 25) classificazione = "⚠️ rischiosa";
    else classificazione = "☠️ tossica";

    const trisString = tris.map(c => c.corsia).sort((a, b) => a - b).join(",");
    const clusterFrequenti = ["1,2,3", "1,3,5", "2,3,4", "3,4,6", "2,4,5", "1,4,6", "1,5,6"];
    const clusterNote = clusterFrequenti.includes(trisString) ? `\n🔁 Tris AI compatibile con cluster vincente frequente (${trisString})` : "";

    report += `\n\n🤖 Predizione Tris AI: ${tris.map(c => c.corsia).join("-")} (Pattern: ${patternTris.join("-")}, Somma quote: ${sommaTris.toFixed(2)}) → ${classificazione}${clusterNote}`;
  } else {
    report += `\n\n🤖 Predizione Tris AI: Non abbastanza dati per una predizione affidabile.`;
  }

  // Sintesi
  report += `\n\n✨ Sintesi:`;
  if (storicoFavorito.vittorie == 0) report += ` ⚠️ Attenzione: favorito solo teorico.`;
  if (storicoSfavorito.vittorie > 0 || storicoSfavorito.podi > 0) report += ` 🎯 Possibile sorpresa dello sfavorito.`;
  if (classificazione.includes("tossica") || classificazione.includes("rischiosa")) {
    report += ` 🚨 Tris AI a rischio: ${classificazione}`;
  } else if (classificazione) {
    report += ` ✅ Tris AI considerata: ${classificazione}`;
  }

  return report;
}
function quotaPattern(quota) {
  if (quota <= 2.5) return "B";    // Bassa
  if (quota <= 6.5) return "M";    // Media
  if (quota <= 9.9) return "A";    // Alta
  return "SA";                     // Super Alta
}
function analisiPodioPerQuotaECorsia(corsia, quota) {
  const storico = datiStoriciPerCorsia[corsia];
  if (!storico || !storico.podi || storico.podi.length === 0) return "Nessun dato podio.";

  const rilevanti = storico.podi.filter(q => Math.abs(q - quota) <= 0.5);
  const totale = storico.podi.length;
  const percentuale = ((rilevanti.length / totale) * 100).toFixed(1);

  if (rilevanti.length === 0) return `🚫 Mai a podio con quota simile (${quota})`;
  return `👍 ${rilevanti.length} podi su ${totale} (${percentuale}%) con quote simili a ${quota}`;
}

function analizzaQuotaECorsia(corsia, quota) {
  const datiStorici = {
    1: { vincenti: [2.0, 5.0], podio: [2.0, 7.0], tossiche: [7.5, 9.0] },
    2: { vincenti: [2.0, 4.0], podio: [2.0, 8.0], tossiche: [25.0, 40.0] },
    3: { vincenti: [1.5, 3.0], podio: [1.5, 6.0], tossiche: [10.0, 15.0] },
    4: { vincenti: [3.5, 5.5], podio: [3.0, 7.0], tossiche: [8.5, 10.0] },
    5: { vincenti: [5.0, 6.5], podio: [4.0, 7.0], tossiche: [12.0, 20.0] },
    6: { vincenti: [3.5, 6.5], podio: [3.0, 7.5], tossiche: [15.0, 30.0] },
  };

  const d = datiStorici[corsia];
  if (!d) return "❓ Nessun dato storico per questa corsia.";

  let messaggio = "";

  const inVincente = quota >= d.vincenti[0] && quota <= d.vincenti[1];
  const inPodio = quota >= d.podio[0] && quota <= d.podio[1];
  const inTossico = quota >= d.tossiche[0] && quota <= d.tossiche[1];

  if (inVincente) messaggio += "🎯 range vincente ";
  if (inPodio) messaggio += "👍 range podio ";
  if (inTossico) messaggio += "⚠️ range tossico";

  if (!inVincente && !inPodio && !inTossico) messaggio = "🚫 quota fuori da tutti i range noti";

  return messaggio.trim();
}
function analisiStoricaPerCorsia(corsia, quota, tipo = "vittorie") {
  const storico = datiStoriciPerCorsia[corsia];
  if (!storico) return "❌ Nessun dato storico per questa corsia.";

  const target = tipo === "vittorie" ? storico.vittorie : storico.podi;
  if (!target || target.length === 0) return `❌ Nessun dato disponibile per ${tipo}.`;

  const rilevanti = target.filter(q => Math.abs(q - quota) <= 0.5);
  const totale = target.length;

  if (rilevanti.length === 0) {
    return `0 ${tipo} su ${totale} (quota ${quota} mai ${tipo === "vittorie" ? "vincente" : "a podio"})`;
  }

  const percentuale = ((rilevanti.length / totale) * 100).toFixed(1);
  return `${rilevanti.length} ${tipo} su ${totale} (${percentuale}%) con quote simili a ${quota}`;
}
function caricaStoricoCavalli() {
  const dati = {
    "shadowfax": {
      primi: 3,
      secondi: 2,
      terzi: 4,
      podi: 9,
      totali: 15,
      quotaMedia: 4.5,
      quotePodi: [3.0, 4.2, 5.1, 2.8, 6.0, 3.3, 5.5, 4.8, 3.7]
    },
    "tornado": {
      primi: 1,
      secondi: 0,
      terzi: 2,
      podi: 3,
      totali: 12,
      quotaMedia: 7.8,
      quotePodi: [7.5, 8.0, 6.9]
    },
    // ...altri cavalli
  };

  // Assicura che ogni cavallo abbia quotePodi calcolate anche se mancano
  for (const nome in dati) {
    const c = dati[nome];
    if (!c.quotePodi || c.quotePodi.length === 0) {
      c.quotePodi = [];
    }
  }

  return dati;
}
function suggerisciTrisAI(cavalli) {
  // Filtra cavalli con quote troppo alte (super tossiche > 30)
  const filtrati = cavalli.filter(c => c.quota <= 30);

  // Ordina per quota crescente
  const ordinati = filtrati.sort((a, b) => a.quota - b.quota);

  // BONUS: se ci sono almeno 4 cavalli, cerca quelli distribuiti su corsie diverse
  let tris = ordinati.slice(0, 3);

  // Verifica che non siano tutte quote super basse (tipo 1.2, 1.5, 1.7) → evita tris sbilanciata
  const somma = tris.reduce((tot, c) => tot + c.quota, 0);
  const media = somma / tris.length;

  // Se media troppo bassa (<2.0), rimpiazza con almeno una quota più equilibrata (media o alta)
  if (media < 2.0 && ordinati.length > 3) {
    const bilanciata = ordinati.find(c => c.quota >= 2.5 && c.quota <= 6.5);
    if (bilanciata && !tris.some(c => c.nome === bilanciata.nome)) {
      tris[2] = bilanciata;
    }
  }

  return tris;
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
    }); // ✅ CHIUSURA MANCANTE AGGIUNTA QUI

    // Dopo la compilazione vocale, lancia subito l'analisi AI
    const dati = getGaraData(1);
    const gare = JSON.parse(localStorage.getItem("gare") || "[]");
    const cavalli = dati.nomi.map((nome, i) => ({
      nome,
      quota: parseFloat(dati.quote[i]),
      corsia: i + 1
    }));
    const reportAI = analisiAIAvanzata(cavalli, dati.quote, gare);
    mostraReport(reportAI);
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
    mostraReport(reportAI);
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
    const cavalli = nomi.map((nome, i) => ({
      nome,
      quota: parseFloat(quote[i]),
      corsia: i + 1
    }));
    const reportAI = analisiAIAvanzata(cavalli, quote, gare);
    mostraReport(reportAI);

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
