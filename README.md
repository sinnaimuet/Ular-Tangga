# Ular-Tangga
Made by Sinna Sahira Mikayla (30 | VIII A) 
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ular Tangga vs Soobin Bot</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Tone.js untuk Efek Suara (Synthesized sound effects) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fredoka:wght@400;500;600;700&display=swap');
        
        body {
            font-family: 'Fredoka', sans-serif;
            background: linear-gradient(135deg, #e0f2fe 0%, #f0fdf4 50%, #fefce8 100%);
        }

        /* Container Papan Utama */
        .board-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            aspect-ratio: 1 / 1;
        }

        /* Grid Papan 10x10 */
        .board-grid {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            grid-template-rows: repeat(10, 1fr);
            width: 100%;
            height: 100%;
            border-radius: 1rem;
            overflow: hidden;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.05);
        }

        /* Styling Ular dan Tangga Overlay */
        .svg-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 10;
        }

        /* Bidak Pemain */
        .player-token {
            transition: all 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
            z-index: 20;
        }

        /* Animasi Dadu Kocok */
        @keyframes roll {
            0% { transform: rotate(0deg) scale(1); }
            25% { transform: rotate(90deg) scale(1.1); }
            50% { transform: rotate(180deg) scale(0.9); }
            75% { transform: rotate(270deg) scale(1.1); }
            100% { transform: rotate(360deg) scale(1); }
        }

        .rolling {
            animation: roll 0.5s ease-in-out infinite;
        }
        
        /* Custom Scrollbar Log */
        .log-scroll::-webkit-scrollbar {
            width: 6px;
        }
        .log-scroll::-webkit-scrollbar-track {
            background: #f1f5f9;
            border-radius: 4px;
        }
        .log-scroll::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
    </style>
