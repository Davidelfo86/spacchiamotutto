<!DOCTYPE html>
<html data-theme="light">
<head>
    <title>Corse Cavalli</title>
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <style>
        /* Variabili tema chiaro/scuro */
        :root[data-theme="light"] {
            --bg-color: #ffffff;
            --text-color: #333333;
            --border-color: #dddddd;
            --header-bg: #f8f9fa;
            --button-primary: #4CAF50;
            --button-secondary: #2196F3;
            --alert-warning: #f44336;
            --alert-info: #2196F3;
        }

        :root[data-theme="dark"] {
            --bg-color: #1a1a1a;
            --text-color: #ffffff;
            --border-color: #444444;
            --header-bg: #2d2d2d;
            --button-primary: #45a049;
            --button-secondary: #1976d2;
            --alert-warning: #d32f2f;
            --alert-info: #1976d2;
        }

        /* Stili generali */
        body {
            font-family: Arial, sans-serif;
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: all 0.3s ease;
        }

        /* Header e controlli tema */
        .header-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
        }

        .theme-switch {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .toggle-switch {
            position: relative;
            display: inline-block;
            width: 60px;
            height: 34px;
        }

        .toggle-switch input {
            opacity: 0;
            width: 0;
            height: 0;
        }

        .slider {
            position: absolute;
            cursor: pointer;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: #ccc;
            transition: .4s;
            border-radius: 34px;
        }

        .slider:before {
            position: absolute;
            content: "";
            height: 26px;
            width: 26px;
            left: 4px;
            bottom: 4px;
            background-color: white;
            transition: .4s;
            border-radius: 50%;
        }

        input:checked + .slider {
            background-color: var(--button-secondary);
        }

        input:checked + .slider:before {
            transform: translateX(26px);
        }

        /* Stili tabella */
        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
            border: 1px solid var(--border-color);
        }

        th, td {
            padding: 12px;
            text-align: left;
            border: 1px solid var(--border-color);
        }

        th {
            background-color: var(--header-bg);
        }

        table th:nth-child(3),
        table th:nth-child(4),
        table th:nth-child(5),
        table td:nth-child(3),
        table td:nth-child(4),
        table td:nth-child(5) {
            width: 10%;
        }

        table th:nth-child(1),
        table td:nth-child(1) {
            width: 5%;
        }

        table th:nth-child(2),
        table td:nth-child(2) {
            width: 32%;
        }

        /* Stili input e select */
        input, select {
            width: 90%;
            padding: 8px;
            border: 1px solid var(--border-color);
            border-radius: 4px;
            background-color: var(--bg-color);
            color: var(--text-color);
        }

        table td:nth-child(3) input,
        table td:nth-child(4) input,
        table td:nth-child(5) input,
        table td:nth-child(3) select,
        table td:nth-child(4) select,
        table td:nth-child(5) select {
            width: 60%;
        }

        /* Stili bottoni */
        button {
            padding: 10px 20px;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            margin: 10px 5px;
            transition: opacity 0.3s ease;
        }

        button:hover {
            opacity: 0.9;
        }

        .btn-salva {
            background-color: var(--button-primary);
        }

        .btn-cerca {
            background-color: var(--button-secondary);
        }

        .btn-toggle {
            background-color: var(--button-secondary);
        }

        /* Stili alert e messaggi */
        .alert {
            padding: 15px;
            margin: 10px 0;
            border-radius: 4px;
            color: white;
        }

        .alert-warning {
            background-color: var(--alert-warning);
        }

        .alert-info {
            background-color: var(--alert-info);
        }

        /* Altri stili */
        .tris-row {
            background-color: var(--header-bg);
            font-weight: bold;
        }

        .button-container {
            margin: 20px 0;
        }

        .log-container {
            margin-top: 20px;
            transition: all 0.3s ease;
        }

        .hidden {
            display: none;
        }

        select {
            background-color: var(--bg-color);
        }

        /* Responsive design */
        @media screen and (max-width: 768px) {
            body {
                padding: 10px;
                font-size: 14px;
            }

            table {
                font-size: 12px;
            }

            th, td {
                padding: 6px;
            }

            input, select {
                padding: 4px;
                font-size: 12px;
            }

            button {
                padding: 8px 16px;
                font-size: 14px;
            }

            .header-controls {
                flex-direction: column;
                gap: 10px;
            }

            .theme-switch {
                margin-top: 10px;
            }
        }
    </style>
