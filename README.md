<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pronostici Calcio PRO</title>
    
    <!-- Meta tag per PWA su iOS/iPhone -->
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="apple-mobile-web-app-title" content="PronosticiPRO">
    <link rel="apple-touch-icon" href="https://cdn-icons-png.flaticon.com/512/53/53283.png">
    
    <!-- Tailwind CSS & Chart.js CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    
    <style>
        body {
            background-color: #0f172a;
            color: #f8fafc;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            -webkit-tap-highlight-color: transparent;
        }
        .card {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
            border-radius: 1rem;
        }
        .glow-button {
            box-shadow: 0 0 15px rgba(59, 130, 246, 0.4);
        }
    </style>
</head>
<body class="pb-20 pt-4 px-3 sm:px-6 max-w-5xl mx-auto">

    <!-- Header & Configurazione API -->
    <header class="flex justify-between items-center mb-6">
        <div>
            <h1 class="text-2xl font-black tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-blue-400 to-emerald-400">
                PRONOSTICI CALCIO <span class="text-xs px-2 py-0.5 rounded bg-blue-600/30 text-blue-400 border border-blue-500/30">PRO</span>
            </h1>
            <p class="text-xs text-slate-400 mt-0.5">Poisson, xG & Value Bet Engine</p>
        </div>
        <button onclick="openApiModal()" class="px-3 py-2 text-xs font-semibold rounded-lg bg-slate-800 hover:bg-slate-700 text-blue-400 border border-blue-500/30 flex items-center gap-1.5 transition">
            <i class="fa-solid fa-key"></i> 🔑 Configura API
        </button>
    </header>

    <!-- Dashboard Finanziaria -->
    <section class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6">
        <div class="card p-4 flex flex-col justify-between">
            <span class="text-xs font-medium text-slate-400">Bankroll Attuale</span>
            <div id="bankrollValue" class="text-xl font-bold text-white mt-1">100.00 U</div>
            <span class="text-[10px] text-slate-500 mt-1">Cassa iniziale: 100 U</span>
        </div>
        <div class="card p-4 flex flex-col justify-between">
            <span class="text-xs font-medium text-slate-400">Profitto / Perdita</span>
            <div id="profitValue" class="text-xl font-bold text-emerald-400 mt-1">+0.00 U</div>
            <span id="roiValue" class="text-[10px] text-emerald-500 mt-1">ROI: 0.0%</span>
        </div>
        <div class="card p-4 flex flex-col justify-between">
            <span class="text-xs font-medium text-slate-400">Win Rate</span>
            <div id="winRateValue" class="text-xl font-bold text-blue-400 mt-1">0%</div>
            <span id="betsRatio" class="text-[10px] text-slate-500 mt-1">0 Vinte / 0 Totali</span>
        </div>
        <div class="card p-4 flex flex-col justify-between">
            <span class="text-xs font-medium text-slate-400">Yield %</span>
            <div id="yieldValue" class="text-xl font-bold text-purple-400 mt-1">0.0%</div>
            <span class="text-[10px] text-slate-500 mt-1">Efficienza puntate</span>
        </div>
    </section>

    <!-- Grafico Bankroll -->
    <section class="card p-4 mb-6">
        <div class="flex justify-between items-center mb-3">
            <h2 class="text-sm font-semibold text-slate-300">Andamento Bankroll (Unità)</h2>
            <button onclick="resetBankroll()" class="text-[11px] text-rose-400 hover:underline">Resetta Dati</button>
        </div>
        <div class="h-44 w-full">
            <canvas id="bankrollChart"></canvas>
        </div>
    </section>

    <!-- AI Match Analyzer & Poisson Simulator -->
    <section class="card p-5 mb-6">
        <div class="flex items-center justify-between mb-4">
            <h2 class="text-base font-bold text-white flex items-center gap-2">
                <i class="fa-solid fa-robot text-blue-400"></i> AI Match Analyzer & Poisson xG
            </h2>
            <button onclick="syncApiData()" class="text-xs px-2.5 py-1 rounded bg-emerald-600/20 text-emerald-400 border border-emerald-500/30 hover:bg-emerald-600/30 transition">
                <i class="fa-solid fa-rotate"></i> 🔄 Sincronizza Live
            </button>
        </div>

        <!-- Selettore Incontro -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mb-4">
            <div>
                <label class="block text-xs text-slate-400 mb-1">Seleziona Campionato</label>
                <select id="leagueSelect" onchange="loadMatchesForLeague()" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-xs text-white">
                    <option value="SA">Serie A</option>
                    <option value="CL">UEFA Champions League</option>
                    <option value="PL">Premier League</option>
                    <option value="PD">LaLiga</option>
                    <option value="BL1">Bundesliga</option>
                    <option value="FL1">Ligue 1</option>
                </select>
            </div>
            <div class="md:col-span-2">
                <label class="block text-xs text-slate-400 mb-1">Match in Palinsesto</label>
                <select id="matchSelect" onchange="populateMatchInputs()" class="w-full bg-slate-800 border border-slate-700 rounded-lg p-2 text-xs text-white">
                    <!-- Dinamico -->
                </select>
            </div>
        </div>

        <!-- Parametri Attacco / Difesa -->
        <div class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-4 text-xs">
            <div>
                <label class="block text-slate-400 mb-1">Attacco Casa</label>
                <input type="number" id="homeAtt" value="82" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
            </div>
            <div>
                <label class="block text-slate-400 mb-1">Difesa Casa</label>
                <input type="number" id="homeDef" value="80" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
            </div>
            <div>
                <label class="block text-slate-400 mb-1">Attacco Ospite</label>
                <input type="number" id="awayAtt" value="78" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
            </div>
            <div>
                <label class="block text-slate-400 mb-1">Difesa Ospite</label>
                <input type="number" id="awayDef" value="82" class="w-full bg-slate-900 border border-slate-700 rounded p-2 text-white">
            </div>
        </div>

        <button onclick="runPoissonSimulation()" class="w-full py-2.5 rounded-lg bg-gradient-to-r from-blue-600 to-indigo-600 hover:from-blue-500 hover:to-indigo-500 font-bold text-sm text-white glow-button transition">
            Simula Match (Poisson & xG)
        </button>

        <!-- Risultati Simulazione -->
        <div id="simulationOutput" class="hidden mt-5 pt-4 border-t border-slate-700/60">
            <div class="grid grid-cols-3 gap-2 text-center mb-4">
                <div class="p-2 bg-slate-800/80 rounded-lg">
                    <span class="block text-[10px] text-slate-400">1 (Casa)</span>
                    <span id="prob1" class="text-sm font-bold text-emerald-400">0%</span>
                    <span id="fairOdds1" class="block text-[10px] text-slate-400 mt-0.5">Q. Equa: -</span>
                </div>
                <div class="p-2 bg-slate-800/80 rounded-lg">
                    <span class="block text-[10px] text-slate-400">X (Pareggio)</span>
                    <span id="probX" class="text-sm font-bold text-amber-400">0%</span>
                    <span id="fairOddsX" class="block text-[10px] text-slate-400 mt-0.5">Q. Equa: -</span>
                </div>
                <div class="p-2 bg-slate-800/80 rounded-lg">
                    <span class="block text-[10px] text-slate-400">2 (Ospite)</span>
                    <span id="prob2" class="text-sm font-bold text-rose-400">0%</span>
                    <span id="fairOdds2" class="block text-[10px] text-slate-400 mt-0.5">Q. Equa: -</span>
                </div>
            </div>

            <div class="flex justify-between items-center text-xs bg-slate-900/60 p-3 rounded-lg border border-slate-800">
                <div>
                    <span class="text-slate-400 block">Expected Goals (xG):</span>
                    <span id="xgResult" class="font-bold text-blue-300">Casa 0.00 - 0.00 Ospite</span>
                </div>
                <button onclick="exportToBets()" class="px-3 py-1.5 rounded bg-blue-600 hover:bg-blue-500 text-white font-semibold text-xs">
                    + Aggiungi ai Pronostici
                </button>
            </div>
        </div>
    </section>

    <!-- Schedina & Pronostici Salvati -->
    <section class="card p-5">
        <h2 class="text-base font-bold text-white mb-4 flex items-center gap-2">
            <i class="fa-solid fa-list-check text-emerald-400"></i> I Miei Pronostici & Schedina
        </h2>
        
        <div id="betsContainer" class="space-y-2.5">
            <p class="text-xs text-slate-500 text-center py-4">Nessun pronostico inserito. Simula un match ed esportalo qui.</p>
        </div>
    </section>

    <!-- Modale Configurazione API -->
    <div id="apiModal" class="fixed inset-0 bg-black/80 backdrop-blur-sm hidden items-center justify-center p-4 z-50">
        <div class="card p-5 max-w-md w-full border border-slate-700">
            <h3 class="text-base font-bold text-white mb-2">🔑 Chiave API football-data.org</h3>
            <p class="text-xs text-slate-400 mb-4">
                Inserisci la tua API Key personale di <b>football-data.org</b> per sincronizzare in tempo reale risultati, calendari e squadre.
            </p>
            <input type="text" id="apiKeyInput" placeholder="Inserisci qui il tuo Token API..." class="w-full bg-slate-900 border border-slate-700 rounded-lg p-2.5 text-xs text-white mb-4">
            <div class="flex justify-end gap-2">
                <button onclick="closeApiModal()" class="px-3 py-1.5 rounded text-xs text-slate-400 hover:bg-slate-800">Annulla</button>
                <button onclick="saveApiKey()" class="px-4 py-1.5 rounded bg-blue-600 hover:bg-blue-500 text-white text-xs font-bold">Salva Key</button>
            </div>
        </div>
    </div>

    <!-- Script Applicativo Logic -->
    <script>
        // Dati predefiniti palinsesto
        const defaultMatches = {
            SA: [
                { id: 'sa1', home: 'SSC Napoli', away: 'AS Roma', hAtt: 84, hDef: 86, aAtt: 78, aDef: 82, odds1: 2.08, oddsX: 3.57, odds2: 4.16 },
                { id: 'sa2', home: 'Inter', away: 'AC Milan', hAtt: 88, hDef: 85, aAtt: 82, aDef: 80, odds1: 1.95, oddsX: 3.40, odds2: 3.90 },
                { id: 'sa3', home: 'Juventus', away: 'Lazio', hAtt: 80, hDef: 88, aAtt: 76, aDef: 78, odds1: 2.00, oddsX: 3.20, odds2: 4.00 }
            ],
            CL: [
                { id: 'cl1', home: 'PSG', away: 'FC Barcelona', hAtt: 90, hDef: 84, aAtt: 88, aDef: 82, odds1: 2.15, oddsX: 3.60, odds2: 3.10 },
                { id: 'cl2', home: 'Bayern München', away: 'Arsenal', hAtt: 89, hDef: 83, aAtt: 86, aDef: 86, odds1: 2.10, oddsX: 3.50, odds2: 3.30 },
                { id: 'cl3', home: 'Real Madrid', away: 'Manchester City', hAtt: 92, hDef: 86, aAtt: 91, aDef: 85, odds1: 2.45, oddsX: 3.50, odds2: 2.75 }
            ],
            PL: [
                { id: 'pl1', home: 'Arsenal', away: 'Chelsea', hAtt: 86, hDef: 87, aAtt: 80, aDef: 79, odds1: 1.75, oddsX: 3.80, odds2: 4.50 },
                { id: 'pl2', home: 'Liverpool', away: 'Tottenham', hAtt: 89, hDef: 82, aAtt: 84, aDef: 78, odds1: 1.65, oddsX: 4.20, odds2: 4.80 }
            ]
        };

        let userBets = JSON.parse(localStorage.getItem('pro_bets')) || [];
        let apiKey = localStorage.getItem('pro_api_key') || '';
        let currentSimulation = null;
        let chartInstance = null;

        // Inizializzazione App
        document.addEventListener('DOMContentLoaded', () => {
            loadMatchesForLeague();
            initChart();
            renderDashboard();
            renderBets();
        });

        // Selettore Partite
        function loadMatchesForLeague() {
            const league = document.getElementById('leagueSelect').value;
            const matchSelect = document.getElementById('matchSelect');
            matchSelect.innerHTML = '';

            const matches = defaultMatches[league] || [];
            matches.forEach((m, idx) => {
                const opt = document.createElement('option');
                opt.value = idx;
                opt.textContent = `${m.home} vs ${m.away}`;
                matchSelect.appendChild(opt);
            });

            if (matches.length > 0) populateMatchInputs();
        }

        function populateMatchInputs() {
            const league = document.getElementById('leagueSelect').value;
            const idx = document.getElementById('matchSelect').value;
            const m = (defaultMatches[league] || [])[idx];

            if (m) {
                document.getElementById('homeAtt').value = m.hAtt;
                document.getElementById('homeDef').value = m.hDef;
                document.getElementById('awayAtt').value = m.aAtt;
                document.getElementById('awayDef').value = m.aDef;
            }
        }

        // Calcolo Poisson / xG
        function runPoissonSimulation() {
            const hAtt = parseFloat(document.getElementById('homeAtt').value) / 50;
            const hDef = parseFloat(document.getElementById('homeDef').value) / 50;
            const aAtt = parseFloat(document.getElementById('awayAtt').value) / 50;
            const aDef = parseFloat(document.getElementById('awayDef').value) / 50;

            const xgHome = Math.max(0.2, (hAtt * (2 - aDef) * 1.35)).toFixed(2);
            const xgAway = Math.max(0.2, (aAtt * (2 - hDef) * 1.05)).toFixed(2);

            // Probabilità Poisson approssimata
            let pHome = 0, pDraw = 0, pAway = 0;
            for (let h = 0; h <= 5; h++) {
                for (let a = 0; a <= 5; a++) {
                    const prob = (poisson(h, xgHome) * poisson(a, xgAway));
                    if (h > a) pHome += prob;
                    else if (h === a) pDraw += prob;
                    else pAway += prob;
                }
            }

            const total = pHome + pDraw + pAway;
            pHome = (pHome / total * 100).toFixed(1);
            pDraw = (pDraw / total * 100).toFixed(1);
            pAway = (pAway / total * 100).toFixed(1);

            document.getElementById('prob1').textContent = `${pHome}%`;
            document.getElementById('probX').textContent = `${pDraw}%`;
            document.getElementById('prob2').textContent = `${pAway}%`;

            document.getElementById('fairOdds1').textContent = `Q. Equa: ${(100 / pHome).toFixed(2)}`;
            document.getElementById('fairOddsX').textContent = `Q. Equa: ${(100 / pDraw).toFixed(2)}`;
            document.getElementById('fairOdds2').textContent = `Q. Equa: ${(100 / pAway).toFixed(2)}`;

            document.getElementById('xgResult').textContent = `Casa ${xgHome} - ${xgAway} Ospite`;
            document.getElementById('simulationOutput').classList.remove('hidden');

            const league = document.getElementById('leagueSelect').value;
            const idx = document.getElementById('matchSelect').value;
            const m = defaultMatches[league][idx];

            currentSimulation = {
                match: `${m.home} vs ${m.away}`,
                pick: '1',
                odds: (100 / pHome).toFixed(2),
                stake: 1.0
            };
        }

        function poisson(k, lambda) {
            return (Math.pow(lambda, k) * Math.exp(-lambda)) / factorial(k);
        }

        function factorial(n) {
            return n <= 1 ? 1 : n * factorial(n - 1);
        }

        // Esportazione nei pronostici
        function exportToBets() {
            if (!currentSimulation) return;
            userBets.unshift({
                id: Date.now(),
                match: currentSimulation.match,
                pick: '1 (Vittoria Casa)',
                odds: parseFloat(currentSimulation.odds),
                stake: 1.0,
                status: 'PENDING'
            });
            saveBets();
            renderBets();
            renderDashboard();
        }

        // Render Schedine
        function renderBets() {
            const container = document.getElementById('betsContainer');
            if (userBets.length === 0) {
                container.innerHTML = '<p class="text-xs text-slate-500 text-center py-4">Nessun pronostico inserito.</p>';
                return;
            }

            container.innerHTML = userBets.map(b => `
                <div class="flex items-center justify-between p-3 bg-slate-800/60 rounded-lg border border-slate-700/50 text-xs">
                    <div>
                        <div class="font-bold text-white">${b.match}</div>
                        <div class="text-slate-400">Esito: <span class="text-blue-400 font-semibold">${b.pick}</span> @ ${b.odds}</div>
                    </div>
                    <div class="flex items-center gap-2">
                        <select onchange="updateBetStatus(${b.id}, this.value)" class="bg-slate-900 border border-slate-700 text-white rounded px-2 py-1">
                            <option value="PENDING" ${b.status === 'PENDING' ? 'selected' : ''}>IN ATTESA</option>
                            <option value="WON" ${b.status === 'WON' ? 'selected' : ''}>VINTO</option>
                            <option value="LOST" ${b.status === 'LOST' ? 'selected' : ''}>PERSO</option>
                        </select>
                        <button onclick="deleteBet(${b.id})" class="text-rose-400 hover:text-rose-300 px-1"><i class="fa-solid fa-trash"></i></button>
                    </div>
                </div>
            `).join('');
        }

        function updateBetStatus(id, status) {
            const bet = userBets.find(b => b.id === id);
            if (bet) {
                bet.status = status;
                saveBets();
                renderDashboard();
            }
        }

        function deleteBet(id) {
            userBets = userBets.filter(b => b.id !== id);
            saveBets();
            renderBets();
            renderDashboard();
        }

        function saveBets() {
            localStorage.setItem('pro_bets', JSON.stringify(userBets));
        }

        // Dashboard & Grafico
        function renderDashboard() {
            let bankroll = 100.0;
            let totalStaked = 0;
            let totalReturn = 0;
            let wonCount = 0;
            let totalResolved = 0;

            let history = [100.0];

            userBets.slice().reverse().forEach(b => {
                if (b.status !== 'PENDING') {
                    totalStaked += b.stake;
                    totalResolved++;
                    if (b.status === 'WON') {
                        const winAmt = b.stake * b.odds;
                        totalReturn += winAmt;
                        bankroll += (winAmt - b.stake);
                        wonCount++;
                    } else if (b.status === 'LOST') {
                        bankroll -= b.stake;
                    }
                    history.push(parseFloat(bankroll.toFixed(2)));
                }
            });

            const profit = bankroll - 100.0;
            const roi = totalStaked > 0 ? ((profit / totalStaked) * 100).toFixed(1) : '0.0';
            const winRate = totalResolved > 0 ? ((wonCount / totalResolved) * 100).toFixed(0) : '0';

            document.getElementById('bankrollValue').textContent = `${bankroll.toFixed(2)} U`;
            document.getElementById('profitValue').textContent = `${profit >= 0 ? '+' : ''}${profit.toFixed(2)} U`;
            document.getElementById('profitValue').className = `text-xl font-bold mt-1 ${profit >= 0 ? 'text-emerald-400' : 'text-rose-400'}`;
            document.getElementById('roiValue').textContent = `ROI: ${roi}%`;
            document.getElementById('winRateValue').textContent = `${winRate}%`;
            document.getElementById('betsRatio').textContent = `${wonCount} Vinte / ${totalResolved} Totali`;

            updateChart(history);
        }

        function initChart() {
            const ctx = document.getElementById('bankrollChart').getContext('2d');
            chartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: ['Start'],
                    datasets: [{
                        label: 'Bankroll (U)',
                        data: [100],
                        borderColor: '#3b82f6',
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: {
                        x: { display: false },
                        y: { grid: { color: 'rgba(255,255,255,0.05)' }, ticks: { color: '#94a3b8' } }
                    }
                }
            });
        }

        function updateChart(dataPoints) {
            if (!chartInstance) return;
            chartInstance.data.labels = dataPoints.map((_, i) => i === 0 ? 'Start' : `#${i}`);
            chartInstance.data.datasets[0].data = dataPoints;
            chartInstance.update();
        }

        function resetBankroll() {
            if (confirm('Vuoi azzerare lo storico dei pronostici?')) {
                userBets = [];
                saveBets();
                renderBets();
                renderDashboard();
            }
        }

        // Modale API
        function openApiModal() {
            document.getElementById('apiKeyInput').value = apiKey;
            document.getElementById('apiModal').classList.remove('hidden');
            document.getElementById('apiModal').classList.add('flex');
        }

        function closeApiModal() {
            document.getElementById('apiModal').classList.add('hidden');
            document.getElementById('apiModal').classList.remove('flex');
        }

        function saveApiKey() {
            apiKey = document.getElementById('apiKeyInput').value.trim();
            localStorage.setItem('pro_api_key', apiKey);
            closeApiModal();
            alert('API Key salvata con successo!');
        }

        function syncApiData() {
            if (!apiKey) {
                openApiModal();
                return;
            }
            alert('Sincronizzazione in corso tramite football-data.org...');
        }
    </script>
</body>
</html>
