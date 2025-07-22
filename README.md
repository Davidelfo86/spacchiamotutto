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
            display: flex;
            flex-direction: column;
            gap: 50px;
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
    </style>
</head>
<body>
<h2>Archivio Gare Cavalli</h2>
<div class="gara-container">
    <div class="gara">
        <table id="gara1">
            <tr><th>Corsia</th><th>Nome</th><th>Quota</th></tr>
            <tbody id="body1"></tbody>
        </table>
        <button onclick="salvaGara(1)">Salva</button>
        <button onclick="pulisciTabella(1)">Pulisci Tabella</button>
        <button onclick="cercaGare()">Cerca</button>
    </div>
    <div class="gara">
        <table id="gara2">
            <tr><th>Corsia</th><th>Nome</th><th>Quota</th></tr>
            <tbody id="body2"></tbody>
        </table>
        <button onclick="salvaGara(2)">Salva</button>
        <button onclick="pulisciTabella(2)">Pulisci Tabella</button>
    </div>
</div>

<button onclick="cancellaTutto()">Cancella Tutto (Storage)</button>
<button onclick="mostraTutteGare()">Mostra tutte le gare salvate</button>

<div id="report"></div>

<script>
    // Genera righe con input dinamicamente
    for (let j = 1; j <= 2; j++) {
        const tbody = document.getElementById(`body${j}`);
        for (let i = 1; i <= 6; i++) {
            tbody.innerHTML += `
                <tr>
                    <td>${i}</td>
                    <td><input type="text" id="nome${j}_${i}" class="nome" autocomplete="off" /></td>
                    <td><input type="number" inputmode="decimal" id="quota${j}_${i}" class="quota" maxlength="4" /></td>
                </tr>
            `;
        }
    }

    function getGaraData(index) {
        const nomi = [], quote = [];
        for (let i = 1; i <= 6; i++) {
            nomi.push(document.getElementById(`nome${index}_${i}`).value.trim());
            quote.push(document.getElementById(`quota${index}_${i}`).value.trim());
        }
        return { nomi, quote };
    }

    function salvaGara(index) {
        const { nomi, quote } = getGaraData(index);
        if (nomi.includes("") || quote.includes("")) {
            alert("Completa tutti i campi prima di salvare.");
            return;
        }

        let gare = JSON.parse(localStorage.getItem("gare") || "[]");
        // Cerco gara con nomi e quote uguali
        const garaEsistente = gare.find(g => JSON.stringify(g.nomi) === JSON.stringify(nomi) && JSON.stringify(g.quote) === JSON.stringify(quote));
        if (garaEsistente) {
            const trisElenco = garaEsistente.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n");
            alert(`Questa gara esiste già e ha queste tris vincenti:\n${trisElenco}`);

            if (confirm("Vuoi aggiungere una nuova tris vincente diversa?")) {
                inserisciNuovaTris(garaEsistente);
            }
            return;
        }

        // Cerco gare con quote uguali ma nomi diversi
        const gareStesseQuote = gare.filter(g => JSON.stringify(g.quote) === JSON.stringify(quote));
        if (gareStesseQuote.length > 0) {
            let messaggio = "⚠️ Attenzione: queste quote sono già presenti in altre gare con nomi differenti.\nEcco le tris vincenti associate:\n";
            gareStesseQuote.forEach(g => {
                messaggio += `Tris: ${g.tris.map(t => t.combinazione + " (Quota: " + t.quota + ")").join(" | ")}\n`;
            });
            alert(messaggio);
            inserisciNuovaTris(null, nomi, quote);
            return;
        }

        // Gara nuova con quote nuove: faccio analisi AI avanzata
        const reportAI = analisiAIAvanzata(nomi, quote, gare);

        if (reportAI.alertMessage) {
            if (!confirm(reportAI.alertMessage + "\n\nVuoi comunque salvare la gara?")) {
                return;
            }
        }

        // Se qui, salvo gara (con tris AI suggerita o senza)
        if (reportAI.suggerimentoTris) {
            if (confirm(`🧠 Suggerimento AI: La tris più frequente tra gare simili è ${reportAI.suggerimentoTris.combinazione} (Quota media: ${reportAI.suggerimentoTris.quota})\nVuoi salvarla come tris vincente?`)) {
                const nuovaGara = { nomi, quote, tris: [{ combinazione: reportAI.suggerimentoTris.combinazione, quota: reportAI.suggerimentoTris.quota }] };
                gare.push(nuovaGara);
                localStorage.setItem("gare", JSON.stringify(gare));
                alert("Gara salvata con tris AI.");
                mostraReport(reportAI.reportTesto);
                return;
            }
        }

        inserisciNuovaTris(null, nomi, quote);
        mostraReport(reportAI.reportTesto);
    }

    function inserisciNuovaTris(gara, nomi = null, quote = null) {
        const trisInput = prompt("Inserire tris vincente (es. 1,3,5)");
        if (!trisInput) return;

        const trisArray = trisInput.split(",").map(s => s.trim()).filter(n => !isNaN(n) && n !== "");
        if (trisArray.length !== 3) {
            alert("La tris vincente deve essere formata da 3 numeri.");
            return;
        }

        const quotaTris = prompt("Inserire quota della tris vincente (es. 18.5)");
        if (!quotaTris || isNaN(quotaTris)) {
            alert("Quota non valida.");
            return;
        }

        const trisStr = trisArray.join(",");

        const gare = JSON.parse(localStorage.getItem("gare") || "[]");

        if (!gara) {
            const nuovaGara = {
                nomi,
                quote,
                tris: [{ combinazione: trisStr, quota: quotaTris }]
            };
            gare.push(nuovaGara);
        } else {
            const idx = gare.findIndex(g => JSON.stringify(g.nomi) === JSON.stringify(gara.nomi) && JSON.stringify(g.quote) === JSON.stringify(gara.quote));
            if (idx === -1) {
                alert("Errore: gara non trovata nello storage.");
                return;
            }
            if (!gare[idx].tris.some(t => t.combinazione === trisStr)) {
                gare[idx].tris.push({ combinazione: trisStr, quota: quotaTris });
            } else {
                alert("Questa tris è già salvata per questa gara.");
                return;
            }
        }

        localStorage.setItem("gare", JSON.stringify(gare));
        alert("Gara salvata correttamente.");
    }

    function pulisciTabella(index) {
        if (confirm("Sei sicuro di voler pulire la tabella? I dati non salvati andranno persi.")) {
            for (let i = 1; i <= 6; i++) {
                document.getElementById(`nome${index}_${i}`).value = "";
                document.getElementById(`quota${index}_${i}`).value = "";
            }
        }
    }

    function cancellaTutto() {
        if (confirm("Sei sicuro di voler eliminare tutte le gare salvate?")) {
            localStorage.removeItem("gare");
            alert("Tutte le gare sono state eliminate.");
            document.getElementById("report").textContent = "";
        }
    }

    function cercaGare() {
        const cavallo = document.getElementById("nome1_1").value.trim();
        if (!cavallo) {
            alert("Inserisci un nome nella prima corsia per cercare.");
            return;
        }

        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        const risultati = gare.filter(g => g.nomi[0].toLowerCase() === cavallo.toLowerCase());

        if (risultati.length === 0) {
            alert("Nessuna gara trovata con quel cavallo in corsia 1.");
            return;
        }

        let index = 0;
        const win = window.open("", "Risultati Ricerca", "width=600,height=400");
        const containerId = `contenitore-${Date.now()}`;

        function mostraGara(i) {
            const gara = risultati[i];
            win.document.body.innerHTML = `<div id="${containerId}">
                <h3>Gara ${i + 1} di ${risultati.length}</h3>
                <ul>${gara.nomi.map((n, idx) => `<li>Corsia ${idx + 1}: ${n} (Quota: ${gara.quote[idx]})</li>`).join("")}</ul>
                <p><strong>Tris vincenti:</strong><br> ${gara.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("<br>")}</p>
                <button onclick="window.opener.prevGara()">&larr;</button>
                <button onclick="window.opener.nextGara()">&rarr;</button>
            </div>`;
        }

        window.prevGara = () => {
            if (index > 0) index--;
            mostraGara(index);
        };

        window.nextGara = () => {
            if (index < risultati.length - 1) index++;
            mostraGara(index);
        };

        mostraGara(index);
    }

    function mostraTutteGare() {
        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        if (gare.length === 0) {
            alert("Nessuna gara salvata.");
            return;
        }

        let html = `<h2>Tutte le gare salvate (${gare.length})</h2>`;
        gare.forEach((g, idx) => {
            html += `<h3>Gara ${idx + 1}</h3><ul>`;
            g.nomi.forEach((nome, i) => {
                html += `<li><strong>Corsia ${i + 1}:</strong> ${nome} (Quota: ${g.quote[i]})</li>`;
            });
            html += `</ul><p><strong>Tris vincenti:</strong><br>${g.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("<br>")}</p><hr>`;
        });

        const win = window.open("", "Tutte le gare salvate", "width=600,height=600,scrollbars=yes");
        win.document.body.style.fontFamily = "Arial, sans-serif";
        win.document.body.style.padding = "10px";
        win.document.body.innerHTML = html;
    }

    // Autocomplete nomi cavalli
    function setupAutocomplete() {
        const inputs = document.querySelectorAll("input.nome");
        const cavalliSalvati = new Set();

        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        gare.forEach(g => {
            g.nomi.forEach(nome => {
                if (nome.trim()) cavalliSalvati.add(nome.trim());
            });
        });

        inputs.forEach(input => {
            input.addEventListener("input", function () {
                closeAllLists();
                const val = this.value;
                if (!val) return;

                const list = document.createElement("div");
                list.setAttribute("class", "autocomplete-items");
                this.parentNode.appendChild(list);

                [...cavalliSalvati].forEach(nome => {
                    if (nome.toLowerCase().startsWith(val.toLowerCase())) {
                        const item = document.createElement("div");
                        item.innerHTML = "<strong>" + nome.substr(0, val.length) + "</strong>" + nome.substr(val.length);
                        item.innerHTML += `<input type='hidden' value='${nome}'>`;
                        item.addEventListener("click", () => {
                            input.value = nome;
                            closeAllLists();
                        });
                        list.appendChild(item);
                    }
                });
            });

            input.addEventListener("blur", () => setTimeout(closeAllLists, 100));
        });

        function closeAllLists() {
            const lists = document.querySelectorAll(".autocomplete-items");
            lists.forEach(l => l.remove());
        }
    }

    window.addEventListener("DOMContentLoaded", setupAutocomplete);

    // --- Funzione di analisi AI avanzata ---
    function analisiAIAvanzata(nomi, quote, gare) {
        const nuoveQuote = quote.map(q => parseFloat(q));
        const sogliaQuota = 0.1; // ±0.1 per range quote simili
        const sogliaFreq = 0.6;   // 60%
        const minGare = 3;        // Minimo gare per considerare pattern

        let reportLines = [];
        let alertMessages = [];
        let suggerimentoTris = null;

        // 1) Cavallo + quota che entra in tris almeno 60% delle volte in corsia specifica
        for (let i = 0; i < 6; i++) {
            const nomeCavallo = nomi[i];
            const quotaCavallo = nuoveQuote[i];
            if (!nomeCavallo || isNaN(quotaCavallo)) continue;

            let tot = 0;
            let countInTris = 0;

            gare.forEach(g => {
                if (g.nomi[i] === nomeCavallo && Math.abs(parseFloat(g.quote[i]) - quotaCavallo) < sogliaQuota) {
                    tot++;
                    // La tris è una stringa "1,3,5" o simile, controllo se la corsia (i+1) è nella tris
                    if (g.tris.some(t => t.combinazione.split(',').includes(String(i + 1)))) {
                        countInTris++;
                    }
                }
            });

            if (tot >= minGare && (countInTris / tot) >= sogliaFreq) {
                reportLines.push(`✔️ Il cavallo "${nomeCavallo}" in corsia ${i + 1} con quota ~${quotaCavallo.toFixed(2)} entra in tris vincente nel ${(countInTris / tot * 100).toFixed(1)}% delle ${tot} gare trovate.`);
                alertMessages.push(`Cavallo "${nomeCavallo}" corsia ${i + 1} con quota ~${quotaCavallo.toFixed(2)} frequente in tris!`);
            }
        }

        // 2) Quote simili in una corsia che fanno vincere altra corsia in tris con frequenza >=60%
        for (let i = 0; i < 6; i++) {
            const quotaCorrente = nuoveQuote[i];
            if (isNaN(quotaCorrente)) continue;

            // Per ogni gara e sua tris, vedo se la quota in corsia i è simile e quali corsie sono nella tris
            let totaleSimili = 0;
            const vinciteCorsie = {};

            gare.forEach(g => {
                if (Math.abs(parseFloat(g.quote[i]) - quotaCorrente) < sogliaQuota) {
                    totaleSimili++;
                    g.tris.forEach(t => {
                        const corsie = t.combinazione.split(',').map(x => parseInt(x));
                        corsie.forEach(c => {
                            vinciteCorsie[c] = (vinciteCorsie[c] || 0) + 1;
                        });
                    });
                }
            });

            if (totaleSimili >= minGare) {
                for (const corsia in vinciteCorsie) {
                    const freq = vinciteCorsie[corsia] / totaleSimili;
                    if (freq >= sogliaFreq) {
                        reportLines.push(`✔️ Quote simili in corsia ${i + 1} associano vittoria frequente a corsia ${corsia} con frequenza ${(freq * 100).toFixed(1)}%.`);
                        alertMessages.push(`Quote corsia ${i + 1} suggeriscono vittoria corsia ${corsia}`);
                    }
                }
            }
        }

        // 3) Coppie di cavalli che insieme sono associate a vincita in corsia (≥60%)
        // Costruisco mappa coppie -> {tot, count vincita corsia x}
        const coppieMap = {}; // chiave: "nome1|nome2" valori: { tot: n, vincite: { corsia: count } }

        // Estraggo tutte le coppie di cavalli presenti nella gara nuova
        for (let a = 0; a < 6; a++) {
            for (let b = a + 1; b < 6; b++) {
                const nomeA = nomi[a];
                const nomeB = nomi[b];
                if (!nomeA || !nomeB) continue;

                const coppiaKey = [nomeA, nomeB].sort().join('|');
                if (!coppieMap[coppiaKey]) {
                    coppieMap[coppiaKey] = { tot: 0, vincite: {} };
                }

                gare.forEach(g => {
                    // Controllo se la gara contiene la coppia (in qualsiasi posizione)
                    const nomiGara = g.nomi;
                    const hasA = nomiGara.includes(nomeA);
                    const hasB = nomiGara.includes(nomeB);
                    if (hasA && hasB) {
                        coppieMap[coppiaKey].tot++;

                        g.tris.forEach(t => {
                            const corsie = t.combinazione.split(',').map(x => parseInt(x));
                            corsie.forEach(c => {
                                coppieMap[coppiaKey].vincite[c] = (coppieMap[coppiaKey].vincite[c] || 0) + 1;
                            });
                        });
                    }
                });
            }
        }

        // Valuto quali coppie raggiungono la soglia di frequenza per vincita corsia
        for (const coppia in coppieMap) {
            const dati = coppieMap[coppia];
            if (dati.tot >= minGare) {
                for (const corsia in dati.vincite) {
                    const freq = dati.vincite[corsia] / dati.tot;
                    if (freq >= sogliaFreq) {
                        reportLines.push(`✔️ Coppia cavalli [${coppia.replace('|', ', ')}] associata a vincita corsia ${corsia} nel ${(freq * 100).toFixed(1)}% delle ${dati.tot} gare.`);
                        alertMessages.push(`Coppia [${coppia.replace('|', ', ')}] spesso vince in corsia ${corsia}`);
                    }
                }
            }
        }

        // 4) Suggerimento tris (la stessa che avevi)
        const trisSuggerita = suggerisciTris(nomi, nuoveQuote, gare);

        if (trisSuggerita) {
            suggerimentoTris = trisSuggerita;
            reportLines.push(`🧠 Suggerimento tris AI più frequente tra gare simili: ${trisSuggerita.combinazione} (Quota media: ${trisSuggerita.quota})`);
        } else {
            reportLines.push("Nessun suggerimento tris AI disponibile (poche gare simili trovate).");
        }

        const alertMessage = alertMessages.length > 0 ? alertMessages.join('\n') : null;
        const reportTesto = reportLines.join('\n');

        return { alertMessage, suggerimentoTris, reportTesto };
    }

    // Funzione usata per suggerire tris come prima, ricavata dal tuo codice
    function suggerisciTris(nomi, quote, gare) {
        const soglia = 0.2;
        // Cerca gare con quote simili (almeno 4 corsie)
        const gareSimili = gare.filter(g => {
            let match = 0;
            for (let i = 0; i < 6; i++) {
                const q = parseFloat(g.quote[i]);
                if (Math.abs(q - quote[i]) <= soglia) match++;
            }
            return match >= 4;
        });

        if (gareSimili.length === 0) return null;

        // Conta tris vincenti tra gare simili
        const trisMap = {};
        gareSimili.forEach(g => {
            g.tris.forEach(t => {
                if (!trisMap[t.combinazione]) trisMap[t.combinazione] = { count: 0, quote: [] };
                trisMap[t.combinazione].count++;
                trisMap[t.combinazione].quote.push(parseFloat(t.quota));
            });
        });

        const trisOrdinata = Object.entries(trisMap)
            .sort((a, b) => b[1].count - a[1].count);

        if (trisOrdinata.length === 0) return null;

        const [combinazione, dati] = trisOrdinata[0];
        const mediaQuota = (dati.quote.reduce((a, b) => a + b, 0) / dati.quote.length).toFixed(2);

        return { combinazione, quota: mediaQuota };
    }

    // Funzione per mostrare report a schermo
    function mostraReport(testo) {
        const div = document.getElementById("report");
        div.textContent = testo;
    }
</script>
</body>
</html>
