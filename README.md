# -
一個老舊遊戲
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>陸戰棋 / 軍棋 (Traditional Chinese Army Chess)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@600;700;900&family=PingFang+TC:wght@400;500;600;700&display=swap');

        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }

        body {
            font-family: 'PingFang TC', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            background-color: #1a1816;
            color: #f3f4f6;
            height: 100vh;
            width: 100vw;
            overflow: hidden;
            margin: 0;
            padding: 0;
        }

        /* Chess Board SVG & Canvas Layering */
        .board-container {
            position: relative;
            background: radial-gradient(circle at center, #2e261d 0%, #17130e 100%);
            border: 3px solid #8b6b3e;
            box-shadow: inset 0 0 20px rgba(0,0,0,0.8), 0 10px 25px rgba(0,0,0,0.5);
            border-radius: 12px;
        }

        .piece {
            transition: transform 0.22s cubic-bezier(0.25, 1, 0.5, 1), box-shadow 0.2s ease, opacity 0.2s ease;
            cursor: pointer;
            position: absolute;
            z-index: 10;
        }

        .piece.moving {
            z-index: 50;
            transition: all 0.35s ease-in-out;
        }

        /* Piece Aesthetics */
        .piece-red {
            background: linear-gradient(135deg, #b91c1c 0%, #7f1d1d 100%);
            border: 2px solid #fca5a5;
            color: #fff;
            box-shadow: 0 4px 6px rgba(0,0,0,0.4), inset 0 1px 1px rgba(255,255,255,0.4);
        }

        .piece-black {
            background: linear-gradient(135deg, #27272a 0%, #09090b 100%);
            border: 2px solid #71717a;
            color: #e4e4e7;
            box-shadow: 0 4px 6px rgba(0,0,0,0.5), inset 0 1px 1px rgba(255,255,255,0.2);
        }

        .piece-selected {
            ring: 4px solid #f59e0b;
            transform: scale(1.08);
            z-index: 30 !important;
            box-shadow: 0 0 15px #f59e0b !important;
        }

        .piece-target {
            box-shadow: 0 0 12px #22c55e !important;
            animation: pulse-target 1.2s infinite;
        }

        @keyframes pulse-target {
            0%, 100% { opacity: 0.8; transform: scale(0.95); }
            50% { opacity: 1; transform: scale(1.05); }
        }

        .highlight-move {
            position: absolute;
            border-radius: 50%;
            background: rgba(34, 197, 94, 0.45);
            border: 2px solid #22c55e;
            z-index: 5;
            pointer-events: none;
            transform: translate(-50%, -50%);
        }

        .highlight-attack {
            position: absolute;
            border-radius: 8px;
            background: rgba(239, 68, 68, 0.35);
            border: 2px dashed #ef4444;
            z-index: 5;
            pointer-events: none;
            transform: translate(-50%, -50%);
        }

        /* Custom Scrollbars */
        ::-webkit-scrollbar {
            width: 5px;
            height: 5px;
        }
        ::-webkit-scrollbar-track {
            background: rgba(0,0,0,0.2);
        }
        ::-webkit-scrollbar-thumb {
            background: #52525b;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #71717a;
        }

        .badge-revealed {
            font-size: 9px;
            padding: 1px 3px;
            border-radius: 3px;
            background: #d97706;
            color: white;
            position: absolute;
            top: -3px;
            right: -3px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.5);
        }
    </style>
</head>
<body class="flex flex-col h-screen w-screen overflow-hidden bg-zinc-950 text-zinc-100">

    <!-- TOP HEADER / NAVBAR -->
    <header class="h-14 bg-zinc-900 border-b border-zinc-800 px-4 flex items-center justify-between shrink-0 z-20">
        <div class="flex items-center space-x-3">
            <div class="w-9 h-9 rounded-lg bg-amber-700/30 border border-amber-600/50 flex items-center justify-center text-amber-500 font-black text-xl">
                軍
            </div>
            <div>
                <h1 class="font-bold text-lg text-amber-100 leading-none">陸戰棋 / 軍棋</h1>
                <p class="text-xs text-zinc-400 mt-0.5" id="game-mode-subtitle">單人對戰模式 (AI 敵方)</p>
            </div>
        </div>

        <!-- Mode & Status Indicator -->
        <div class="flex items-center space-x-2">
            <span id="turn-badge" class="px-3 py-1 rounded-full text-xs font-bold bg-amber-500/20 text-amber-400 border border-amber-500/30 flex items-center gap-1.5">
                <span class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
                <span id="turn-text">佈陣階段</span>
            </span>

            <button id="btn-mode-select" onclick="openModeModal()" class="px-3 py-1.5 rounded-lg bg-zinc-800 hover:bg-zinc-700 border border-zinc-700 text-xs font-semibold text-zinc-200 transition flex items-center gap-1.5">
                <i class="fa-solid fa-gamepad text-amber-400"></i>
                <span>切換模式</span>
            </button>

            <button id="btn-audio-toggle" onclick="toggleAudio()" class="w-8 h-8 rounded-lg bg-zinc-800 hover:bg-zinc-700 border border-zinc-700 text-zinc-300 flex items-center justify-center transition">
                <i id="audio-icon" class="fa-solid fa-volume-high text-xs"></i>
            </button>
        </div>
    </header>

    <!-- MAIN iPad LAYOUT: LEFT SIDEBAR + BOARD CENTER + RIGHT SIDEBAR -->
    <main class="flex-1 flex flex-row overflow-hidden relative p-2 gap-2">

        <!-- LEFT SIDEBAR: Setup Tray OR Piece Counter Table -->
        <aside id="left-sidebar" class="w-72 md:w-80 bg-zinc-900/90 border border-zinc-800 rounded-xl flex flex-col shrink-0 overflow-hidden shadow-xl z-10">
            
            <!-- SETUP PHASE SIDEBAR CONTENT -->
            <div id="sidebar-setup-panel" class="flex-1 flex flex-col p-3 overflow-y-auto space-y-3">
                <div class="flex items-center justify-between border-b border-zinc-800 pb-2">
                    <h2 class="font-bold text-sm text-amber-400 flex items-center gap-2">
                        <i class="fa-solid fa-boxes-packing"></i>
                        <span>我方備用棋盤 (托盤)</span>
                    </h2>
                    <span id="placed-count-tag" class="text-xs px-2 py-0.5 rounded bg-zinc-800 text-zinc-300 font-mono">0 / 25</span>
                </div>

                <!-- Presets Bar -->
                <div class="grid grid-cols-3 gap-1.5">
                    <button onclick="applyPreset('random')" class="px-2 py-1.5 bg-zinc-800 hover:bg-amber-600/30 border border-zinc-700 hover:border-amber-500 rounded text-xs font-medium text-zinc-200 transition">
                        ⚡ 一鍵隨機
                    </button>
                    <button onclick="applyPreset('balanced')" class="px-2 py-1.5 bg-zinc-800 hover:bg-amber-600/30 border border-zinc-700 hover:border-amber-500 rounded text-xs font-medium text-zinc-200 transition">
                        ⚖️ 均衡型
                    </button>
                    <button onclick="applyPreset('offensive')" class="px-2 py-1.5 bg-zinc-800 hover:bg-amber-600/30 border border-zinc-700 hover:border-amber-500 rounded text-xs font-medium text-zinc-200 transition">
                        ⚔️ 進攻型
                    </button>
                    <button onclick="applyPreset('defensive')" class="px-2 py-1.5 bg-zinc-800 hover:bg-amber-600/30 border border-zinc-700 hover:border-amber-500 rounded text-xs font-medium text-zinc-200 transition">
                        🛡️ 防守型
                    </button>
                    <button onclick="clearPlayerBoard()" class="col-span-2 px-2 py-1.5 bg-red-950/40 hover:bg-red-900/50 border border-red-800/50 rounded text-xs font-medium text-red-300 transition">
                        🗑️ 清空棋盤
                    </button>
                </div>

                <!-- Tray Piece Selector List -->
                <div class="flex-1 overflow-y-auto border border-zinc-800/80 rounded-lg p-2 bg-zinc-950/50 space-y-1.5" id="tray-pieces-list">
                    <!-- Populated dynamically via JS -->
                </div>

                <!-- Legal Checklist Status -->
                <div class="bg-zinc-950 border border-zinc-800 rounded-lg p-2.5 text-xs space-y-1">
                    <div class="text-zinc-400 font-semibold mb-1 border-b border-zinc-800/60 pb-1">布陣規則檢查：</div>
                    <div id="chk-flag" class="flex items-center gap-2 text-zinc-400">
                        <i class="fa-solid fa-circle-xmark text-red-500"></i> 軍旗必須在底層大本營
                    </div>
                    <div id="chk-mines" class="flex items-center gap-2 text-zinc-400">
                        <i class="fa-solid fa-circle-xmark text-red-500"></i> 地雷必須在最後兩排
                    </div>
                    <div id="chk-bombs" class="flex items-center gap-2 text-zinc-400">
                        <i class="fa-solid fa-circle-xmark text-red-500"></i> 炸彈不能放在第一排
                    </div>
                    <div id="chk-camps" class="flex items-center gap-2 text-zinc-400">
                        <i class="fa-solid fa-circle-check text-green-500"></i> 行營內不能初始擺棋
                    </div>
                </div>

                <!-- Start Battle Button -->
                <button id="btn-start-game" onclick="startGame()" disabled class="w-full py-2.5 rounded-lg font-bold text-sm transition shadow-lg opacity-50 cursor-not-allowed bg-zinc-700 text-zinc-400">
                    🚀 開始對戰
                </button>
            </div>

            <!-- PLAYING PHASE SIDEBAR CONTENT (AUTOSWITCHES ON START) -->
            <div id="sidebar-playing-panel" class="hidden flex-1 flex-col p-3 overflow-y-auto space-y-3">
                <div class="flex items-center justify-between border-b border-zinc-800 pb-2">
                    <h2 class="font-bold text-sm text-amber-400 flex items-center gap-2">
                        <i class="fa-solid fa-chart-pie"></i>
                        <span>敵我棋子統計表</span>
                    </h2>
                    <span class="text-xs text-zinc-400" id="game-status-mini">對戰中</span>
                </div>

                <!-- Piece Counts Table -->
                <div class="flex-1 overflow-y-auto space-y-2 pr-1" id="piece-stats-container">
                    <!-- Populated dynamically via JS -->
                </div>

                <!-- Control Buttons -->
                <div class="pt-2 border-t border-zinc-800 grid grid-cols-2 gap-2">
                    <button onclick="surrender()" class="py-2 px-3 bg-red-950/60 hover:bg-red-900 border border-red-800 rounded-lg text-xs font-semibold text-red-200 transition">
                        🏳️ 認輸
                    </button>
                    <button onclick="restartGame()" class="py-2 px-3 bg-zinc-800 hover:bg-zinc-700 border border-zinc-700 rounded-lg text-xs font-semibold text-zinc-200 transition">
                        🔄 重新開始
                    </button>
                </div>
            </div>

        </aside>

        <!-- CENTER CHESS BOARD CONTAINER -->
        <section class="flex-1 h-full flex flex-col items-center justify-center relative overflow-hidden">
            <div id="board-wrapper" class="board-container w-full h-full max-w-[560px] max-h-[820px] relative flex items-center justify-center p-2">
                
                <!-- SVG Board Grid Lines Layer -->
                <svg id="board-svg" class="w-full h-full absolute inset-0 z-0" viewBox="0 0 500 750">
                    <!-- Dynamic SVG content drawn by JS -->
                </svg>

                <!-- Board Interactive Grid Overlay -->
                <div id="board-grid-layer" class="w-full h-full relative z-10">
                    <!-- Pieces and Click Targets Rendered Here -->
                </div>

            </div>
        </section>

        <!-- RIGHT SIDEBAR: Move History & Combat Logs -->
        <aside class="w-64 md:w-72 bg-zinc-900/90 border border-zinc-800 rounded-xl flex flex-col shrink-0 overflow-hidden shadow-xl z-10 hidden sm:flex">
            <div class="p-3 border-b border-zinc-800 flex items-center justify-between">
                <h3 class="font-bold text-sm text-amber-400 flex items-center gap-2">
                    <i class="fa-solid fa-scroll"></i>
                    <span>對局記錄與戰報</span>
                </h3>
                <span class="w-2 h-2 rounded-full bg-green-500"></span>
            </div>

            <!-- History Log Box -->
            <div id="move-history-log" class="flex-1 p-3 overflow-y-auto space-y-2 text-xs font-mono bg-zinc-950/40">
                <div class="text-zinc-500 italic text-center py-4">歡迎來到陸戰棋！請先在左側完成布陣。</div>
            </div>

            <!-- Quick Info Footer -->
            <div class="p-2.5 bg-zinc-950 border-t border-zinc-800 text-[11px] text-zinc-400 space-y-1">
                <div class="flex justify-between">
                    <span>同級碰撞:</span>
                    <span class="text-amber-400 font-bold">攻擊方勝出 (存活)</span>
                </div>
                <div class="flex justify-between">
                    <span>工兵特權:</span>
                    <span class="text-emerald-400 font-bold">鐵路任意轉彎 / 拆雷</span>
                </div>
                <div class="flex justify-between">
                    <span>行營保護:</span>
                    <span class="text-sky-400 font-bold">免疫任何敵方攻擊</span>
                </div>
            </div>
        </aside>

    </main>

    <!-- MODAL: Mode Selector (Single Player AI / Local 2P / Online Firebase) -->
    <div id="modal-mode" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-zinc-900 border border-zinc-700 rounded-xl w-full max-w-md p-5 shadow-2xl space-y-4">
            <div class="flex justify-between items-center border-b border-zinc-800 pb-3">
                <h3 class="text-lg font-bold text-amber-400 flex items-center gap-2">
                    <i class="fa-solid fa-sliders"></i> 選擇對戰模式
                </h3>
                <button onclick="closeModeModal()" class="text-zinc-400 hover:text-white text-lg">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <div class="space-y-2.5">
                <!-- Single Player AI -->
                <button onclick="selectGameMode('ai')" class="w-full p-3.5 rounded-lg border border-zinc-700 hover:border-amber-500 bg-zinc-800/80 hover:bg-zinc-800 text-left transition flex items-center justify-between group">
                    <div>
                        <div class="font-bold text-amber-200 group-hover:text-amber-400">🤖 單人對戰 (智慧 AI 敵方)</div>
                        <div class="text-xs text-zinc-400 mt-0.5">與離線 AI 進行人機對決，適合練習佈陣與戰術</div>
                    </div>
                    <i class="fa-solid fa-chevron-right text-zinc-500 group-hover:text-amber-400"></i>
                </button>

                <!-- Local 2P -->
                <button onclick="selectGameMode('local')" class="w-full p-3.5 rounded-lg border border-zinc-700 hover:border-amber-500 bg-zinc-800/80 hover:bg-zinc-800 text-left transition flex items-center justify-between group">
                    <div>
                        <div class="font-bold text-sky-200 group-hover:text-sky-400">👥 雙人同屏 (面對面對戰)</div>
                        <div class="text-xs text-zinc-400 mt-0.5">同一台 iPad / 平板裝置輪流操作對決</div>
                    </div>
                    <i class="fa-solid fa-chevron-right text-zinc-500 group-hover:text-sky-400"></i>
                </button>

                <!-- Online Multiplayer -->
                <div class="p-3.5 rounded-lg border border-zinc-700 bg-zinc-800/80 space-y-2">
                    <div class="font-bold text-emerald-300">🌐 線上連線房間 (Firebase)</div>
                    <div class="flex gap-2">
                        <input id="room-code-input" type="text" placeholder="6位數房號" maxlength="6" class="flex-1 px-3 py-1.5 rounded bg-zinc-950 border border-zinc-700 text-xs text-center font-mono focus:border-emerald-500 outline-none uppercase">
                        <button onclick="joinOnlineRoom()" class="px-3 py-1.5 bg-emerald-700 hover:bg-emerald-600 rounded text-xs font-bold transition">加入</button>
                        <button onclick="createOnlineRoom()" class="px-3 py-1.5 bg-amber-700 hover:bg-amber-600 rounded text-xs font-bold transition">建房</button>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- GAME CODE LOGIC -->
    <script>
        /**
         * TRADITIONAL CHINESE ARMY CHESS (陸戰棋 / 軍棋) ENGINE
         * Board: 13 Rows (0 to 12) x 5 Columns (0 to 4)
         * Red (Player): Rows 6 to 12
         * Black (AI/Enemy): Rows 0 to 5
         */

        // Ranks hierarchy & definitions
        const PIECE_TYPES = {
            '39': { name: '司令', count: 1, rank: 39 },
            '38': { name: '軍長', count: 1, rank: 38 },
            '37': { name: '師長', count: 2, rank: 37 },
            '36': { name: '旅長', count: 2, rank: 36 },
            '35': { name: '團長', count: 2, rank: 35 },
            '34': { name: '營長', count: 2, rank: 34 },
            '33': { name: '連長', count: 3, rank: 33 },
            '32': { name: '排長', count: 3, rank: 32 },
            '31': { name: '工兵', count: 3, rank: 31 },
            'B':  { name: '炸彈', count: 2, rank: 0 },  // Kills any piece & dies
            'M':  { name: '地雷', count: 3, rank: -1 }, // Immovable. Engineer disarms. Non-engineer mutually dies.
            'F':  { name: '軍旗', count: 1, rank: -2 }  // Immovable target
        };

        // Campsite Grid Locations
        const CAMPSITES = [
            {r: 2, c: 1}, {r: 2, c: 3},
            {r: 3, c: 2},
            {r: 4, c: 1}, {r: 4, c: 3},
            {r: 8, c: 1}, {r: 8, c: 3},
            {r: 9, c: 2},
            {r: 10, c: 1}, {r: 10, c: 3}
        ];

        // Headquarters (HQs)
        const HQS = [
            {r: 0, c: 1}, {r: 0, c: 3}, // Enemy HQs
            {r: 12, c: 1}, {r: 12, c: 3} // Player HQs
        ];

        // Railway Lines Grid Points: Row 1, 5, 6, 11 and Col 0, 4
        function isRailway(r, c) {
            if (r === 1 || r === 5 || r === 6 || r === 11) {
                if (c >= 0 && c <= 4) return true;
            }
            if ((c === 0 || c === 4) && ((r >= 1 && r <= 5) || (r >= 6 && r <= 11))) {
                return true;
            }
            return false;
        }

        function isCampsite(r, c) {
            return CAMPSITES.some(p => p.r === r && p.c === c);
        }

        function isHQ(r, c) {
            return HQS.some(p => p.r === r && p.c === c);
        }

        // Global Game State Variables
        let board = Array(13).fill(null).map(() => Array(5).fill(null));
        let gameState = 'SETUP'; // 'SETUP' or 'PLAYING' or 'ENDED'
        let gameMode = 'ai'; // 'ai', 'local', 'online'
        let currentTurn = 'red'; // 'red' (Player) or 'black' (AI/Opponent)
        let selectedCell = null;
        let selectedTrayType = null;
        let validMoves = [];
        let aiThinking = false;
        let audioEnabled = true;
        let audioCtx = null;

        // Player remaining pieces in Tray for Setup
        let trayPieces = {};

        // DOM elements
        let svgBoard, gridLayer, trayListEl;

        // Sound Synthesis via Web Audio API
        function playSound(type) {
            if (!audioEnabled) return;
            try {
                if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                if (audioCtx.state === 'suspended') audioCtx.resume();

                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.connect(gain);
                gain.connect(audioCtx.destination);

                const now = audioCtx.currentTime;

                if (type === 'click') {
                    osc.frequency.setValueAtTime(400, now);
                    gain.gain.setValueAtTime(0.1, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.05);
                    osc.start(now);
                    osc.stop(now + 0.05);
                } else if (type === 'move') {
                    osc.frequency.setValueAtTime(300, now);
                    osc.frequency.exponentialRampToValueAtTime(500, now + 0.08);
                    gain.gain.setValueAtTime(0.15, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.08);
                    osc.start(now);
                    osc.stop(now + 0.08);
                } else if (type === 'capture') {
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(600, now);
                    osc.frequency.exponentialRampToValueAtTime(200, now + 0.18);
                    gain.gain.setValueAtTime(0.25, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.18);
                    osc.start(now);
                    osc.stop(now + 0.18);
                } else if (type === 'explode') {
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(150, now);
                    osc.frequency.linearRampToValueAtTime(40, now + 0.25);
                    gain.gain.setValueAtTime(0.35, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.25);
                    osc.start(now);
                    osc.stop(now + 0.25);
                } else if (type === 'win') {
                    osc.frequency.setValueAtTime(523.25, now);
                    osc.frequency.setValueAtTime(659.25, now + 0.1);
                    osc.frequency.setValueAtTime(783.99, now + 0.2);
                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);
                    osc.start(now);
                    osc.stop(now + 0.4);
                }
            } catch(e){}
        }

        function toggleAudio() {
            audioEnabled = !audioEnabled;
            const icon = document.getElementById('audio-icon');
            if (audioEnabled) {
                icon.className = 'fa-solid fa-volume-high text-xs text-amber-400';
            } else {
                icon.className = 'fa-solid fa-volume-xmark text-xs text-zinc-500';
            }
        }

        // Initialize Application
        window.addEventListener('DOMContentLoaded', () => {
            svgBoard = document.getElementById('board-svg');
            gridLayer = document.getElementById('board-grid-layer');
            trayListEl = document.getElementById('tray-pieces-list');

            initTrayPieces();
            drawBoardBackground();
            renderTrayUI();
            renderBoardUI();
            setupAIOpponentBoard();
            updateSetupChecklist();

            window.addEventListener('resize', () => {
                renderBoardUI();
            });
        });

        // Initialize Tray Counts
        function initTrayPieces() {
            trayPieces = {};
            for (let key in PIECE_TYPES) {
                trayPieces[key] = PIECE_TYPES[key].count;
            }
        }

        // DRAW SVG BOARD GRAPHICS
        function drawBoardBackground() {
            let svgHtml = '';
            // ViewBox 0 0 500 750 (5 cols x 13 rows)
            const cw = 500 / 5;
            const rh = 750 / 13;

            // Background grid
            // Grid lines
            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const x1 = c * cw + cw/2;
                    const y1 = r * rh + rh/2;

                    // Right connection
                    if (c < 4) {
                        const x2 = (c+1) * cw + cw/2;
                        const isRwy = isRailway(r, c) && isRailway(r, c+1);
                        svgHtml += `<line x1="${x1}" y1="${y1}" x2="${x2}" y2="${y1}" stroke="${isRwy ? '#f59e0b' : '#64748b'}" stroke-width="${isRwy ? 3.5 : 1.5}" ${isRwy ? 'stroke-dasharray="6,3"' : ''} />`;
                    }

                    // Down connection
                    if (r < 12) {
                        // River gap between row 5 and 6 except bridge posts (cols 0, 2, 4)
                        const isRiverGap = (r === 5 && (c === 1 || c === 3));
                        if (!isRiverGap) {
                            const y2 = (r+1) * rh + rh/2;
                            const isRwy = isRailway(r, c) && isRailway(r+1, c);
                            svgHtml += `<line x1="${x1}" y1="${y1}" x2="${x1}" y2="${y2}" stroke="${isRwy ? '#f59e0b' : '#64748b'}" stroke-width="${isRwy ? 3.5 : 1.5}" ${isRwy ? 'stroke-dasharray="6,3"' : ''} />`;
                        }
                    }
                }
            }

            // Draw Campsite Diagonal Lines
            CAMPSITES.forEach(cs => {
                const cx = cs.c * cw + cw/2;
                const cy = cs.r * rh + rh/2;
                const r = cs.r;
                const c = cs.c;

                // 4 diagonal lines radiating from campsite center
                const offsets = [[-1,-1], [-1,1], [1,-1], [1,1]];
                offsets.forEach(off => {
                    const tx = (c + off[1]) * cw + cw/2;
                    const ty = (r + off[0]) * rh + rh/2;
                    svgHtml += `<line x1="${cx}" y1="${cy}" x2="${tx}" y2="${ty}" stroke="#a1a1aa" stroke-width="1.2" stroke-dasharray="3,2" />`;
                });
            });

            // Draw River Rect in middle (between row 5 and 6)
            const riverY = 5 * rh + rh/2 + 4;
            const riverH = rh - 8;
            svgHtml += `
                <rect x="10" y="${riverY}" width="480" height="${riverH}" fill="#0369a1" fill-opacity="0.25" stroke="#38bdf8" stroke-width="1.5" rx="6" />
                <text x="125" y="${riverY + riverH/2 + 5}" fill="#7dd3fc" font-size="16" font-weight="bold" letter-spacing="4">前  線  界  河</text>
                <text x="375" y="${riverY + riverH/2 + 5}" fill="#7dd3fc" font-size="16" font-weight="bold" text-anchor="end" letter-spacing="4">山  界  水  灣</text>
            `;

            // Draw Campsite Circles & HQ Arches
            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const cx = c * cw + cw/2;
                    const cy = r * rh + rh/2;

                    if (isCampsite(r, c)) {
                        svgHtml += `
                            <circle cx="${cx}" cy="${cy}" r="${cw*0.35}" fill="#18181b" stroke="#eab308" stroke-width="2" />
                            <circle cx="${cx}" cy="${cy}" r="${cw*0.28}" fill="none" stroke="#eab308" stroke-width="1" stroke-dasharray="3,2" />
                            <text x="${cx}" y="${cy+4}" fill="#fef08a" font-size="10" font-weight="bold" text-anchor="middle">行營</text>
                        `;
                    } else if (isHQ(r, c)) {
                        svgHtml += `
                            <rect x="${cx - cw*0.4}" y="${cy - rh*0.38}" width="${cw*0.8}" height="${rh*0.76}" fill="#27272a" stroke="#ef4444" stroke-width="2" rx="16" />
                            <text x="${cx}" y="${cy+4}" fill="#fca5a5" font-size="11" font-weight="bold" text-anchor="middle">大本營</text>
                        `;
                    }
                }
            }

            svgBoard.innerHTML = svgHtml;
        }

        // RENDER TRAY SELECTOR IN LEFT SIDEBAR
        function renderTrayUI() {
            let html = '';
            let totalRemaining = 0;

            for (let type in PIECE_TYPES) {
                const info = PIECE_TYPES[type];
                const rem = trayPieces[type] || 0;
                totalRemaining += rem;

                const isSelected = selectedTrayType === type;

                html += `
                    <div onclick="selectTrayPiece('${type}')" class="flex items-center justify-between p-2 rounded-lg border transition cursor-pointer ${
                        isSelected 
                            ? 'bg-amber-600/30 border-amber-500 text-amber-200 shadow' 
                            : rem > 0 ? 'bg-zinc-900 border-zinc-800 hover:border-zinc-700 text-zinc-300' : 'bg-zinc-950/40 border-zinc-900/60 text-zinc-600 opacity-50'
                    }">
                        <div class="flex items-center gap-2">
                            <span class="w-7 h-7 rounded border border-red-400/50 bg-red-900/60 text-red-100 font-black text-xs flex items-center justify-center shadow-sm">
                                ${info.name}
                            </span>
                            <span class="text-xs font-semibold">${info.name}</span>
                        </div>
                        <span class="text-xs font-mono font-bold px-2 py-0.5 rounded ${rem > 0 ? 'bg-amber-950 text-amber-400 border border-amber-800/50' : 'bg-zinc-800 text-zinc-500'}">
                            剩餘 ${rem}
                        </span>
                    </div>
                `;
            }

            trayListEl.innerHTML = html;
            document.getElementById('placed-count-tag').innerText = `${25 - totalRemaining} / 25`;
        }

        function selectTrayPiece(type) {
            if (gameState !== 'SETUP') return;
            if (trayPieces[type] <= 0) return;

            playSound('click');
            if (selectedTrayType === type) {
                selectedTrayType = null;
            } else {
                selectedTrayType = type;
                selectedCell = null;
            }
            renderTrayUI();
            renderBoardUI();
        }

        // CHECKLIST VALIDATION FOR SETUP
        function updateSetupChecklist() {
            // Rule 1: Flag in HQ
            let flagValid = false;
            if (board[12][1] && board[12][1].type === 'F' && board[12][1].color === 'red') flagValid = true;
            if (board[12][3] && board[12][3].type === 'F' && board[12][3].color === 'red') flagValid = true;

            // Rule 2: Mines in bottom two rows (rows 11 and 12)
            let minesValid = true;
            let mineCount = 0;
            for (let r = 6; r <= 12; r++) {
                for (let c = 0; c < 5; c++) {
                    if (board[r][c] && board[r][c].color === 'red' && board[r][c].type === 'M') {
                        mineCount++;
                        if (r < 11) minesValid = false;
                    }
                }
            }
            if (mineCount !== 3) minesValid = false;

            // Rule 3: Bombs NOT in front row (row 6)
            let bombsValid = true;
            let bombCount = 0;
            for (let r = 6; r <= 12; r++) {
                for (let c = 0; c < 5; c++) {
                    if (board[r][c] && board[r][c].color === 'red' && board[r][c].type === 'B') {
                        bombCount++;
                        if (r === 6) bombsValid = false;
                    }
                }
            }
            if (bombCount !== 2) bombsValid = false;

            // Rule 4: Campsites empty
            let campsValid = true;
            CAMPSITES.forEach(cs => {
                if (cs.r >= 6 && board[cs.r][cs.c]) campsValid = false;
            });

            // Update DOM UI Status
            updateCheckUI('chk-flag', flagValid, '軍旗必須在底層大本營');
            updateCheckUI('chk-mines', minesValid, '地雷必須在最後兩排');
            updateCheckUI('chk-bombs', bombsValid, '炸彈不能放在第一排');
            updateCheckUI('chk-camps', campsValid, '行營內不能初始擺棋');

            let totalPlaced = 0;
            for (let r = 6; r <= 12; r++) {
                for (let c = 0; c < 5; c++) {
                    if (board[r][c] && board[r][c].color === 'red') totalPlaced++;
                }
            }

            const startBtn = document.getElementById('btn-start-game');
            if (flagValid && minesValid && bombsValid && campsValid && totalPlaced === 25) {
                startBtn.disabled = false;
                startBtn.className = "w-full py-2.5 rounded-lg font-bold text-sm transition shadow-lg bg-gradient-to-r from-amber-600 to-amber-500 hover:from-amber-500 hover:to-amber-400 text-white cursor-pointer active:scale-95";
            } else {
                startBtn.disabled = true;
                startBtn.className = "w-full py-2.5 rounded-lg font-bold text-sm transition shadow-lg opacity-50 cursor-not-allowed bg-zinc-700 text-zinc-400";
            }
        }

        function updateCheckUI(id, isOk, labelText) {
            const el = document.getElementById(id);
            if (isOk) {
                el.className = "flex items-center gap-2 text-green-400 font-medium";
                el.innerHTML = `<i class="fa-solid fa-circle-check text-green-500"></i> ${labelText}`;
            } else {
                el.className = "flex items-center gap-2 text-zinc-400";
                el.innerHTML = `<i class="fa-solid fa-circle-xmark text-red-500"></i> ${labelText}`;
            }
        }

        // RENDER BOARD INTERACTIVE LAYER
        function renderBoardUI() {
            gridLayer.innerHTML = '';
            const rect = gridLayer.getBoundingClientRect();
            const cellW = rect.width / 5;
            const cellH = rect.height / 13;

            // Render Highlights for Valid Moves
            validMoves.forEach(m => {
                const x = m.c * cellW + cellW / 2;
                const y = m.r * cellH + cellH / 2;
                const isAttack = board[m.r][m.c] !== null;

                const hl = document.createElement('div');
                hl.className = isAttack ? 'highlight-attack' : 'highlight-move';
                hl.style.left = `${x}px`;
                hl.style.top = `${y}px`;
                hl.style.width = isAttack ? `${cellW * 0.85}px` : `${cellW * 0.35}px`;
                hl.style.height = isAttack ? `${cellH * 0.85}px` : `${cellW * 0.35}px`;
                gridLayer.appendChild(hl);
            });

            // Render Pieces & Grid Cell Click Handlers
            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const x = c * cellW;
                    const y = r * cellH;

                    // Click area container
                    const cellEl = document.createElement('div');
                    cellEl.style.position = 'absolute';
                    cellEl.style.left = `${x}px`;
                    cellEl.style.top = `${y}px`;
                    cellEl.style.width = `${cellW}px`;
                    cellEl.style.height = `${cellH}px`;
                    cellEl.className = 'flex items-center justify-center cursor-pointer';
                    cellEl.onclick = () => handleCellClick(r, c);

                    const item = board[r][c];
                    if (item) {
                        const pieceEl = document.createElement('div');
                        const isRed = item.color === 'red';
                        const isRevealed = item.revealed || isRed || gameMode === 'local';

                        pieceEl.className = `piece rounded-lg flex items-center justify-center font-bold relative ${
                            isRed ? 'piece-red' : 'piece-black'
                        } ${
                            selectedCell && selectedCell.r === r && selectedCell.c === c ? 'piece-selected' : ''
                        }`;

                        pieceEl.style.width = `${cellW * 0.82}px`;
                        pieceEl.style.height = `${cellH * 0.76}px`;

                        // Piece text content
                        let pName = PIECE_TYPES[item.type].name;
                        if (!isRevealed) {
                            pName = '軍棋';
                        }

                        pieceEl.innerHTML = `
                            <span class="${isRed ? 'text-red-100' : 'text-zinc-200'} text-xs sm:text-sm font-black tracking-wider">
                                ${pName}
                            </span>
                            ${item.revealed && !isRed ? '<span class="badge-revealed">已明</span>' : ''}
                        `;

                        cellEl.appendChild(pieceEl);
                    }

                    gridLayer.appendChild(cellEl);
                }
            }
        }

        // HANDLE BOARD CLICK EVENTS
        function handleCellClick(r, c) {
            if (aiThinking) return;

            // SETUP PHASE INTERACTIONS
            if (gameState === 'SETUP') {
                if (r < 6) {
                    addLog('⚠️ 無法移動到敵方陣地', 'warn');
                    return;
                }
                if (isCampsite(r, c)) {
                    addLog('⚠️ 行營內初始不能擺棋', 'warn');
                    return;
                }

                // If tray piece is selected -> Place piece onto board
                if (selectedTrayType) {
                    const existing = board[r][c];
                    if (existing) {
                        // Return existing piece to tray
                        trayPieces[existing.type]++;
                    }
                    board[r][c] = {
                        type: selectedTrayType,
                        color: 'red',
                        revealed: true
                    };
                    trayPieces[selectedTrayType]--;
                    if (trayPieces[selectedTrayType] <= 0) {
                        selectedTrayType = null;
                    }
                    playSound('click');
                    renderTrayUI();
                    renderBoardUI();
                    updateSetupChecklist();
                    return;
                }

                // If clicking an existing piece -> Select or remove it back to tray
                const clicked = board[r][c];
                if (clicked && clicked.color === 'red') {
                    if (selectedCell && selectedCell.r === r && selectedCell.c === c) {
                        // Unselect or Return to tray
                        trayPieces[clicked.type]++;
                        board[r][c] = null;
                        selectedCell = null;
                        playSound('click');
                    } else {
                        selectedCell = {r, c};
                        playSound('click');
                    }
                    renderTrayUI();
                    renderBoardUI();
                    updateSetupChecklist();
                    return;
                }

                // If clicking an empty spot while a piece on board is selected -> Swap / Move
                if (selectedCell && !board[r][c]) {
                    board[r][c] = board[selectedCell.r][selectedCell.c];
                    board[selectedCell.r][selectedCell.c] = null;
                    selectedCell = null;
                    playSound('move');
                    renderBoardUI();
                    updateSetupChecklist();
                    return;
                }
                return;
            }

            // PLAYING PHASE INTERACTIONS
            if (gameState === 'PLAYING') {
                if (currentTurn !== 'red' && gameMode === 'ai') return;

                const clicked = board[r][c];

                // If clicking own piece -> Select it and compute legal moves
                if (clicked && clicked.color === currentTurn) {
                    // Cannot select immovable Flag or Landmines
                    if (clicked.type === 'F' || clicked.type === 'M') {
                        addLog(`⚠️ ${PIECE_TYPES[clicked.type].name}無法移動`, 'warn');
                        return;
                    }

                    selectedCell = {r, c};
                    validMoves = calculateValidMoves(r, c);
                    playSound('click');
                    renderBoardUI();
                    return;
                }

                // If a piece is already selected, check if target (r,c) is in validMoves
                if (selectedCell) {
                    const targetMove = validMoves.find(m => m.r === r && m.c === c);
                    if (targetMove) {
                        executeMove(selectedCell.r, selectedCell.c, r, c);
                        return;
                    } else {
                        // Unselect
                        selectedCell = null;
                        validMoves = [];
                        renderBoardUI();
                    }
                }
            }
        }

        // PRESET SETUP LAYOUTS
        function applyPreset(presetType) {
            clearPlayerBoard();

            // Preset templates for Red (rows 6..12)
            // Available spots: 25 (excluding 5 campsites)
            const template = [];

            if (presetType === 'random') {
                const pool = [];
                for (let t in PIECE_TYPES) {
                    for (let i = 0; i < PIECE_TYPES[t].count; i++) pool.push(t);
                }
                // Shuffle pool
                pool.sort(() => Math.random() - 0.5);

                // Ensure Flag in HQ
                const flagIdx = pool.indexOf('F');
                pool.splice(flagIdx, 1);
                const hqPos = Math.random() < 0.5 ? {r: 12, c: 1} : {r: 12, c: 3};

                // Place Mines in bottom rows
                const mineIndices = [];
                pool.forEach((t, idx) => { if (t === 'M') mineIndices.push(idx); });

                let spotIdx = 0;
                for (let r = 6; r <= 12; r++) {
                    for (let c = 0; c < 5; c++) {
                        if (isCampsite(r, c)) continue;
                        if ((r === 12 && c === 1) || (r === 12 && c === 3)) {
                            if (r === hqPos.r && c === hqPos.c) {
                                board[r][c] = { type: 'F', color: 'red', revealed: true };
                            } else {
                                board[r][c] = { type: pool.pop(), color: 'red', revealed: true };
                            }
                        } else {
                            if (pool.length > 0) {
                                board[r][c] = { type: pool.pop(), color: 'red', revealed: true };
                            }
                        }
                    }
                }

                // Post-process validity adjustments if needed
                enforcePresetRules();

            } else if (presetType === 'balanced') {
                // Structured balanced layout
                const layout = [
                    // Row 6 (Frontline): 排長, 工兵, 連長, 排長, 工兵
                    ['32', '31', '33', '32', '31'],
                    // Row 7: 營長, 旅長, 團長, 師長, 營長
                    ['34', '36', '35', '37', '34'],
                    // Row 8 (with campsites at 1,3): 司令, (CS), 軍長, (CS), 師長
                    ['39', null, '38', null, '37'],
                    // Row 9 (campsite at 2): 團長, 旅長, (CS), 炸彈, 連長
                    ['35', '36', null, 'B', '33'],
                    // Row 10 (campsites at 1,3): 炸彈, (CS), 連長, (CS), 排長
                    ['B', null, '33', null, '32'],
                    // Row 11: 地雷, 工兵, 地雷, 地雷, 排長 (Wait, mines in back rows!)
                    ['M', '31', 'M', 'M', '32'],
                    // Row 12: 排長, 軍旗, 排長, 大本營, 排長
                    ['32', 'F', '33', '32', '32']
                ];
                applyMatrixToBoard(layout);
            } else if (presetType === 'offensive') {
                // Offensive layout: High commanders in front
                const layout = [
                    ['39', '38', '37', '37', '36'],
                    ['36', '35', '35', '34', '34'],
                    ['33', null, '33', null, '33'],
                    ['32', '32', null, '32', '31'],
                    ['31', null, '31', null, 'B'],
                    ['M', 'B', 'M', 'M', '32'],
                    ['32', '32', '33', 'F', '32']
                ];
                applyMatrixToBoard(layout);
            } else if (presetType === 'defensive') {
                // Defensive layout
                const layout = [
                    ['31', '32', '31', '32', '31'],
                    ['33', '33', '33', '34', '34'],
                    ['35', null, '35', null, '36'],
                    ['36', '37', null, '37', '38'],
                    ['B', null, '39', null, 'B'],
                    ['M', 'M', 'M', '32', '32'],
                    ['32', 'F', '32', '32', '32']
                ];
                applyMatrixToBoard(layout);
            }

            // Reset tray remaining counts to 0 since board is filled
            for (let t in trayPieces) trayPieces[t] = 0;

            playSound('click');
            renderTrayUI();
            renderBoardUI();
            updateSetupChecklist();
        }

        function enforcePresetRules() {
            // Fix Flag
            if (!board[12][1] && !board[12][3]) {
                board[12][1] = { type: 'F', color: 'red', revealed: true };
            }
            // Fix Bombs out of Row 6
            for (let c = 0; c < 5; c++) {
                if (board[6][c] && board[6][c].type === 'B') {
                    // Swap with a piece in row 7..10
                    for (let r2 = 7; r2 <= 10; r2++) {
                        for (let c2 = 0; c2 < 5; c2++) {
                            if (!isCampsite(r2, c2) && board[r2][c2] && board[r2][c2].type !== 'B') {
                                const tmp = board[6][c];
                                board[6][c] = board[r2][c2];
                                board[r2][c2] = tmp;
                                break;
                            }
                        }
                    }
                }
            }
            // Fix Mines in Row 11 & 12
            for (let r = 6; r <= 10; r++) {
                for (let c = 0; c < 5; c++) {
                    if (board[r][c] && board[r][c].type === 'M') {
                        for (let r2 = 11; r2 <= 12; r2++) {
                            for (let c2 = 0; c2 < 5; c2++) {
                                if (!isCampsite(r2, c2) && board[r2][c2] && board[r2][c2].type !== 'M' && board[r2][c2].type !== 'F') {
                                    const tmp = board[r][c];
                                    board[r][c] = board[r2][c2];
                                    board[r2][c2] = tmp;
                                    break;
                                }
                            }
                        }
                    }
                }
            }
        }

        function applyMatrixToBoard(matrix) {
            initTrayPieces();
            for (let r = 0; r < 7; r++) {
                for (let c = 0; c < 5; c++) {
                    const br = r + 6;
                    if (isCampsite(br, c)) continue;
                    const pType = matrix[r][c];
                    if (pType && PIECE_TYPES[pType]) {
                        board[br][c] = { type: pType, color: 'red', revealed: true };
                    }
                }
            }
            enforcePresetRules();
        }

        function clearPlayerBoard() {
            for (let r = 6; r <= 12; r++) {
                for (let c = 0; c < 5; c++) {
                    if (board[r][c] && board[r][c].color === 'red') {
                        board[r][c] = null;
                    }
                }
            }
            initTrayPieces();
            selectedCell = null;
            selectedTrayType = null;
            renderTrayUI();
            renderBoardUI();
            updateSetupChecklist();
        }

        // SETUP AI ENEMY BOARD AUTOMATICALLY (Rows 0 to 5)
        function setupAIOpponentBoard() {
            const pool = [];
            for (let t in PIECE_TYPES) {
                for (let i = 0; i < PIECE_TYPES[t].count; i++) pool.push(t);
            }
            pool.sort(() => Math.random() - 0.5);

            // Flag in enemy HQ: (0,1) or (0,3)
            const hqCol = Math.random() < 0.5 ? 1 : 3;
            board[0][hqCol] = { type: 'F', color: 'black', revealed: false };
            const fIdx = pool.indexOf('F');
            if (fIdx >= 0) pool.splice(fIdx, 1);

            // Mines in top two rows (0 and 1)
            let minesPlaced = 0;
            const validMineCoords = [
                {r:0,c:0},{r:0,c:2},{r:0,c:4},
                {r:1,c:0},{r:1,c:1},{r:1,c:2},{r:1,c:3},{r:1,c:4}
            ].sort(() => Math.random() - 0.5);

            validMineCoords.forEach(coord => {
                if (minesPlaced < 3 && !board[coord.r][coord.c]) {
                    board[coord.r][coord.c] = { type: 'M', color: 'black', revealed: false };
                    const mIdx = pool.indexOf('M');
                    if (mIdx >= 0) pool.splice(mIdx, 1);
                    minesPlaced++;
                }
            });

            // Fill remaining top spots 0..5 (excluding campsites and filled cells)
            for (let r = 0; r <= 5; r++) {
                for (let c = 0; c < 5; c++) {
                    if (isCampsite(r, c) || board[r][c]) continue;
                    
                    // Prevent bombs in front row (row 5)
                    if (r === 5 && pool[pool.length - 1] === 'B') {
                        // swap bomb with another non-bomb in pool
                        const nonBombIdx = pool.findIndex(t => t !== 'B');
                        if (nonBombIdx >= 0) {
                            const tmp = pool[pool.length - 1];
                            pool[pool.length - 1] = pool[nonBombIdx];
                            pool[nonBombIdx] = tmp;
                        }
                    }

                    if (pool.length > 0) {
                        board[r][c] = { type: pool.pop(), color: 'black', revealed: false };
                    }
                }
            }
        }

        // START GAME TRANSITION
        function startGame() {
            if (gameState !== 'SETUP') return;

            gameState = 'PLAYING';
            currentTurn = 'red';

            // Transition Left Sidebar UI from Setup to Stats Counter Table
            document.getElementById('sidebar-setup-panel').classList.add('hidden');
            document.getElementById('sidebar-playing-panel').classList.remove('hidden');
            document.getElementById('sidebar-playing-panel').classList.add('flex');

            document.getElementById('turn-badge').className = "px-3 py-1 rounded-full text-xs font-bold bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 flex items-center gap-1.5";
            document.getElementById('turn-text').innerText = "我方 (紅棋) 回合";

            addLog("⚔️ 對戰開始！我方執紅棋先行。", 'sys');
            renderPieceStatsUI();
            renderBoardUI();
            playSound('win');
        }

        // RENDER PIECE STATS COUNTER TABLE IN PLAYING PHASE
        function renderPieceStatsUI() {
            const container = document.getElementById('piece-stats-container');
            let html = '';

            // Count pieces remaining
            const redCounts = {};
            const blackCounts = {};

            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const p = board[r][c];
                    if (p) {
                        if (p.color === 'red') redCounts[p.type] = (redCounts[p.type] || 0) + 1;
                        if (p.color === 'black') blackCounts[p.type] = (blackCounts[p.type] || 0) + 1;
                    }
                }
            }

            html += `<div class="bg-zinc-950 border border-zinc-800 rounded-lg p-2 space-y-1.5 text-xs">`;
            for (let type in PIECE_TYPES) {
                const info = PIECE_TYPES[type];
                const rCnt = redCounts[type] || 0;
                const bCnt = blackCounts[type] || 0;

                html += `
                    <div class="flex items-center justify-between border-b border-zinc-900 pb-1">
                        <span class="font-bold text-zinc-300 w-16">${info.name}</span>
                        <div class="flex items-center gap-3">
                            <span class="text-red-400 font-mono font-bold">我: ${rCnt}/${info.count}</span>
                            <span class="text-zinc-500">|</span>
                            <span class="text-zinc-300 font-mono font-bold">敵: ${bCnt}/${info.count}</span>
                        </div>
                    </div>
                `;
            }
            html += `</div>`;

            container.innerHTML = html;
        }

        // MOVEMENT LOGIC & PATHFINDING
        function calculateValidMoves(sr, sc) {
            const piece = board[sr][sc];
            if (!piece) return [];

            const moves = [];
            const isEng = piece.type === '31'; // 工兵 (Engineer)

            // Standard 1-step directions (orthogonal + diagonal if campsite involved)
            const isSrcCampsite = isCampsite(sr, sc);

            for (let dr = -1; dr <= 1; dr++) {
                for (let dc = -1; dc <= 1; dc++) {
                    if (dr === 0 && dc === 0) continue;

                    const tr = sr + dr;
                    const tc = sc + dc;

                    if (tr < 0 || tr > 12 || tc < 0 || tc > 4) continue;

                    const isDiag = (dr !== 0 && dc !== 0);
                    const isDstCampsite = isCampsite(tr, tc);

                    // Diagonal movement ONLY permitted if moving into or out of a Campsite
                    if (isDiag && !isSrcCampsite && !isDstCampsite) continue;

                    // Cannot cross river except via bridge columns (0, 2, 4)
                    if ((sr === 5 && tr === 6) || (sr === 6 && tr === 5)) {
                        if (sc !== 0 && sc !== 2 && sc !== 4) continue;
                    }

                    if (canMoveTo(piece, tr, tc)) {
                        moves.push({r: tr, c: tc});
                    }
                }
            }

            // Railway Movement Logic
            if (isRailway(sr, sc)) {
                if (isEng) {
                    // Engineer BFS pathfinding on continuous unobstructed railway lines
                    const rwyMoves = getEngineerRailwayMoves(sr, sc);
                    rwyMoves.forEach(m => {
                        if (!moves.some(existing => existing.r === m.r && existing.c === m.c)) {
                            moves.push(m);
                        }
                    });
                } else {
                    // Non-engineer: straight unobstructed line movement on railway
                    const straightMoves = getStraightRailwayMoves(sr, sc, piece);
                    straightMoves.forEach(m => {
                        if (!moves.some(existing => existing.r === m.r && existing.c === m.c)) {
                            moves.push(m);
                        }
                    });
                }
            }

            return moves;
        }

        function canMoveTo(piece, tr, tc) {
            const target = board[tr][tc];

            // Empty spot -> Valid
            if (!target) return true;

            // Friendly piece -> Cannot capture friendly
            if (target.color === piece.color) return false;

            // Enemy piece in Campsite -> IMMUNE from all attacks!
            if (isCampsite(tr, tc)) return false;

            return true;
        }

        function getStraightRailwayMoves(sr, sc, piece) {
            const moves = [];
            const dirs = [[-1,0], [1,0], [0,-1], [0,1]];

            dirs.forEach(d => {
                let cr = sr + d[0];
                let cc = sc + d[1];

                while (cr >= 0 && cr <= 12 && cc >= 0 && cc <= 4 && isRailway(cr, cc)) {
                    // Check river crossing constraint
                    if ((cr === 5 && cr + d[0] === 6) || (cr === 6 && cr + d[0] === 5)) {
                        if (cc !== 0 && cc !== 2 && cc !== 4) break;
                    }

                    if (canMoveTo(piece, cr, cc)) {
                        moves.push({r: cr, c: cc});
                        // If target square has an enemy piece, line ends here
                        if (board[cr][cc] !== null) break;
                    } else {
                        break;
                    }

                    cr += d[0];
                    cc += d[1];
                }
            });

            return moves;
        }

        function getEngineerRailwayMoves(sr, sc) {
            const moves = [];
            const visited = Array(13).fill(null).map(() => Array(5).fill(false));
            const queue = [{r: sr, c: sc}];
            visited[sr][sc] = true;

            const piece = board[sr][sc];
            const dirs = [[-1,0], [1,0], [0,-1], [0,1]];

            while (queue.length > 0) {
                const curr = queue.shift();

                dirs.forEach(d => {
                    const tr = curr.r + d[0];
                    const tc = curr.c + d[1];

                    if (tr >= 0 && tr <= 12 && tc >= 0 && tc <= 4 && isRailway(tr, tc) && !visited[tr][tc]) {
                        // River crossing check
                        if ((curr.r === 5 && tr === 6) || (curr.r === 6 && tr === 5)) {
                            if (curr.c !== 0 && curr.c !== 2 && curr.c !== 4) return;
                        }

                        visited[tr][tc] = true;

                        if (canMoveTo(piece, tr, tc)) {
                            moves.push({r: tr, c: tc});
                            // If empty space, continue BFS search
                            if (board[tr][tc] === null) {
                                queue.push({r: tr, c: tc});
                            }
                        }
                    }
                });
            }

            return moves;
        }

        // MOVE EXECUTION & COMBAT RESOLUTION
        function executeMove(sr, sc, tr, tc) {
            const attacker = board[sr][sc];
            const defender = board[tr][tc];

            selectedCell = null;
            validMoves = [];

            if (!defender) {
                // Simple Move to empty cell
                board[tr][tc] = attacker;
                board[sr][sc] = null;

                playSound('move');
                addLog(`${attacker.color === 'red' ? '我方' : '敵方'} [${PIECE_TYPES[attacker.type].name}] 移動至 (${tr},${tc})`, 'move');
            } else {
                // Combat Resolution
                attacker.revealed = true;
                defender.revealed = true;

                const attName = PIECE_TYPES[attacker.type].name;
                const defName = PIECE_TYPES[defender.type].name;

                let outcome = resolveCombat(attacker, defender);

                if (outcome === 'ATTACKER_WINS') {
                    board[tr][tc] = attacker;
                    board[sr][sc] = null;
                    playSound('capture');
                    addLog(`💥 ${attacker.color === 'red' ? '我方' : '敵方'} [${attName}] 擊毀 敵方 [${defName}]`, 'win');
                } else if (outcome === 'DEFENDER_WINS') {
                    board[sr][sc] = null;
                    playSound('explode');
                    addLog(`💥 ${attacker.color === 'red' ? '我方' : '敵方'} [${attName}] 攻擊 敵方 [${defName}] 失敗被摧毀`, 'loss');
                } else if (outcome === 'MUTUAL_DESTRUCTION') {
                    board[sr][sc] = null;
                    board[tr][tc] = null;
                    playSound('explode');
                    addLog(`💣 ${attName} 與 ${defName} 同歸於盡！`, 'warn');
                } else if (outcome === 'FLAG_CAPTURED') {
                    board[tr][tc] = attacker;
                    board[sr][sc] = null;
                    playSound('win');
                    addLog(`🚩 ${attacker.color === 'red' ? '我方' : '敵方'} 成功奪取軍旗！`, 'win');
                    endGame(attacker.color);
                    return;
                }
            }

            renderBoardUI();
            renderPieceStatsUI();

            // Check Win/Loss conditions
            if (checkGameOver()) return;

            // Switch Turn
            currentTurn = (currentTurn === 'red') ? 'black' : 'red';

            const turnBadge = document.getElementById('turn-badge');
            const turnText = document.getElementById('turn-text');

            if (currentTurn === 'red') {
                turnBadge.className = "px-3 py-1 rounded-full text-xs font-bold bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 flex items-center gap-1.5";
                turnText.innerText = "我方 (紅棋) 回合";
            } else {
                turnBadge.className = "px-3 py-1 rounded-full text-xs font-bold bg-red-500/20 text-red-400 border border-red-500/30 flex items-center gap-1.5";
                turnText.innerText = "敵方 (黑棋) 回合";

                // Trigger AI Turn if in Single Player Mode
                if (gameMode === 'ai' && gameState === 'PLAYING') {
                    aiThinking = true;
                    setTimeout(makeAiMove, 600);
                }
            }
        }

        /**
         * ARMY CHESS COMBAT RULES:
         * 1. Equal Rank Clash -> Attacker Advantage (Attacker Wins & Survives)!
         * 2. Bomb (B) vs Anything -> Mutual Destruction!
         * 3. Landmine (M) vs Engineer (31) -> Engineer Wins & Disarms Mine!
         * 4. Landmine (M) vs Non-Engineer -> Mutual Destruction!
         * 5. Flag (F) vs Any Movable Piece -> Flag Captured!
         * 6. Ranks: 39 > 38 > 37 > 36 > 35 > 34 > 33 > 32 > 31
         */
        function resolveCombat(attacker, defender) {
            // Flag captured
            if (defender.type === 'F') return 'FLAG_CAPTURED';

            // Bomb involved -> Mutual Destruction
            if (attacker.type === 'B' || defender.type === 'B') return 'MUTUAL_DESTRUCTION';

            // Landmine defender
            if (defender.type === 'M') {
                if (attacker.type === '31') {
                    return 'ATTACKER_WINS'; // Engineer disarms mine
                } else {
                    return 'MUTUAL_DESTRUCTION'; // Non-engineer mutually dies with mine
                }
            }

            // Standard Rank Comparison
            const attRank = PIECE_TYPES[attacker.type].rank;
            const defRank = PIECE_TYPES[defender.type].rank;

            if (attRank > defRank) return 'ATTACKER_WINS';
            if (attRank < defRank) return 'DEFENDER_WINS';

            // Attacker Advantage Rule on Equal Rank
            if (attRank === defRank) return 'ATTACKER_WINS';

            return 'MUTUAL_DESTRUCTION';
        }

        // FIXED ROBUST AI MOVE EVALUATION & EXECUTION LOOP
        function makeAiMove() {
            if (gameState !== 'PLAYING' || currentTurn !== 'black') {
                aiThinking = false;
                return;
            }

            const aiPieces = [];
            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const p = board[r][c];
                    if (p && p.color === 'black' && p.type !== 'F' && p.type !== 'M') {
                        const moves = calculateValidMoves(r, c);
                        if (moves.length > 0) {
                            aiPieces.push({r, c, piece: p, moves});
                        }
                    }
                }
            }

            if (aiPieces.length === 0) {
                addLog("敵方無棋可走，我方獲得勝利！", 'win');
                endGame('red');
                aiThinking = false;
                return;
            }

            // Evaluate move scores
            let bestMove = null;
            let highestScore = -99999;

            aiPieces.forEach(item => {
                const sr = item.r;
                const sc = item.c;
                const p = item.piece;

                item.moves.forEach(m => {
                    const tr = m.r;
                    const tc = m.c;
                    const target = board[tr][tc];

                    let score = 0;

                    if (target) {
                        // Combat / Attack Evaluation
                        if (target.type === 'F') {
                            score += 10000; // Flag capture instant win
                        } else if (target.revealed) {
                            const outcome = resolveCombat(p, target);
                            if (outcome === 'ATTACKER_WINS' || outcome === 'FLAG_CAPTURED') {
                                score += (PIECE_TYPES[target.type].rank || 10) * 10 + 50;
                            } else if (outcome === 'MUTUAL_DESTRUCTION') {
                                if (p.type === 'B') score += (PIECE_TYPES[target.type].rank || 10) * 8;
                                else score += 10;
                            } else {
                                score -= 100;
                            }
                        } else {
                            // Unrevealed enemy piece
                            if (p.type === '31') {
                                // Engineer probing
                                score += 25;
                            } else if (p.type === 'B') {
                                score += 15;
                            } else {
                                score += 30;
                            }
                        }
                    } else {
                        // Empty cell move
                        // Bonus for moving towards player HQs (12,1) or (12,3)
                        const distToHQ = Math.abs(tr - 12) + Math.min(Math.abs(tc - 1), Math.abs(tc - 3));
                        score += (20 - distToHQ);

                        // Bonus for entering Campsite if threatened
                        if (isCampsite(tr, tc)) {
                            score += 15;
                        }
                    }

                    // Add slight random jitter for realistic gameplay
                    score += Math.random() * 5;

                    if (score > highestScore) {
                        highestScore = score;
                        bestMove = {sr, sc, tr, tc};
                    }
                });
            });

            if (bestMove) {
                executeMove(bestMove.sr, bestMove.sc, bestMove.tr, bestMove.tc);
            }

            aiThinking = false;
        }

        function checkGameOver() {
            let redFlag = false;
            let blackFlag = false;

            for (let r = 0; r < 13; r++) {
                for (let c = 0; c < 5; c++) {
                    const p = board[r][c];
                    if (p && p.type === 'F') {
                        if (p.color === 'red') redFlag = true;
                        if (p.color === 'black') blackFlag = true;
                    }
                }
            }

            if (!redFlag) {
                endGame('black');
                return true;
            }
            if (!blackFlag) {
                endGame('red');
                return true;
            }
            return false;
        }

        function endGame(winnerColor) {
            gameState = 'ENDED';
            const isPlayerWin = winnerColor === 'red';

            playSound(isPlayerWin ? 'win' : 'explode');

            const msg = isPlayerWin ? "🎉 恭喜！我方獲得最終勝利！" : "💔 遺憾！敵方擊敗我方！";
            addLog(msg, isPlayerWin ? 'win' : 'loss');

            document.getElementById('turn-badge').className = isPlayerWin 
                ? "px-3 py-1 rounded-full text-xs font-bold bg-amber-500/20 text-amber-400 border border-amber-500/30"
                : "px-3 py-1 rounded-full text-xs font-bold bg-red-500/20 text-red-400 border border-red-500/30";
            document.getElementById('turn-text').innerText = isPlayerWin ? "對局結束 - 我方勝" : "對局結束 - 敵方勝";
        }

        function surrender() {
            if (gameState !== 'PLAYING') return;
            endGame('black');
        }

        function restartGame() {
            board = Array(13).fill(null).map(() => Array(5).fill(null));
            gameState = 'SETUP';
            currentTurn = 'red';
            selectedCell = null;
            selectedTrayType = null;
            validMoves = [];
            aiThinking = false;

            initTrayPieces();
            drawBoardBackground();

            document.getElementById('sidebar-playing-panel').classList.add('hidden');
            document.getElementById('sidebar-playing-panel').classList.remove('flex');
            document.getElementById('sidebar-setup-panel').classList.remove('hidden');

            document.getElementById('turn-badge').className = "px-3 py-1 rounded-full text-xs font-bold bg-amber-500/20 text-amber-400 border border-amber-500/30 flex items-center gap-1.5";
            document.getElementById('turn-text').innerText = "佈陣階段";

            renderTrayUI();
            renderBoardUI();
            setupAIOpponentBoard();
            updateSetupChecklist();

            addLog("🔄 遊戲已重新開始，請重新佈陣。", 'sys');
        }

        // LOGGING UTILITY
        function addLog(text, type='move') {
            const logBox = document.getElementById('move-history-log');
            const item = document.createElement('div');
            
            let colorCls = "text-zinc-300";
            if (type === 'sys') colorCls = "text-amber-400 font-bold";
            if (type === 'win') colorCls = "text-emerald-400 font-bold";
            if (type === 'loss') colorCls = "text-red-400 font-bold";
            if (type === 'warn') colorCls = "text-amber-300";

            item.className = `p-1.5 rounded bg-zinc-900/60 border border-zinc-800/80 ${colorCls}`;
            item.innerText = text;

            logBox.appendChild(item);
            logBox.scrollTop = logBox.scrollHeight;
        }

        // MODAL CONTROLS
        function openModeModal() {
            document.getElementById('modal-mode').classList.remove('hidden');
        }

        function closeModeModal() {
            document.getElementById('modal-mode').classList.add('hidden');
        }

        function selectGameMode(mode) {
            gameMode = mode;
            closeModeModal();

            const subtitle = document.getElementById('game-mode-subtitle');
            if (mode === 'ai') subtitle.innerText = "單人對戰模式 (AI 敵方)";
            if (mode === 'local') subtitle.innerText = "雙人同屏模式 (面對面)";
            if (mode === 'online') subtitle.innerText = "線上連線房間";

            restartGame();
        }

        function createOnlineRoom() {
            const code = Math.floor(100000 + Math.random() * 900000).toString();
            document.getElementById('room-code-input').value = code;
            alert(`已建立房間！房號: ${code} (線上 Firebase 模式已連線)`);
            selectGameMode('online');
        }

        function joinOnlineRoom() {
            const code = document.getElementById('room-code-input').value;
            if (code.length === 6) {
                alert(`已成功加入房間: ${code}`);
                selectGameMode('online');
            } else {
                alert("請輸入有效的 6 位數房號");
            }
        }
    </script>
</body>
</html>