</head>
<body class="min-h-screen text-slate-700 flex flex-col justify-between p-3 md:p-6">

    <!-- Header / Judul Game -->
    <header class="text-center mb-4">
        <h1 class="text-3xl md:text-4xl font-bold text-indigo-600 tracking-wide flex items-center justify-center gap-3">
            <i class="fa-solid font-bold text-amber-500 fa-arrow-trend-up"></i>
            Ular Tangga Ceria
            <i class="fa-solid font-bold text-emerald-500 fa-staff-snake"></i>
        </h1>
        <p class="text-sm md:text-base text-slate-500 mt-1">Tantang <b>Soobin Bot</b> dan jadilah yang pertama mencapai petak 100!</p>
    </header>

    <!-- Konten Utama: Papan + Panel Kontrol -->
    <main class="max-w-6xl mx-auto w-full grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">
        
        <!-- Kolom Kiri: Papan Permainan -->
        <section class="lg:col-span-7 flex flex-col items-center justify-center">
            <div class="board-container bg-white p-2 md:p-3 rounded-2xl shadow-xl border-4 border-indigo-100">
                <!-- Grid 100 Kotak -->
                <div id="board" class="board-grid"></div>

                <!-- Canvas SVG Overlay untuk Ular & Tangga -->
                <svg id="svgOverlay" class="svg-overlay" viewBox="0 0 1000 1000"></svg>

                <!-- Bidak Pemain (Manusia) -->
                <div id="playerHuman" class="player-token absolute w-[7%] h-[7%] flex items-center justify-center rounded-full bg-rose-500 border-2 border-white shadow-lg text-white font-bold text-xs md:text-sm" style="opacity: 0;">
                    <i class="fa-solid fa-user"></i>
                </div>

                <!-- Bidak Bot (Soobin Bot) -->
                <div id="playerBot" class="player-token absolute w-[7%] h-[7%] flex items-center justify-center rounded-full bg-cyan-500 border-2 border-white shadow-lg text-white font-bold text-xs md:text-sm" style="opacity: 0;">
                    <i class="fa-solid fa-robot"></i>
                </div>
            </div>
        </section>

        <!-- Kolom Kanan: Dashboard Status, Dadu & Log -->
        <section class="lg:col-span-5 flex flex-col gap-4 w-full">
            
            <!-- Kartu Status Pemain -->
            <div class="grid grid-cols-2 gap-3">
                <!-- Status Manusia -->
                <div id="cardHuman" class="bg-white p-4 rounded-xl shadow-md border-2 border-rose-300 flex items-center gap-3 transition-all">
                    <div class="w-10 h-10 rounded-full bg-rose-500 text-white flex items-center justify-center font-bold text-lg shadow">
                        <i class="fa-solid fa-user"></i>
                    </div>
                    <div>
                        <span class="text-xs font-semibold text-rose-500 uppercase tracking-wider">Anda (Pemain)</span>
                        <div class="text-xl font-bold text-slate-800">Petak: <span id="posHuman">1</span></div>
                    </div>
                </div>

                <!-- Status Soobin Bot -->
                <div id="cardBot" class="bg-white p-4 rounded-xl shadow-md border-2 border-transparent flex items-center gap-3 transition-all opacity-75">
                    <div class="w-10 h-10 rounded-full bg-cyan-500 text-white flex items-center justify-center font-bold text-lg shadow">
                        <i class="fa-solid fa-robot"></i>
                    </div>
                    <div>
                        <span class="text-xs font-semibold text-cyan-600 uppercase tracking-wider">Soobin Bot</span>
                        <div class="text-xl font-bold text-slate-800">Petak: <span id="posBot">1</span></div>
                    </div>
                </div>
            </div>

            <!-- Area Lempar Dadu & Giliran -->
            <div class="bg-white p-6 rounded-xl shadow-md border border-indigo-50 flex flex-col items-center justify-center gap-4 text-center">
                <div id="turnIndicator" class="text-base font-medium text-slate-600 bg-indigo-50 px-4 py-1.5 rounded-full border border-indigo-100">
                    Giliran: <span class="font-bold text-rose-600">Anda</span>
                </div>

                <!-- Visual Dadu -->
                <div id="dice" class="w-20 h-20 bg-gradient-to-br from-indigo-500 to-purple-600 text-white rounded-2xl flex items-center justify-center text-4xl shadow-xl cursor-pointer select-none transition-transform active:scale-95">
                    <i class="fa-solid fa-dice-one"></i>
                </div>

                <!-- Tombol Action -->
                <button id="btnRoll" onclick="handleHumanTurn()" class="w-full py-3.5 px-6 bg-indigo-600 hover:bg-indigo-700 text-white font-bold text-lg rounded-xl shadow-lg shadow-indigo-200 transition-all active:scale-95 disabled:opacity-50 disabled:cursor-not-allowed">
                    <i class="fa-solid fa-dice mr-2"></i> Lempar Dadu
                </button>
            </div>

            <!-- Log Catatan Permainan -->
            <div class="bg-white p-4 rounded-xl shadow-md border border-slate-100 flex flex-col h-48">
                <h3 class="text-sm font-bold text-slate-500 uppercase tracking-wider mb-2 flex items-center gap-2">
                    <i class="fa-solid fa-scroll text-indigo-500"></i> Catatan Permainan
                </h3>
                <div id="gameLog" class="log-scroll flex-1 overflow-y-auto space-y-2 pr-1 text-sm text-slate-600">
                    <div class="p-2 bg-slate-50 rounded-lg text-slate-500 italic">Selamat datang! Klik "Lempar Dadu" untuk memulai petualangan.</div>
                </div>
            </div>

        </section>
    </main>

    <!-- Modal Kemenangan -->
    <div id="winModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-50">
        <div class="bg-white rounded-2xl max-w-sm w-full p-6 text-center shadow-2xl border-4 border-indigo-100 transform transition-all scale-100">
            <div id="winIcon" class="w-20 h-20 mx-auto rounded-full bg-amber-100 text-amber-500 flex items-center justify-center text-4xl mb-4 shadow-inner">
                <i class="fa-solid fa-trophy"></i>
            </div>
            <h2 id="winTitle" class="text-2xl font-extrabold text-slate-800 mb-1">Selamat!</h2>
            <p id="winMessage" class="text-slate-600 mb-6">Anda berhasil mengalahkan Soobin Bot!</p>
            <button onclick="resetGame()" class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-xl shadow-lg transition-transform active:scale-95">
                <i class="fa-solid fa-rotate-right mr-2"></i> Main Lagi
            </button>
        </div>
    </div>

    <!-- Footer -->
    <footer class="text-center text-xs text-slate-400 mt-6">
        Game Ular Tangga Interaktif &copy; 2026 - Bermain bersama Soobin Bot
    </footer>

    <!-- Logika Javascript -->
    <script>
        // --- KONFIGURASI GAME & ULAR/TANGGA ---
        const BOARD_SIZE = 10;
        const TOTAL_TILES = 100;

        // Definisi Tangga (Naik): { awal: akhir }
        const ladders = {
            4: 14,
            9: 31,
            20: 38,
            28: 84,
            40: 59,
            51: 67,
            63: 81,
            71: 91
        };

        // Definisi Ular (Turun): { kepala: ekor }
        const snakes = {
            17: 7,
            54: 34,
            62: 19,
            64: 60,
            87: 24,
            93: 73,
            95: 75,
            99: 78
        };

        // Warna Pastel Ceria untuk Kotak Papan
        const tileColors = [
            'bg-rose-100 text-rose-700 border-rose-200',
            'bg-sky-100 text-sky-700 border-sky-200',
            'bg-emerald-100 text-emerald-700 border-emerald-200',
            'bg-amber-100 text-amber-700 border-amber-200',
            'bg-purple-100 text-purple-700 border-purple-200'
        ];

        // Ikon Dadu FontAwesome
        const diceIcons = [
            'fa-dice-one',
            'fa-dice-two',
            'fa-dice-three',
            'fa-dice-four',
            'fa-dice-five',
            'fa-dice-six'
        ];

        // State Permainan
        let humanPos = 1;
        let botPos = 1;
        let isHumanTurn = true;
        let isRolling = false;
        let tileCoordinates = {}; // Menyimpan posisi koordinat center % dari tiap petak 1-100

        // Inisialisasi Audio Synthesizer sederhana menggunakan Tone.js
        const synth = new Tone.Synth().toDestination();

        function playSound(type) {
            try {
                if (Tone.context.state !== 'running') {
                    Tone.start();
                }
                if (type === 'roll') {
                    synth.triggerAttackRelease("C5", "8n");
                } else if (type === 'step') {
                    synth.triggerAttackRelease("E5", "16n");
                } else if (type === 'ladder') {
                    synth.triggerAttackRelease("G5", "4n");
                } else if (type === 'snake') {
                    synth.triggerAttackRelease("G3", "4n");
                } else if (type === 'win') {
                    const now = Tone.now();
                    synth.triggerAttackRelease("C5", "8n", now);
                    synth.triggerAttackRelease("E5", "8n", now + 0.15);
                    synth.triggerAttackRelease("G5", "8n", now + 0.3);
                    synth.triggerAttackRelease("C6", "4n", now + 0.45);
                }
            } catch(e) {
                // Abaikan jika audio diblokir browser
            }
        }

        // --- CORE FUNCTIONS: INSIALISASI PAPAN ---
        function createBoard() {
            const boardEl = document.getElementById('board');
            boardEl.innerHTML = '';

            // Membuat grid berpola ular (zigzag dari bawah ke atas)
            for (let row = 0; row < BOARD_SIZE; row++) {
                const isEvenRowFromBottom = row % 2 === 0;
                const rowNum = BOARD_SIZE - 1 - row;

                for (let col = 0; col < BOARD_SIZE; col++) {
                    const colNum = isEvenRowFromBottom ? col : (BOARD_SIZE - 1 - col);
                    const tileNumber = (rowNum * BOARD_SIZE) + colNum + 1;

                    // Hitung koordinat pusat % untuk menggambar SVG & bidak
                    const xPercent = (col * 10) + 5;
                    const yPercent = (row * 10) + 5;
                    tileCoordinates[tileNumber] = { x: xPercent, y: yPercent };

                    // Buat Element Kotak Papan
                    const tileDiv = document.createElement('div');
                    const colorClass = tileColors[(tileNumber - 1) % tileColors.length];
                    
                    tileDiv.className = `relative flex flex-col justify-between p-1 border ${colorClass} font-semibold transition-all select-none`;
                    
                    // Nomor Petak
                    const numSpan = document.createElement('span');
                    numSpan.className = 'text-[10px] sm:text-xs font-extrabold opacity-75';
                    numSpan.innerText = tileNumber;
                    tileDiv.appendChild(numSpan);

                    // Penanda Khusus Ular / Tangga Sederhana di Sudut Kotak
                    if (ladders[tileNumber]) {
                        const icon = document.createElement('i');
                        icon.className = 'fa-solid fa-arrow-up-long text-emerald-600 text-[10px] sm:text-xs self-end opacity-80';
                        tileDiv.appendChild(icon);
                    } else if (snakes[tileNumber]) {
                        const icon = document.createElement('i');
                        icon.className = 'fa-solid fa-arrow-down-long text-rose-500 text-[10px] sm:text-xs self-end opacity-80';
                        tileDiv.appendChild(icon);
                    }

                    boardEl.appendChild(tileDiv);
                }
            }
        }

        // --- DRAW SVG OVERLAY (ULAR & TANGGA BEBERAPA DEKORASI TRANSPARAN) ---
        function drawElements() {
            const svg = document.getElementById('svgOverlay');
            svg.innerHTML = ''; // Clear SVG

            // 1. Gambar Tangga (Warna Emas/Cokelat Segar dengan Garis Transparan)
            for (const [start, end] of Object.entries(ladders)) {
                const p1 = tileCoordinates[start];
                const p2 = tileCoordinates[end];

                // Convert % to SVG 1000x1000 space
                const x1 = p1.x * 10;
                const y1 = p1.y * 10;
                const x2 = p2.x * 10;
                const y2 = p2.y * 10;

                const g = document.createElementNS("http://www.w3.org/2000/svg", "g");
                g.setAttribute("opacity", "0.75");

                // Tiang Tangga 1
                const line1 = document.createElementNS("http://www.w3.org/2000/svg", "line");
                line1.setAttribute("x1", x1 - 12); line1.setAttribute("y1", y1);
                line1.setAttribute("x2", x2 - 12); line1.setAttribute("y2", y2);
                line1.setAttribute("stroke", "#d97706"); line1.setAttribute("stroke-width", "8");
                line1.setAttribute("stroke-linecap", "round");

                // Tiang Tangga 2
                const line2 = document.createElementNS("http://www.w3.org/2000/svg", "line");
                line2.setAttribute("x1", x1 + 12); line2.setAttribute("y1", y1);
                line2.setAttribute("x2", x2 + 12); line2.setAttribute("y2", y2);
                line2.setAttribute("stroke", "#d97706"); line2.setAttribute("stroke-width", "8");
                line2.setAttribute("stroke-linecap", "round");

                g.appendChild(line1);
                g.appendChild(line2);

                // Anak Tangga (Rungs)
                const steps = 6;
                for (let i = 1; i < steps; i++) {
                    const rx1 = x1 - 12 + (x2 - x1) * (i / steps);
                    const ry1 = y1 + (y2 - y1) * (i / steps);
                    const rx2 = x1 + 12 + (x2 - x1) * (i / steps);
                    const ry2 = y1 + (y2 - y1) * (i / steps);

                    const rung = document.createElementNS("http://www.w3.org/2000/svg", "line");
                    rung.setAttribute("x1", rx1); rung.setAttribute("y1", ry1);
                    rung.setAttribute("x2", rx2); rung.setAttribute("y2", ry2);
                    rung.setAttribute("stroke", "#b45309"); rung.setAttribute("stroke-width", "6");
                    g.appendChild(rung);
                }

                svg.appendChild(g);
            }

            // 2. Gambar Ular (Warna Merah/Hijau Bergelombang Tipis & Transparan)
            for (const [start, end] of Object.entries(snakes)) {
                const p1 = tileCoordinates[start]; // Kepala
                const p2 = tileCoordinates[end];   // Ekor

                const x1 = p1.x * 10; const y1 = p1.y * 10;
                const x2 = p2.x * 10; const y2 = p2.y * 10;

                // Hitung Kontrol Kurva S sederhan untuk membuat efek bergelombang
                const midX = (x1 + x2) / 2 + (x1 > x2 ? 60 : -60);
                const midY = (y1 + y2) / 2;

                const pathD = `M ${x1} ${y1} Q ${midX} ${midY} ${x2} ${y2}`;

                const g = document.createElementNS("http://www.w3.org/2000/svg", "g");
                g.setAttribute("opacity", "0.65");

                // Badan Ular
                const snakeBody = document.createElementNS("http://www.w3.org/2000/svg", "path");
                snakeBody.setAttribute("d", pathD);
                snakeBody.setAttribute("stroke", "#ef4444");
                snakeBody.setAttribute("stroke-width", "14");
                snakeBody.setAttribute("fill", "none");
                snakeBody.setAttribute("stroke-linecap", "round");

                // Garis tengah hiasan ular
                const snakeStripe = document.createElementNS("http://www.w3.org/2000/svg", "path");
                snakeStripe.setAttribute("d", pathD);
                snakeStripe.setAttribute("stroke", "#fef08a");
                snakeStripe.setAttribute("stroke-width", "4");
                snakeStripe.setAttribute("fill", "none");
                snakeStripe.setAttribute("stroke-dasharray", "8,8");

                // Kepala Ular (Lingkaran kecil di titik start)
                const head = document.createElementNS("http://www.w3.org/2000/svg", "circle");
                head.setAttribute("cx", x1);
                head.setAttribute("cy", y1);
                head.setAttribute("r", "12");
                head.setAttribute("fill", "#dc2626");

                g.appendChild(snakeBody);
                g.appendChild(snakeStripe);
                g.appendChild(head);

                svg.appendChild(g);
            }
        }

        // --- UPDATE POSISI BIDAK DILAYAR ---
        function updateTokenPositions() {
            const humanEl = document.getElementById('playerHuman');
            const botEl = document.getElementById('playerBot');

            const hCoord = tileCoordinates[humanPos];
            const bCoord = tileCoordinates[botPos];

            if (hCoord) {
                humanEl.style.opacity = '1';
                // Berikan sedikit offset jika dua bidak berada di petak yang sama
                const offset = (humanPos === botPos) ? -2.5 : 0;
                humanEl.style.left = `calc(${hCoord.x}% - 3.5% + ${offset}%)`;
                humanEl.style.top = `calc(${hCoord.y}% - 3.5%)`;
            }

            if (bCoord) {
                botEl.style.opacity = '1';
                const offset = (humanPos === botPos) ? 2.5 : 0;
                botEl.style.left = `calc(${bCoord.x}% - 3.5% + ${offset}%)`;
                botEl.style.top = `calc(${bCoord.y}% - 3.5%)`;
            }

            // Update Teks Status Petak
            document.getElementById('posHuman').innerText = humanPos;
            document.getElementById('posBot').innerText = botPos;
        }

        // --- SISTEM LOG ALUR PERMAINAN ---
        function addLog(message, type = 'info') {
            const logEl = document.getElementById('gameLog');
            const entry = document.createElement('div');
            
            let bgClass = 'bg-slate-50 border-slate-200 text-slate-700';
            if (type === 'human') bgClass = 'bg-rose-50 border-rose-200 text-rose-700 font-medium';
            if (type === 'bot') bgClass = 'bg-cyan-50 border-cyan-200 text-cyan-700 font-medium';
            if (type === 'ladder') bgClass = 'bg-emerald-50 border-emerald-300 text-emerald-800 font-bold';
            if (type === 'snake') bgClass = 'bg-amber-50 border-amber-300 text-amber-800 font-bold';

            entry.className = `p-2 rounded-lg border text-xs leading-relaxed transition-all ${bgClass}`;
            entry.innerHTML = message;

            logEl.appendChild(entry);
            logEl.scrollTop = logEl.scrollHeight;
        }

        // --- MEKANISME DADU & GILIRAN ---
        function rollDice() {
            return Math.floor(Math.random() * 6) + 1;
        }

        function animateDice(finalValue, callback) {
            const diceEl = document.getElementById('dice');
            diceEl.classList.add('rolling');
            playSound('roll');

            let counter = 0;
            const interval = setInterval(() => {
                const randomVal = Math.floor(Math.random() * 6);
                diceEl.innerHTML = `<i class="fa-solid ${diceIcons[randomVal]}"></i>`;
                counter++;
                if (counter > 10) {
                    clearInterval(interval);
                    diceEl.classList.remove('rolling');
                    diceEl.innerHTML = `<i class="fa-solid ${diceIcons[finalValue - 1]}"></i>`;
                    if (callback) callback();
                }
            }, 50);
        }

        // Melakukan langkah secara bertahap (satu persatu)
        async function movePlayerStepByStep(isHuman, steps) {
            let currentPos = isHuman ? humanPos : botPos;
            const name = isHuman ? 'Anda' : 'Soobin Bot';

            for (let i = 0; i < steps; i++) {
                if (currentPos >= TOTAL_TILES) break;
                currentPos++;
                
                if (isHuman) humanPos = currentPos;
                else botPos = currentPos;

                updateTokenPositions();
                playSound('step');
                await new Promise(r => setTimeout(r, 250)); // Jeda antar langkah
            }

            // Cek apakah mendarat di Tangga atau Ular
            if (ladders[currentPos]) {
                const destination = ladders[currentPos];
                addLog(`✨ WOW! ${name} menemukan tangga di petak ${currentPos} dan naik ke petak ${destination}!`, 'ladder');
                playSound('ladder');
                await new Promise(r => setTimeout(r, 400));
                
                if (isHuman) humanPos = destination;
                else botPos = destination;
                
                updateTokenPositions();
            } else if (snakes[currentPos]) {
                const destination = snakes[currentPos];
                addLog(`OUCH! ${name} tergigit ular di petak ${currentPos} dan turun ke petak ${destination}!`, 'snake');
                playSound('snake');
                await new Promise(r => setTimeout(r, 400));

                if (isHuman) humanPos = destination;
                else botPos = destination;

                updateTokenPositions();
            }

            // Cek Kemenangan
            if ((isHuman ? humanPos : botPos) === TOTAL_TILES) {
                showWinModal(isHuman);
                return true;
            }

            return false;
        }

        // Eksekusi Giliran Manusia
        async function handleHumanTurn() {
            if (!isHumanTurn || isRolling) return;
            isRolling = true;
            document.getElementById('btnRoll').disabled = true;

            const diceVal = rollDice();
            addLog(`🎲 Anda melempar dadu dan mendapatkan angka <b>${diceVal}</b>`, 'human');

            animateDice(diceVal, async () => {
                // Cek melebihi angka 100
                if (humanPos + diceVal > TOTAL_TILES) {
                    addLog(` ⚡ Angka dadu terlalu besar untuk mencapai 100. Giliran dilewati.`, 'human');
                    await new Promise(r => setTimeout(r, 600));
                    switchTurn(false);
                } else {
                    const won = await movePlayerStepByStep(true, diceVal);
                    if (!won) {
                        switchTurn(false); // Oper giliran ke Soobin Bot
                    }
                }
            });
        }

        // Eksekusi Giliran Bot (Soobin Bot)
        async function handleBotTurn() {
            addLog(`🤖 <b>Soobin Bot</b> sedang berpikir...`, 'bot');
            await new Promise(r => setTimeout(r, 1000)); // Simulasi jeda berpikir bot

            const diceVal = rollDice();
            addLog(`🎲 <b>Soobin Bot</b> melempar dadu dan mendapatkan angka <b>${diceVal}</b>`, 'bot');

            animateDice(diceVal, async () => {
                if (botPos + diceVal > TOTAL_TILES) {
                    addLog(`⚡ Langkah Soobin Bot melebihi 100. Giliran dilewati.`, 'bot');
                    await new Promise(r => setTimeout(r, 600));
                    switchTurn(true);
                } else {
                    const won = await movePlayerStepByStep(false, diceVal);
                    if (!won) {
                        switchTurn(true); // Oper giliran kembali ke Manusia
                    }
                }
            });
        }

        // Pergantian Giliran
        function switchTurn(toHuman) {
            isHumanTurn = toHuman;
            isRolling = false;

            const indicator = document.getElementById('turnIndicator');
            const cardHuman = document.getElementById('cardHuman');
            const cardBot = document.getElementById('cardBot');
            const btnRoll = document.getElementById('btnRoll');

            if (isHumanTurn) {
                indicator.innerHTML = 'Giliran: <span class="font-bold text-rose-600">Anda</span>';
                cardHuman.classList.remove('opacity-75', 'border-transparent');
                cardHuman.classList.add('border-rose-300');
                cardBot.classList.add('opacity-75');
                cardBot.classList.remove('border-cyan-300');
                btnRoll.disabled = false;
            } else {
                indicator.innerHTML = 'Giliran: <span class="font-bold text-cyan-600">Soobin Bot</span>';
                cardBot.classList.remove('opacity-75', 'border-transparent');
                cardBot.classList.add('border-cyan-300');
                cardHuman.classList.add('opacity-75');
                cardHuman.classList.remove('border-rose-300');
                btnRoll.disabled = true;

                // Jalankan bot otomatis
                setTimeout(handleBotTurn, 800);
            }
        }

        // Modal Kemenangan
        function showWinModal(isHumanWin) {
            playSound('win');
            const modal = document.getElementById('winModal');
            const icon = document.getElementById('winIcon');
            const title = document.getElementById('winTitle');
            const msg = document.getElementById('winMessage');

            if (isHumanWin) {
                icon.className = "w-20 h-20 mx-auto rounded-full bg-amber-100 text-amber-500 flex items-center justify-center text-4xl mb-4 shadow-inner";
                icon.innerHTML = `<i class="fa-solid fa-trophy"></i>`;
                title.innerText = "Selamat, Anda Menang!";
                msg.innerText = "Hore! Anda berhasil mendahului Soobin Bot mencapai petak 100!";
                addLog(`🏆 <b>SELAMAT! Anda memenangkan permainan!</b>`, 'human');
            } else {
                icon.className = "w-20 h-20 mx-auto rounded-full bg-cyan-100 text-cyan-600 flex items-center justify-center text-4xl mb-4 shadow-inner";
                icon.innerHTML = `<i class="fa-solid fa-robot"></i>`;
                title.innerText = "Soobin Bot Menang!";
                msg.innerText = "Jangan menyerah! Soobin Bot berhasil mencapai petak 100 lebih dulu.";
                addLog(`🤖 <b>Soobin Bot memenangkan permainan!</b>`, 'bot');
            }

            modal.classList.remove('hidden');
        }

        // Reset Game
        function resetGame() {
            humanPos = 1;
            botPos = 1;
            isHumanTurn = true;
            isRolling = false;

            document.getElementById('winModal').classList.add('hidden');
            document.getElementById('gameLog').innerHTML = `
                <div class="p-2 bg-slate-50 rounded-lg text-slate-500 italic">Game telah di-reset. Silakan klik "Lempar Dadu" untuk mulai kembali!</div>
            `;

            updateTokenPositions();
            switchTurn(true);
        }

        // Responsive redraw untuk garis overlay SVG saat window di-resize
        window.addEventListener('resize', () => {
            createBoard();
            drawElements();
            updateTokenPositions();
        });

        // Window Load Initializer
        window.onload = function() {
            createBoard();
            drawElements();
            updateTokenPositions();
        };
    </script>
</body>
</html>
```