</head>
<body>
    <div class="header-controls">
        <h1>Corse Cavalli</h1>
        <div class="theme-switch">
            <span>☀️</span>
            <label class="toggle-switch">
                <input type="checkbox" id="themeToggle">
                <span class="slider"></span>
            </label>
            <span>🌙</span>
        </div>
    </div>
    
    <div id="messageBox"></div>

    <table id="mainTable">
        <tr>
            <th></th>
            <th>Cavalli</th>
            <th>1°posto</th>
            <th>2°posto</th>
            <th>3°posto</th>
        </tr>
        <tr>
            <td>1</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr>
            <td>2</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr>
            <td>3</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr>
            <td>4</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr>
            <td>5</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr>
            <td>6</td>
            <td><input type="text" class="cavalli-input"></td>
            <td><input type="number" inputmode="decimal" class="primo-input"></td>
            <td><input type="number" inputmode="decimal" class="secondo-input"></td>
            <td><input type="number" inputmode="decimal" class="terzo-input"></td>
        </tr>
        <tr class="tris-row">
            <td></td>
            <td>Tris vincente</td>
            <td>
                <select id="primoTris" style="width: 60%">
                    <option value="">-</option>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                    <option value="4">4</option>
                    <option value="5">5</option>
                    <option value="6">6</option>
                </select>
            </td>
            <td>
                <select id="secondoTris" style="width: 60%">
                    <option value="">-</option>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                    <option value="4">4</option>
                    <option value="5">5</option>
                    <option value="6">6</option>
                </select>
            </td>
            <td>
                <select id="terzoTris" style="width: 60%">
                    <option value="">-</option>
                    <option value="1">1</option>
                    <option value="2">2</option>
                    <option value="3">3</option>
                    <option value="4">4</option>
                    <option value="5">5</option>
                    <option value="6">6</option>
                </select>
            </td>
        </tr>
    </table>

    <div class="button-container">
        <button onclick="cercaGara()" class="btn-cerca">Cerca Gara</button>
        <button onclick="salvaDati()" class="btn-salva">Salva Corsa</button>
        <button onclick="toggleLogDati()" class="btn-toggle" id="toggleLogBtn">
            Nascondi Log
        </button>
    </div>

    <div class="log-container" id="logContainer">
        <div class="log-controls">
            <label>Ordina per: </label>
            <select id="sortOrder" onchange="ordinaLog()">
                <option value="newest">Più recenti</option>
                <option value="oldest">Più vecchie</option>
            </select>
        </div>
        <h3>Log Dati</h3>
        <div id="logDati"></div>
    </div>
    <script>
        let corse = JSON.parse(localStorage.getItem('corse')) || [];
        const SHEETS_URL = 'https://script.google.com/macros/s/AKfycbxLFkI2fU329vYO1Q73rsorW3SLIt8lKO4PpvxyuHZZuZOnHm5BsY2Jg92lRxhxXsf5/exec';

        // Gestione tema
        const themeToggle = document.getElementById('themeToggle');
        themeToggle.checked = localStorage.getItem('theme') === 'dark';
        
        function toggleTheme() {
            const html = document.documentElement;
            const isDark = themeToggle.checked;
            html.setAttribute('data-theme', isDark ? 'dark' : 'light');
            localStorage.setItem('theme', isDark ? 'dark' : 'light');
        }

        themeToggle.addEventListener('change', toggleTheme);
        toggleTheme();

        // Gestione visibilità log
        function toggleLogDati() {
            const logContainer = document.getElementById('logContainer');
            const toggleBtn = document.getElementById('toggleLogBtn');
            const isVisible = !logContainer.classList.contains('hidden');
            
            logContainer.classList.toggle('hidden');
            toggleBtn.textContent = isVisible ? 'Mostra Log' : 'Nascondi Log';
        }

        // Ordinamento log
        function ordinaLog() {
            const sortOrder = document.getElementById('sortOrder').value;
            const corsaOrdinate = [...corse].sort((a, b) => {
                const dateA = new Date(a.data.split('/').reverse().join('-'));
                const dateB = new Date(b.data.split('/').reverse().join('-'));
                return sortOrder === 'newest' ? dateB - dateA : dateA - dateB;
            });
            
            aggiornaLogDati(corsaOrdinate);
        }

        async function salvaInGoogleSheets(corsa) {
            try {
                const response = await fetch(SHEETS_URL, {
                    method: 'POST',
                    mode: 'no-cors',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        ...corsa,
                        trisVincente: "'" + corsa.trisVincente
                    })
                });
                return true;
            } catch (error) {
                console.error('Errore nel salvataggio:', error);
                return false;
            }
        }

        function mostraMessaggio(messaggio, tipo) {
            const messageBox = document.getElementById('messageBox');
            messageBox.innerHTML = `<div class="alert alert-${tipo}">${messaggio}</div>`;
            setTimeout(() => {
                messageBox.innerHTML = '';
            }, 5000);
        }

        function cercaGara() {
            const righe = document.querySelectorAll('#mainTable tr');
            const cavalliAttuali = [];

            for (let i = 1; i < 7; i++) {
                const riga = righe[i];
                const cavalloInput = riga.querySelectorAll('input')[0];
                if (cavalloInput.value.trim()) {
                    cavalliAttuali.push(cavalloInput.value.trim());
                }
            }

            if (cavalliAttuali.length === 0) {
                mostraMessaggio('⚠️ Inserisci almeno un cavallo prima di cercare', 'warning');
                return;
            }

            let trovata = false;
            for (let corsa of corse) {
                let cavalliCorsa = corsa.dati
                    .filter(d => d.cavalli.trim())
                    .map(d => d.cavalli.trim());

                if (JSON.stringify(cavalliCorsa) === JSON.stringify(cavalliAttuali)) {
                    mostraMessaggio(`🎯 Corsa trovata! La tris vincente era: ${corsa.trisVincente}`, 'info');
                    trovata = true;
                    break;
                }
            }

            if (!trovata) {
                mostraMessaggio('🔍 Nessuna corsa simile trovata nel database', 'warning');
            }
        }

        async function salvaDati() {
            const primo = document.getElementById('primoTris').value;
            const secondo = document.getElementById('secondoTris').value;
            const terzo = document.getElementById('terzoTris').value;

            if (!primo || !secondo || !terzo) {
                mostraMessaggio('⚠️ Seleziona tutti i numeri della tris!', 'warning');
                return;
            }

            const data = new Date().toLocaleDateString('it-IT', {
                day: '2-digit',
                month: '2-digit',
                year: 'numeric'
            });
            
            const corsa = {
                id: Date.now(),
                data: data,
                trisVincente: `${primo}-${secondo}-${terzo}`,
                dati: []
            };

            for (let i = 1; i < 7; i++) {
                const riga = document.querySelectorAll('#mainTable tr')[i];
                const inputs = riga.querySelectorAll('input');
                const numero = riga.cells[0].textContent;
                corsa.dati.push({
                    numero: numero,
                    cavalli: inputs[0].value,
                    primo: inputs[1].value,
                    secondo: inputs[2].value,
                    terzo: inputs[3].value
                });
            }

            if (corsa.dati.every(riga => !riga.cavalli)) {
                mostraMessaggio('⚠️ Inserisci almeno un cavallo!', 'warning');
                return;
            }

            mostraMessaggio('⌛ Salvataggio in corso...', 'info');

            const salvataggioOk = await salvaInGoogleSheets(corsa);
            
            if (salvataggioOk) {
                corse.push(corsa);
                localStorage.setItem('corse', JSON.stringify(corse));
                aggiornaLogDati();
                pulisciForm();
                mostraMessaggio('✅ Corsa salvata con successo!', 'info');
            }
        }

        function aggiornaLogDati(corseDaMostrare = corse) {
            const logDiv = document.getElementById('logDati');
            logDiv.innerHTML = '';
            
            corseDaMostrare.forEach(corsa => {
                const corsaDiv = document.createElement('div');
                corsaDiv.style.marginBottom = '20px';
                corsaDiv.style.borderBottom = '1px solid var(--border-color)';
                
                corsaDiv.innerHTML = `
                    <p><strong>Data: ${corsa.data}</strong> | Tris: ${corsa.trisVincente}</p>
                    <table style="width: 100%; margin-bottom: 10px;">
                        <tr>
                            <th>Numero</th>
                            <th>Cavalli</th>
                            <th>1°posto</th>
                            <th>2°posto</th>
                            <th>3°posto</th>
                        </tr>
                        ${corsa.dati.map(riga => `
                            <tr>
                                <td>${riga.numero}</td>
                                <td>${riga.cavalli}</td>
                                <td>${riga.primo}</td>
                                <td>${riga.secondo}</td>
                                <td>${riga.terzo}</td>
                            </tr>
                        `).join('')}
                    </table>
                    <button onclick="eliminaCorsa(${corsa.id})" class="btn-cerca">Elimina</button>
                `;
                logDiv.appendChild(corsaDiv);
            });
        }

        function eliminaCorsa(id) {
            if (confirm('Sei sicuro di voler eliminare questa corsa?')) {
                corse = corse.filter(corsa => corsa.id !== id);
                localStorage.setItem('corse', JSON.stringify(corse));
                aggiornaLogDati();
                mostraMessaggio('✅ Corsa eliminata con successo', 'info');
            }
        }

        function pulisciForm() {
            const inputs = document.querySelectorAll('#mainTable input');
            inputs.forEach(input => input.value = '');
            document.getElementById('primoTris').value = '';
            document.getElementById('secondoTris').value = '';
            document.getElementById('terzoTris').value = '';
        }

        // Inizializzazione
        window.onload = function() {
            aggiornaLogDati();
            toggleTheme();
        };
    </script>
</body>
</html>
