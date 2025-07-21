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
            display: block; /* Tabelle una sotto l'altra */
            gap: 30px;
            margin-bottom: 30px;
        }
        .gara {
            margin-bottom: 30px;
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
        }
        /* Colonne dimezzate */
        th:first-child, td:first-child {
            width: 5%;
        }
        th:nth-child(2), td:nth-child(2) {
            width: 35%;
            text-align: left;
            padding-left: 8px;
        }
        th:nth-child(3), td:nth-child(3) {
            width: 15%;
        }
        input[type="text"] {
            width: 90%;
            padding: 4px;
        }
        input.quota {
            width: 50px;
        }
        button {
            margin-top: 5px;
            margin-right: 10px;
        }
        /* Stile autocomplete */
        .autocomplete-items {
            position: absolute;
            border: 1px solid #d4d4d4;
            border-bottom: none;
            border-top: none;
            z-index: 99;
            top: 100%;
            left: 0;
            right: 0;
            background: white;
            max-height: 150px;
            overflow-y: auto;
        }
        .autocomplete-items div {
            padding: 10px;
            cursor: pointer;
        }
        .autocomplete-items div:hover {
            background-color: #e9e9e9;
        }
    </style>
</head>
<body>
<h2>Archivio Gare Cavalli</h2>

<div class="gara-container">
    <div class="gara">
        <table id="gara1">
            <tr><th>Corsia</th><th>Nome</th><th>Quota</th></tr>
            <script>
                for (let i = 1; i <= 6; i++) {
                    document.write(`<tr><td style="position:relative">${i}</td><td style="position:relative"><input type="text" id="nome1_${i}" class="nome" autocomplete="off" /></td><td><input type="text" id="quota1_${i}" class="quota" maxlength="4" inputmode="numeric" pattern="[0-9]*" /></td></tr>`);
                }
            </script>
        </table>
        <button onclick="salvaGara(1)">Salva</button>
        <button onclick="pulisciTabella(1)">Pulisci Tabella</button>
        <button onclick="cercaGare()">Cerca</button>
    </div>

    <div class="gara">
        <table id="gara2">
            <tr><th>Corsia</th><th>Nome</th><th>Quota</th></tr>
            <script>
                for (let i = 1; i <= 6; i++) {
                    document.write(`<tr><td style="position:relative">${i}</td><td style="position:relative"><input type="text" id="nome2_${i}" class="nome" autocomplete="off" /></td><td><input type="text" id="quota2_${i}" class="quota" maxlength="4" inputmode="numeric" pattern="[0-9]*" /></td></tr>`);
                }
            </script>
        </table>
        <button onclick="salvaGara(2)">Salva</button>
        <button onclick="pulisciTabella(2)">Pulisci Tabella</button>
    </div>
</div>

<button onclick="cancellaTutto()">Cancella Tutto (Storage)</button>
<button onclick="mostraStorage()">Mostra tutte le gare salvate</button>

