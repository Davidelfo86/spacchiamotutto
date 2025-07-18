<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <title>Archivio Gare Cavalli</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            padding: 20px;
        }
        .gara-container {
            display: flex;
            gap: 50px;
            margin-bottom: 30px;
        }
        .gara {
            flex: 1;
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
                    document.write(`<tr><td>${i}</td><td><input type="text" id="nome1_${i}" class="nome" /></td><td><input type="text" id="quota1_${i}" class="quota" maxlength="4" /></td></tr>`);
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
                    document.write(`<tr><td>${i}</td><td><input type="text" id="nome2_${i}" class="nome" /></td><td><input type="text" id="quota2_${i}" class="quota" maxlength="4" /></td></tr>`);
                }
            </script>
        </table>
        <button onclick="salvaGara(2)">Salva</button>
        <button onclick="pulisciTabella(2)">Pulisci Tabella</button>
    </div>
</div>
<button onclick="cancellaTutto()">Cancella Tutto (Storage)</button>

<script>
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
            const tris = garaEsistente.tris.join(", ");
            const quotaTris = garaEsistente.quotaTris;
            if (confirm(`Gara già esistente.\nTris vincente: ${tris}\nQuota: ${quotaTris}\nÈ la stessa gara?`)) {
                alert("Ottimo, abbiamo vinto!");
            } else {
                inserisciNuovaTris(garaEsistente);
            }
            return;
        }

        const gareStesseQuote = gare.filter(g => JSON.stringify(g.quote) === JSON.stringify(quote));
        if (gareStesseQuote.length > 0) {
            let messaggio = "Quote identiche ad un'altra gara. Ecco le tris salvate:\n";
            gareStesseQuote.forEach(g => {
                messaggio += `Tris: ${g.tris.join(", ")} - Quota: ${g.quotaTris}\n`;
            });
            alert(messaggio);
            inserisciNuovaTris(null, nomi, quote);
            return;
        }

        inserisciNuovaTris(null, nomi, quote);
    }

    function inserisciNuovaTris(gara, nomi = null, quote = null) {
        const trisInput = prompt("Inserire tris vincente (separata da virgola)");
        if (!trisInput) return;
        const quotaTris = prompt("Inserire quota della tris vincente");
        if (!quotaTris) return;

        const nuovaGara = gara || {
            nomi,
            quote,
            tris: trisInput.split(",").map(s => s.trim()),
            quotaTris
        };

        if (!gara) {
            const gare = JSON.parse(localStorage.getItem("gare") || "[]");
            gare.push(nuovaGara);
            localStorage.setItem("gare", JSON.stringify(gare));
        } else {
            gara.tris.push(...trisInput.split(",").map(s => s.trim()));
            localStorage.setItem("gare", JSON.stringify(JSON.parse(localStorage.getItem("gare"))));
        }

        alert("Gara salvata correttamente.");
    }

    function pulisciTabella(index) {
        for (let i = 1; i <= 6; i++) {
            document.getElementById(`nome${index}_${i}`).value = "";
            document.getElementById(`quota${index}_${i}`).value = "";
        }
    }

    function cancellaTutto() {
        localStorage.removeItem("gare");
        alert("Tutte le gare sono state eliminate.");
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
                <p><strong>Tris vincenti:</strong> ${gara.tris.join(", ")}<br><strong>Quota tris:</strong> ${gara.quotaTris}</p>
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
</script>
</body>
</html>
