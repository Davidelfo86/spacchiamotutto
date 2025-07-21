<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8" />
    <title>Archivio Gare Cavalli</title>
    <style>
        body {
            font-family: Arial, sans- serif;
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

        const gare = JSON.parse(localStorage.getItem("gare") || "[]");
        const garaEsistente = gare.find(g => JSON.stringify(g.nomi) === JSON.stringify(nomi) && JSON.stringify(g.quote) === JSON.stringify(quote));

        if (garaEsistente) {
            const trisElenco = garaEsistente.tris.map(t => `→ ${t.combinazione} (Quota: ${t.quota})`).join("\n");
            alert(`Questa gara esiste già e ha queste tris vincenti:\n${trisElenco}`);

            if (confirm("Vuoi aggiungere una nuova tris vincente diversa?")) {
                inserisciNuovaTris(garaEsistente);
            }
            return;
        }

        const gareStesseQuote = gare.filter(g => JSON.stringify(g.quote) === JSON.stringify(quote));
        if (gareStesseQuote.length > 0) {
            let messaggio = "Quote identiche ad un'altra gara. Ecco le tris salvate:\n";
            gareStesseQuote.forEach(g => {
                messaggio += `Tris: ${g.tris.map(t => t.combinazione + " (Quota: " + t.quota + ")").join(" | ")}\n`;
            });
            alert(messaggio);
            inserisciNuovaTris(null, nomi, quote);
            return;
        }

        inserisciNuovaTris(null, nomi, quote);
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
</script>
</body>
</html>