<script>
    // Autocomplete nomi cavalli
    function setupAutocomplete() {
        const inputs = document.querySelectorAll("input.nome");
        let cavalliSalvati = new Set();

        function aggiornaCavalliSalvati() {
            cavalliSalvati = new Set();
            const gare = JSON.parse(localStorage.getItem("gare") || "[]");
            gare.forEach(g => {
                g.nomi.forEach(nome => {
                    if (nome.trim()) cavalliSalvati.add(nome.trim());
                });
            });
        }
        aggiornaCavalliSalvati();

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

        // Trova gara uguale per nomi+quote
        const garaEsistente = gare.find(g => JSON.stringify(g.nomi) === JSON.stringify(nomi) && JSON.stringify(g.quote) === JSON.stringify(quote));

        if (garaEsistente) {
            const trisElenco = garaEsistente.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n");
            alert(`Questa gara esiste già e ha queste tris vincenti:\n${trisElenco}`);

            if (confirm("Vuoi aggiungere una nuova tris vincente diversa?")) {
                inserisciNuovaTris(garaEsistente);
            }
            return;
        }

        // Gara NON esistente: mostra analisi AI
        const win = window.open("", "Analisi Cavalli Inseriti", "width=400,height=350,scrollbars=yes");
        const analisiHtml = generaReportAnalisiCavalli(nomi);
        win.document.body.style.fontFamily = "Arial, sans-serif";
        win.document.body.style.padding = "10px";
        win.document.body.innerHTML = analisiHtml;

        // Aspetta chiusura popup prima di chiedere tris e quota
        const timer = setInterval(() => {
            if (win.closed) {
                clearInterval(timer);
                inserisciNuovaTris(null, nomi, quote);
            }
        }, 300);
    }

    function inserisciNuovaTris(gara, nomi = null, quote = null) {
        let trisInput = prompt("Inserire tris vincente (3 numeri separati da virgola, es: 1,3,5)");
        if (!trisInput) return;
        const trisArray = trisInput.split(",").map(s => s.trim());
        if (trisArray.length !== 3 || trisArray.some(n => isNaN(n) || n === "")) {
            alert("La tris vincente deve essere formata da 3 numeri validi.");
            return;
        }

        let quotaTris = prompt("Inserire quota della tris vincente (solo numeri)");
        if (!quotaTris) return;
        if (isNaN(quotaTris) || quotaTris.trim() === "") {
            alert("Devi inserire una quota numerica valida.");
            return;
        }

        const nuovaTris = { combinazione: trisArray.join(","), quota: quotaTris };

        if (!gara) {
            const gare = JSON.parse(localStorage.getItem("gare") || "[]");
            gare.push({
                nomi,
                quote,
                tris: [nuovaTris]
            });
            localStorage.setItem("gare", JSON.stringify(gare));
        } else {
            if (!gara.tris.some(t => t.combinazione === nuovaTris.combinazione)) {
                gara.tris.push(nuovaTris);
                let gareSalvate = JSON.parse(localStorage.getItem("gare") || "[]");
                const idx = gareSalvate.findIndex(g => JSON.stringify(g.nomi) === JSON.stringify(gara.nomi) && JSON.stringify(g.quote) === JSON.stringify(gara.quote));
                if (idx !== -1) {
                    gareSalvate[idx] = gara;
                    localStorage.setItem("gare", JSON.stringify(gareSalvate));
                }
            } else {
                alert("Questa tris è già salvata per questa gara.");
            }
        }
        alert("Gara salvata correttamente.");
        setupAutocomplete();
    }

    function pulisciTabella(index) {
        for (let i = 1; i <= 6; i++) {
            document.getElementById(`nome${index}_${i}`).value = "";
            document.getElementById(`quota${index}_${i}`).value = "";
        }
    }

    function cancellaTutto() {
        if (confirm("Sei sicuro di voler cancellare tutte le gare salvate?")) {
            localStorage.removeItem("gare");
            alert("Tutte le gare sono state eliminate.");
            setupAutocomplete();
        }
    }

    function mostraStorage() {
        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        if (gare.length === 0) {
            alert("Nessuna gara salvata.");
            return;
        }
        let output = "<h2>Gare Salvate</h2><ul style='font-family: Arial, sans-serif;'>";
        gare.forEach((g, i) => {
            output += `<li><strong>Gara ${i+1}:</strong><br>NomI: ${g.nomi.join(", ")}<br>Quote: ${g.quote.join(", ")}<br>Tris vincenti:<ul>`;
            g.tris.forEach(t => {
                output += `<li>${t.combinazione} (Quota: ${t.quota})</li>`;
            });
            output += `</ul></li>`;
        });
        output += "</ul>";

        const win = window.open("", "Storage Gare Salvate", "width=600,height=400,scrollbars=yes");
        win.document.body.style.fontFamily = "Arial, sans-serif";
        win.document.body.style.padding = "10px";
        win.document.body.innerHTML = output;
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
                <p><strong>Tris vincenti:</strong> ${gara.tris.map(t => `${t.combinazione} (Quota: ${t.quota})`).join(", ")}</p>
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

    // Funzione di analisi che conta le posizioni da tris vincenti (corsie)
    function generaReportAnalisiCavalli(nomiDaAnalizzare) {
        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        if (gare.length === 0) return "<p>Nessuna gara salvata da analizzare.</p>";

        const gareFiltrate = gare.filter(g =>
            g.nomi.some(nome => nomiDaAnalizzare.includes(nome))
        );
        if (gareFiltrate.length === 0) return "<p>Nessuna gara trovata con i cavalli inseriti.</p>";

        const statsCavalli = {};
        nomiDaAnalizzare.forEach(nome => {
            statsCavalli[nome] = {
                apparizioni: 0,
                primePosizioni: 0,
                secondePosizioni: 0,
                terzePosizioni: 0,
            };
        });

        gareFiltrate.forEach((gara, idx) => {
            const corsiaNomeMap = {};
            gara.nomi.forEach((nome, i) => {
                corsiaNomeMap[i + 1] = nome;
            });

            gara.tris.forEach(t => {
                const trisCorsie = t.combinazione.split(",").map(x => parseInt(x.trim(), 10));
                if (trisCorsie.length !== 3) return;

                const posizioni = [
                    { corsia: trisCorsie[0], posizione: 1 },
                    { corsia: trisCorsie[1], posizione: 2 },
                    { corsia: trisCorsie[2], posizione: 3 }
                ];

                posizioni.forEach(p => {
                    const cavallo = corsiaNomeMap[p.corsia];
                    if (cavallo && statsCavalli[cavallo]) {
                        statsCavalli[cavallo].apparizioni++;
                        if (p.posizione === 1) statsCavalli[cavallo].primePosizioni++;
                        else if (p.posizione === 2) statsCavalli[cavallo].secondePosizioni++;
                        else if (p.posizione === 3) statsCavalli[cavallo].terzePosizioni++;
                    }
                });
            });
        });

        // Tabella senza colonna "Gare con presenza"
        let html = `<h3>Analisi cavalli inseriti</h3><table style="width:100%;border-collapse:collapse;">
            <tr style="background:#ddd;">
                <th>Cavallo</th>
                <th>Apparizioni</th>
                <th>1° posto</th>
                <th>2° posto</th>
                <th>3° posto</th>
            </tr>`;

        for (const nome of nomiDaAnalizzare) {
            const s = statsCavalli[nome];
            html += `<tr>
                <td style="text-align:left;padding-left:5px;">${nome}</td>
                <td>${s.apparizioni}</td>
                <td>${s.primePosizioni}</td>
                <td>${s.secondePosizioni}</td>
                <td>${s.terzePosizioni}</td>
            </tr>`;
        }
        html += "</table>";

        // Aggiungo previsione tris in fondo
        html += "<h3>Previsione combinazioni tris con i cavalli inseriti</h3>";
        html += generaPrevisioneTris(nomiDaAnalizzare);

        return html;
    }

    // Funzione che genera tutte le combinazioni 3 a 3 ordinate (permuta semplice)
    function generaPrevisioneTris(cavalli) {
        if (cavalli.length < 3) return "<p>Inserire almeno 3 cavalli per la previsione tris.</p>";

        let output = "<ul style='text-align:left; max-height:200px; overflow-y:auto;'>";
        for (let i = 0; i < cavalli.length; i++) {
            for (let j = 0; j < cavalli.length; j++) {
                if (j === i) continue;
                for (let k = 0; k < cavalli.length; k++) {
                    if (k === i || k === j) continue;
                    output += `<li>1°: <strong>${cavalli[i]}</strong>, 2°: <strong>${cavalli[j]}</strong>, 3°: <strong>${cavalli[k]}</strong></li>`;
                }
            }
        }
        output += "</ul>";
        return output;
    }
</script>
</body>
</html>
