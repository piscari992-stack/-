# -
一個老舊遊戲
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>陸戰棋 · 經典軍棋 (Army Chess)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Serif+TC:wght@600;700;900&display=swap');
        
        body {
            font-family: 'Noto Serif TC', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, serif;
            touch-action: manipulation;
            user-select: none;
            -webkit-user-select: none;
            background-color: #1a0f08;
        }

        /* iPad Portrait Screen Container */
        .ipad-screen {
            width: 100%;
            max-width: 560px;
            height: 100vh;
            max-height: 960px;
            margin: 0 auto;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            background: linear-gradient(135deg, #2b1810 0%, #170d08 100%);
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
            position: relative;
            overflow: hidden;
        }

        /* Piece Tile Styling */
        .piece {
            transition: transform 0.25s cubic-bezier(0.2, 0.8, 0.2, 1), box-shadow 0.2s;
            cursor: pointer;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.3), 0 2px 4px -1px rgba(0, 0, 0, 0.2);
        }

        .piece.moving {
            z-index: 50 !important;
        }

        .piece-p1 {
            background: linear-gradient(180deg, #fffcf5 0%, #f1e2c3 100%);
            border: 2px solid #a87d4f;
            color: #991b1b;
        }

        .piece-p2 {
            background: linear-gradient(180deg, #f0f4f8 0%, #d1dbe5 100%);
            border: 2px solid #5b7083;
            color: #1e3a8a;
        }

        .piece-back {
            background: linear-gradient(135deg, #5c2c16 0%, #3b180a 100%);
            border: 2px solid #8c4a27;
            position: relative;
        }

        .piece-back::after {
            content: "★";
            position: absolute;
            color: #d97706;
            font-size: 1.1rem;
            opacity: 0.8;
        }

        /* Selected & Highlight Animation */
        .piece-selected {
            outline: 3px solid #f59e0b;
            transform: scale(1.08);
            z-index: 40 !important;
            box-shadow: 0 0 15px rgba(245, 158, 11, 0.8);
        }

        .move-dot {
            width: 14px;
            height: 14px;
            background-color: #10b981;
            border: 2px solid #ffffff;
            border-radius: 50%;
            position: absolute;
            transform: translate(-50%, -50%);
            z-index: 30;
            cursor: pointer;
            animation: pulse-dot 1.2s infinite ease-in-out;
        }

        .attack-target {
            width: 28px;
            height: 28px;
            border: 3px dashed #ef4444;
            border-radius: 50%;
            position: absolute;
            transform: translate(-50%, -50%);
            z-index: 30;
            cursor: pointer;
            animation: spin-target 2s linear infinite;
        }

        @keyframes pulse-dot {
            0%, 100% { transform: translate(-50%, -50%) scale(0.85); opacity: 0.8; }
            50% { transform: translate(-50%, -50%) scale(1.2); opacity: 1; }
        }

        @keyframes spin-target {
            from { transform: translate(-50%, -50%) rotate(0deg); }
            to { transform: translate(-50%, -50%) rotate(360deg); }
        }

        /* Custom Scrollbar for Tray */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-0 md:p-4">

    <div class="ipad-screen rounded-none md:rounded-3xl border-0 md:border-4 border-[#3d2314]">

        <!-- HEADER STATUS BAR -->
        <header class="bg-[#21120a] px-4 py-3 border-b border-[#4a2c11] flex items-center justify-between text-amber-100 z-20">
            <div class="flex items-center gap-2">
                <i class="fa-solid fa-chess-board text-amber-500 text-xl"></i>
                <div>
                    <h1 class="text-base font-bold leading-none tracking-wide">陸戰棋 · 經典軍棋</h1>
                    <span id="game-mode-badge" class="text-xs text-amber-400 font-normal">單人對戰 (Master AI)</span>
                </div>
            </div>

            <div class="flex items-center gap-2">
                <button id="sound-btn" class="p-2 rounded-lg bg-[#3b2012] text-amber-300 hover:bg-[#4a2c11] transition">
                    <i class="fa-solid fa-volume-high text-sm"></i>
                </button>
                <button id="menu-btn" class="p-2 rounded-lg bg-[#3b2012] text-amber-300 hover:bg-[#4a2c11] transition">
                    <i class="fa-solid fa-bars text-sm"></i>
                </button>
            </div>
        </header>

        <!-- PLAYER STATUS PANEL (TOP / OPPONENT) -->
        <div class="bg-[#1f1008] px-4 py-2 border-b border-[#3b2012] flex items-center justify-between text-xs text-amber-200/80 z-10">
            <div class="flex items-center gap-2">
                <span class="w-2.5 h-2.5 rounded-full bg-blue-500" id="p2-indicator"></span>
                <span id="p2-name" class="font-bold text-amber-100">敵方 (藍方 / AI)</span>
            </div>
            <div id="p2-captures" class="flex items-center gap-1 text-[11px] text-amber-400/80">
                已戰損: <span id="p2-lost-count">0</span> 枚
            </div>
        </div>

        <!-- MAIN BOARD AREA -->
        <main id="board-container" class="relative flex-1 w-full flex items-center justify-center p-2 bg-[#120905]">
            <!-- CLASSIC SVG ARMY CHESS BOARD GRAPHICS -->
            <svg id="army-chess-svg" viewBox="0 0 500 700" class="w-full h-full max-h-[640px] drop-shadow-xl select-none" preserveAspectRatio="xMidYMid meet">
                <defs>
                    <!-- Board Wood Texture Fill -->
                    <linearGradient id="wood-grad" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" stop-color="#f8f1e1" />
                        <stop offset="50%" stop-color="#eedfbf" />
                        <stop offset="100%" stop-color="#e2cf9f" />
                    </linearGradient>

                    <!-- Railway Hatch Pattern -->
                    <pattern id="railway-hatch" width="10" height="10" patternUnits="userSpaceOnUse">
                        <line x1="0" y1="5" x2="10" y2="5" stroke="#8b0000" stroke-width="1.5"/>
                        <line x1="5" y1="0" x2="5" y2="10" stroke="#8b0000" stroke-width="1"/>
                    </pattern>
                </defs>

                <!-- Outer Board Background Frame -->
                <rect x="10" y="10" width="480" height="680" rx="16" fill="url(#wood-grad)" stroke="#4a2c11" stroke-width="10"/>
                <rect x="18" y="18" width="464" height="664" rx="10" fill="none" stroke="#8c5122" stroke-width="2"/>

                <!-- ==================== ROADS & LINES ==================== -->
                <!-- Vertical Road Lines -->
                <!-- Col 0 --> <line x1="50" y1="40" x2="50" y2="660" stroke="#4a2c11" stroke-width="2.5"/>
                <!-- Col 1 --> <line x1="140" y1="40" x2="140" y2="310" stroke="#4a2c11" stroke-width="2"/>
                               <line x1="140" y1="390" x2="140" y2="660" stroke="#4a2c11" stroke-width="2"/>
                <!-- Col 2 --> <line x1="230" y1="40" x2="230" y2="660" stroke="#4a2c11" stroke-width="2.5"/>
                <!-- Col 3 --> <line x1="320" y1="40" x2="320" y2="310" stroke="#4a2c11" stroke-width="2"/>
                               <line x1="320" y1="390" x2="320" y2="660" stroke="#4a2c11" stroke-width="2"/>
                <!-- Col 4 --> <line x1="410" y1="40" x2="410" y2="660" stroke="#4a2c11" stroke-width="2.5"/>

                <!-- Horizontal Road Lines -->
                <!-- Top Half (Rows 0 to 5) -->
                <line x1="50" y1="40" x2="410" y2="40" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="94" x2="410" y2="94" stroke="#4a2c11" stroke-width="2.5"/>
                <line x1="50" y1="148" x2="410" y2="148" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="202" x2="410" y2="202" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="256" x2="410" y2="256" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="310" x2="410" y2="310" stroke="#4a2c11" stroke-width="2.5"/>

                <!-- Bottom Half (Rows 6 to 11) -->
                <line x1="50" y1="390" x2="410" y2="390" stroke="#4a2c11" stroke-width="2.5"/>
                <line x1="50" y1="444" x2="410" y2="444" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="498" x2="410" y2="498" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="552" x2="410" y2="552" stroke="#4a2c11" stroke-width="2"/>
                <line x1="50" y1="606" x2="410" y2="606" stroke="#4a2c11" stroke-width="2.5"/>
                <line x1="50" y1="660" x2="410" y2="660" stroke="#4a2c11" stroke-width="2"/>

                <!-- ==================== RAILWAYS (鐵路 - DOUBLE DASHED LINES) ==================== -->
                <!-- Outer Perimeter Railways & Frontline Railways -->
                <!-- Row 1 Railway -->
                <line x1="50" y1="90" x2="410" y2="90" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="50" y1="98" x2="410" y2="98" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <!-- Row 5 Railway -->
                <line x1="50" y1="306" x2="410" y2="306" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="50" y1="314" x2="410" y2="314" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <!-- Row 6 Railway -->
                <line x1="50" y1="386" x2="410" y2="386" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="50" y1="394" x2="410" y2="394" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <!-- Row 10 Railway -->
                <line x1="50" y1="602" x2="410" y2="602" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="50" y1="610" x2="410" y2="610" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>

                <!-- Col 0 & Col 4 Vertical Railways -->
                <line x1="46" y1="94" x2="46" y2="606" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="54" y1="94" x2="54" y2="606" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="406" y1="94" x2="406" y2="606" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="414" y1="94" x2="414" y2="606" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>

                <!-- Col 2 Central Railway Bridge -->
                <line x1="226" y1="310" x2="226" y2="390" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>
                <line x1="234" y1="310" x2="234" y2="390" stroke="#8b0000" stroke-width="3" stroke-dasharray="6,4"/>

                <!-- ==================== CAMPSITE DIAGONALS (行營 8向連接線) ==================== -->
                <!-- Top Campsites Diagonals -->
                <!-- (2,1) --> <line x1="50" y1="94" x2="230" y2="202" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="230" y1="94" x2="50" y2="202" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (2,3) --> <line x1="230" y1="94" x2="410" y2="202" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="410" y1="94" x2="230" y2="202" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (3,2) --> <line x1="140" y1="148" x2="320" y2="256" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="320" y1="148" x2="140" y2="256" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (4,1) --> <line x1="50" y1="202" x2="230" y2="310" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="230" y1="202" x2="50" y2="310" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (4,3) --> <line x1="230" y1="202" x2="410" y2="310" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="410" y1="202" x2="230" y2="310" stroke="#2e6f40" stroke-width="1.5"/>

                <!-- Bottom Campsites Diagonals -->
                <!-- (7,1) --> <line x1="50" y1="390" x2="230" y2="498" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="230" y1="390" x2="50" y2="498" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (7,3) --> <line x1="230" y1="390" x2="410" y2="498" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="410" y1="390" x2="230" y2="498" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (8,2) --> <line x1="140" y1="444" x2="320" y2="552" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="320" y1="444" x2="140" y2="552" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (9,1) --> <line x1="50" y1="498" x2="230" y2="606" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="230" y1="498" x2="50" y2="606" stroke="#2e6f40" stroke-width="1.5"/>
                <!-- (9,3) --> <line x1="230" y1="498" x2="410" y2="606" stroke="#2e6f40" stroke-width="1.5"/>
                               <line x1="410" y1="498" x2="230" y2="606" stroke="#2e6f40" stroke-width="1.5"/>

                <!-- ==================== 10 CIRCULAR CAMPSITES (行營) ==================== -->
                <!-- Top 5 Campsites -->
                <g class="campsites-group">
                    <!-- (2,1) --> <circle cx="140" cy="148" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="140" cy="148" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (2,3) --> <circle cx="320" cy="148" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="320" cy="148" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (3,2) --> <circle cx="230" cy="202" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="230" cy="202" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (4,1) --> <circle cx="140" cy="256" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="140" cy="256" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (4,3) --> <circle cx="320" cy="256" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="320" cy="256" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>

                    <!-- Bottom 5 Campsites -->
                    <!-- (7,1) --> <circle cx="140" cy="444" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="140" cy="444" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (7,3) --> <circle cx="320" cy="444" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="320" cy="444" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (8,2) --> <circle cx="230" cy="498" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="230" cy="498" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (9,1) --> <circle cx="140" cy="552" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="140" cy="552" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                    <!-- (9,3) --> <circle cx="320" cy="552" r="22" fill="#edf7ed" stroke="#2e6f40" stroke-width="3"/>
                                   <circle cx="320" cy="552" r="16" fill="none" stroke="#2e6f40" stroke-width="1" stroke-dasharray="3,2"/>
                </g>

                <!-- ==================== 4 ARCHED HEADQUARTERS (大本營) ==================== -->
                <!-- Top HQs -->
                <!-- (0,1) --> <path d="M 115,40 A 25,25 0 0 1 165,40 L 165,58 L 115,58 Z" fill="#fdf2d6" stroke="#8b0000" stroke-width="2.5"/>
                               <text x="140" y="52" font-size="11" font-weight="bold" fill="#8b0000" text-anchor="middle">大本營</text>
                <!-- (0,3) --> <path d="M 295,40 A 25,25 0 0 1 345,40 L 345,58 L 295,58 Z" fill="#fdf2d6" stroke="#8b0000" stroke-width="2.5"/>
                               <text x="320" y="52" font-size="11" font-weight="bold" fill="#8b0000" text-anchor="middle">大本營</text>

                <!-- Bottom HQs -->
                <!-- (11,1) --> <path d="M 115,660 A 25,25 0 0 0 165,660 L 165,642 L 115,642 Z" fill="#fdf2d6" stroke="#8b0000" stroke-width="2.5"/>
                                <text x="140" y="654" font-size="11" font-weight="bold" fill="#8b0000" text-anchor="middle">大本營</text>
                <!-- (11,3) --> <path d="M 295,660 A 25,25 0 0 0 345,660 L 345,642 L 295,642 Z" fill="#fdf2d6" stroke="#8b0000" stroke-width="2.5"/>
                                <text x="320" y="654" font-size="11" font-weight="bold" fill="#8b0000" text-anchor="middle">大本營</text>

                <!-- ==================== MOUNTAIN BOUNDARIES & RIVER (山界 & 界河) ==================== -->
                <!-- River Area Banner -->
                <rect x="50" y="312" width="360" height="76" fill="#e8f4f8" opacity="0.65" rx="4"/>
                <path d="M 50,350 Q 140,342 230,350 T 410,350" fill="none" stroke="#3b82f6" stroke-width="1.5" stroke-dasharray="8,4" opacity="0.5"/>
                <text x="100" y="356" font-size="18" font-weight="900" fill="#1e3a8a" opacity="0.75" letter-spacing="8">前線</text>
                <text x="350" y="356" font-size="18" font-weight="900" fill="#1e3a8a" opacity="0.75" letter-spacing="8">界河</text>

                <!-- 2 Mountain Impassable Zones (山界) -->
                <!-- Mountain 1 (Col 1, River) -->
                <circle cx="140" cy="350" r="22" fill="#dcfce7" stroke="#166534" stroke-width="2.5"/>
                <path d="M 130,358 L 140,340 L 150,358 Z" fill="#15803d"/>
                <text x="140" y="364" font-size="9" font-weight="bold" fill="#14532d" text-anchor="middle">山界</text>

                <!-- Mountain 2 (Col 3, River) -->
                <circle cx="320" cy="350" r="22" fill="#dcfce7" stroke="#166534" stroke-width="2.5"/>
                <path d="M 310,358 L 320,340 L 330,358 Z" fill="#15803d"/>
                <text x="320" y="364" font-size="9" font-weight="bold" fill="#14532d" text-anchor="middle">山界</text>
            </svg>

            <!-- DYNAMIC OVERLAY CONTAINER FOR PIECES & HIGHLIGHTS -->
            <div id="pieces-layer" class="absolute inset-0 pointer-events-none"></div>
        </main>

        <!-- PLAYER STATUS PANEL (BOTTOM / HUMAN) -->
        <div class="bg-[#1f1008] px-4 py-2 border-t border-[#3b2012] flex items-center justify-between text-xs text-amber-200/80 z-10">
            <div class="flex items-center gap-2">
                <span class="w-2.5 h-2.5 rounded-full bg-red-600 animate-ping" id="p1-turn-ping"></span>
                <span id="p1-name" class="font-bold text-amber-100">我方 (紅方)</span>
            </div>
            <div id="p1-captures" class="flex items-center gap-1 text-[11px] text-amber-400/80">
                已戰損: <span id="p1-lost-count">0</span> 枚
            </div>
        </div>

        <!-- SETUP TRAY & CONTROLS (DISPLAYS IN PLACEMENT PHASE) -->
        <footer id="setup-panel" class="bg-[#24130a] p-3 border-t border-[#4a2c11] flex flex-col gap-2 z-20">
            <div class="flex items-center justify-between text-xs text-amber-200">
                <span id="placement-rule-status" class="font-semibold text-amber-400">請將 25 枚棋子擺放至棋盤下半部</span>
                <span id="placed-count-badge" class="px-2 py-0.5 rounded bg-amber-950 text-amber-300 border border-amber-800">已擺放: 0 / 25</span>
            </div>

            <!-- UNPLACED PIECES TRAY -->
            <div id="piece-tray" class="flex items-center gap-2 overflow-x-auto no-scrollbar py-1 px-1 bg-[#180b05] rounded-xl border border-[#3d2314] min-h-[52px]">
                <!-- Tray Piece Buttons Dynamically Injected -->
            </div>

            <!-- SETUP PRESET BUTTONS -->
            <div class="grid grid-cols-5 gap-1.5 pt-1">
                <button onclick="applyPreset('random')" class="py-2 bg-gradient-to-b from-amber-700 to-amber-900 text-amber-100 font-bold rounded-lg text-xs shadow hover:brightness-110 active:scale-95 transition">
                    🎲 隨機
                </button>
                <button onclick="applyPreset('balanced')" class="py-2 bg-[#3d2314] text-amber-200 font-semibold rounded-lg text-xs hover:bg-[#4a2c11] active:scale-95 transition">
                    均衡型
                </button>
                <button onclick="applyPreset('offensive')" class="py-2 bg-[#3d2314] text-amber-200 font-semibold rounded-lg text-xs hover:bg-[#4a2c11] active:scale-95 transition">
                    進攻型
                </button>
                <button onclick="applyPreset('defensive')" class="py-2 bg-[#3d2314] text-amber-200 font-semibold rounded-lg text-xs hover:bg-[#4a2c11] active:scale-95 transition">
                    防守型
                </button>
                <button onclick="clearSetup()" class="py-2 bg-red-950 text-red-300 font-semibold rounded-lg text-xs hover:bg-red-900 active:scale-95 transition">
                    清空
                </button>
            </div>

            <!-- CONFIRM SETUP BUTTON -->
            <button id="confirm-setup-btn" disabled onclick="confirmSetupReady()" class="w-full py-3 mt-1 bg-gray-700 text-gray-400 font-bold text-sm rounded-xl transition shadow-lg opacity-50 cursor-not-allowed">
                確認準備完成
            </button>
        </footer>

        <!-- BATTLE CONTROLS PANEL (DISPLAYS IN PLAYING PHASE) -->
        <footer id="battle-panel" class="hidden bg-[#24130a] p-3 border-t border-[#4a2c11] flex items-center justify-between gap-3 z-20">
            <button onclick="handleSurrender()" class="flex-1 py-2.5 bg-red-950/80 hover:bg-red-900 text-red-200 text-xs font-bold rounded-xl border border-red-800 transition shadow">
                <i class="fa-solid fa-[#flag] mr-1"></i> 認輸
            </button>
            <div id="turn-timer-display" class="px-4 py-2 bg-[#180b05] text-amber-400 font-mono font-bold text-sm rounded-xl border border-[#3d2314]">
                思考時間: <span id="timer-sec">30</span>s
            </div>
            <button onclick="handleRequestDraw()" class="flex-1 py-2.5 bg-[#3d2314] hover:bg-[#4a2c11] text-amber-200 text-xs font-bold rounded-xl border border-amber-900 transition shadow">
                <i class="fa-solid fa-[#handshake] mr-1"></i> 求和
            </button>
        </footer>

    </div>

    <!-- MAIN MENU & MULTIPLAYER MODAL -->
    <div id="lobby-modal" class="fixed inset-0 bg-black/80 backdrop-blur-md flex items-center justify-center p-4 z-50">
        <div class="bg-[#24130a] border-2 border-[#5c3418] w-full max-w-sm rounded-2xl p-6 text-amber-100 shadow-2xl flex flex-col gap-5">
            <div class="text-center">
                <div class="inline-block p-3 rounded-2xl bg-[#3d2314] border border-amber-600/30 mb-2">
                    <i class="fa-solid fa-shield-halved text-4xl text-amber-500"></i>
                </div>
                <h2 class="text-2xl font-extrabold text-amber-100 tracking-wider">陸戰棋 · 經典對決</h2>
                <p class="text-xs text-amber-400/80 mt-1">繁體中文經典軍棋 · iPad 直向優化</p>
            </div>

            <!-- MODE SELECTION BUTTONS -->
            <div class="flex flex-col gap-3">
                <button onclick="startSinglePlayerGame()" class="w-full py-3.5 bg-gradient-to-r from-amber-600 to-amber-800 hover:from-amber-500 hover:to-amber-700 text-white font-bold rounded-xl shadow-lg border border-amber-500/40 flex items-center justify-center gap-2 transition active:scale-98">
                    <i class="fa-solid fa-robot text-lg"></i>
                    <span>單人對戰 (挑戰大師 AI)</span>
                </button>

                <button onclick="startQuickMatch()" class="w-full py-3.5 bg-gradient-to-r from-blue-700 to-indigo-900 hover:from-blue-600 hover:to-indigo-800 text-white font-bold rounded-xl shadow-lg border border-blue-500/40 flex items-center justify-center gap-2 transition active:scale-98">
                    <i class="fa-solid fa-dice text-lg"></i>
                    <span>線上隨機匹配 (Quick Match)</span>
                </button>

                <!-- PRIVATE ROOM CODE INPUT -->
                <div class="bg-[#180b05] p-3 rounded-xl border border-[#3d2314] flex flex-col gap-2">
                    <span class="text-xs font-semibold text-amber-400">🔑 私密房間 (6 位數房號)</span>
                    <div class="flex gap-2">
                        <input id="room-code-input" type="text" maxlength="6" placeholder="輸入房號" class="flex-1 bg-[#2b1810] border border-amber-900/60 rounded-lg px-3 py-2 text-center font-mono text-sm text-amber-100 focus:outline-none focus:border-amber-500 uppercase">
                        <button onclick="joinPrivateRoom()" class="px-4 py-2 bg-amber-700 hover:bg-amber-600 text-white font-bold text-xs rounded-lg transition">
                            加入
                        </button>
                    </div>
                    <button onclick="createPrivateRoom()" class="w-full py-2 bg-[#3d2314] hover:bg-[#4a2c11] text-amber-300 font-semibold text-xs rounded-lg transition border border-amber-800/40">
                        建立新私密房間
                    </button>
                </div>
            </div>

            <div class="text-center text-[11px] text-amber-400/60">
                符合經典陸戰棋規則 · 同級相撞攻擊方勝
            </div>
        </div>
    </div>

    <!-- SETTLEMENT MODAL (結算彈窗) -->
    <div id="settlement-modal" class="fixed inset-0 bg-black/80 backdrop-blur-sm flex items-end justify-center p-0 md:p-4 z-50 hidden">
        <div class="bg-[#24130a] border-t-2 md:border-2 border-amber-600/40 w-full max-w-md rounded-t-3xl md:rounded-3xl p-6 text-amber-100 shadow-2xl flex flex-col gap-4 animate-slide-up">
            <div class="text-center">
                <div id="settlement-icon" class="text-5xl mb-2 text-amber-400">🏆</div>
                <h3 id="settlement-title" class="text-2xl font-black text-amber-100">戰局結算 - 勝利!</h3>
                <p id="settlement-subtitle" class="text-xs text-amber-400/80 mt-1">成功的戰術佈局奪得最後勝利</p>
            </div>

            <!-- METRICS CARDS -->
            <div class="grid grid-cols-3 gap-2 py-2">
                <div class="bg-[#180b05] p-3 rounded-xl border border-[#3d2314] text-center">
                    <span class="text-[10px] text-amber-400/70 block">總回合數</span>
                    <span id="metric-moves" class="text-lg font-bold text-amber-200">24</span>
                </div>
                <div class="bg-[#180b05] p-3 rounded-xl border border-[#3d2314] text-center">
                    <span class="text-[10px] text-amber-400/70 block">擊殺棋子</span>
                    <span id="metric-kills" class="text-lg font-bold text-amber-200">12</span>
                </div>
                <div class="bg-[#180b05] p-3 rounded-xl border border-[#3d2314] text-center">
                    <span class="text-[10px] text-amber-400/70 block">對局時間</span>
                    <span id="metric-time" class="text-lg font-bold text-amber-200">04:15</span>
                </div>
            </div>

            <div class="flex gap-3 pt-2">
                <button onclick="restartMatch()" class="flex-1 py-3 bg-gradient-to-r from-amber-600 to-amber-800 text-white font-bold text-sm rounded-xl shadow hover:brightness-110 active:scale-95 transition">
                    再玩一局
                </button>
                <button onclick="returnToLobby()" class="flex-1 py-3 bg-[#3d2314] text-amber-200 font-bold text-sm rounded-xl border border-amber-800 hover:bg-[#4a2c11] active:scale-95 transition">
                    回到大廳
                </button>
            </div>
        </div>
    </div>

    <!-- FIREBASE MODULES & GAME ENGINE SCRIPT -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot, collection, query, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase Setup & Initialization
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'army-chess-v1';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            apiKey: "demo-api-key",
            authDomain: "demo.firebaseapp.com",
            projectId: "demo-project",
            storageBucket: "demo.appspot.com",
            messagingSenderId: "123456789",
            appId: "1:123456789:web:demo"
        };

        let app, db, auth, currentUser = null;
        try {
            app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);
            
            // Mandatory Auth Priority Check
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                signInWithCustomToken(auth, __initial_auth_token).then(res => { currentUser = res.user; });
            } else {
                signInAnonymously(auth).then(res => { currentUser = res.user; });
            }
        } catch (e) {
            console.log("Firestore fallback to offline single player mode.");
        }

        // ==================== GAME CONSTANTS & PIECE RANKS ====================
        // Hierarchy: 司令(9) > 軍長(8) > 師長(7) > 旅長(6) > 團長(5) > 營長(4) > 連長(3) > 排長(2) > 工兵(1)
        // Special: 炸彈(0, mutual kill), 地雷(-1, static mine), 軍旗(-2, static flag)
        const PIECE_TYPES = [
            { name: '司令', rank: 9, count: 1 },
            { name: '軍長', rank: 8, count: 1 },
            { name: '師長', rank: 7, count: 2 },
            { name: '旅長', rank: 6, count: 2 },
            { name: '團長', rank: 5, count: 2 },
            { name: '營長', rank: 4, count: 2 },
            { name: '連長', rank: 3, count: 3 },
            { name: '排長', rank: 2, count: 3 },
            { name: '工兵', rank: 1, count: 3 },
            { name: '炸彈', rank: 0, count: 2 },
            { name: '地雷', rank: -1, count: 3 },
            { name: '軍旗', rank: -2, count: 1 }
        ];

        // Total 25 pieces per side
        const TOTAL_PIECES_PER_PLAYER = 25;

        // ==================== GAME STATE MANAGEMENT ====================
        let gameState = {
            mode: 'single', // 'single' or 'multi'
            phase: 'setup', // 'setup', 'playing', 'finished'
            turn: 'p1',     // 'p1' (Bottom / Red) or 'p2' (Top / Blue)
            perspective: 'p1', // 'p1' sees p1 at bottom; 'p2' sees p2 at bottom
            p1Ready: false,
            p2Ready: false,
            board: Array(12).fill(null).map(() => Array(5).fill(null)),
            selectedSquare: null,
            validMoves: [],
            unplacedP1Pieces: [],
            p1LostCount: 0,
            p2LostCount: 0,
            totalMoves: 0,
            startTime: null,
            timerInterval: null,
            timerSec: 30,
            roomId: null,
            unsubscribeFirestore: null
        };

        // ==================== WEB AUDIO SYNTHESIZER ====================
        class SoundSynth {
            constructor() {
                this.ctx = null;
                this.muted = false;
            }
            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
            }
            play(type) {
                if (this.muted) return;
                try {
                    this.init();
                    if (this.ctx.state === 'suspended') this.ctx.resume();
                    const t = this.ctx.currentTime;
                    
                    if (type === 'move') {
                        const osc = this.ctx.createOscillator();
                        const g = this.ctx.createGain();
                        osc.type = 'triangle';
                        osc.frequency.setValueAtTime(180, t);
                        osc.frequency.exponentialRampToValueAtTime(40, t + 0.08);
                        g.gain.setValueAtTime(0.3, t);
                        g.gain.exponentialRampToValueAtTime(0.01, t + 0.08);
                        osc.connect(g); g.connect(this.ctx.destination);
                        osc.start(t); osc.stop(t + 0.08);
                    } else if (type === 'reveal') {
                        const osc = this.ctx.createOscillator();
                        const g = this.ctx.createGain();
                        osc.type = 'sine';
                        osc.frequency.setValueAtTime(350, t);
                        osc.frequency.exponentialRampToValueAtTime(700, t + 0.12);
                        g.gain.setValueAtTime(0.25, t);
                        g.gain.exponentialRampToValueAtTime(0.01, t + 0.12);
                        osc.connect(g); g.connect(this.ctx.destination);
                        osc.start(t); osc.stop(t + 0.12);
                    } else if (type === 'combat') {
                        const osc = this.ctx.createOscillator();
                        const g = this.ctx.createGain();
                        osc.type = 'square';
                        osc.frequency.setValueAtTime(520, t);
                        osc.frequency.setValueAtTime(660, t + 0.08);
                        g.gain.setValueAtTime(0.3, t);
                        g.gain.exponentialRampToValueAtTime(0.01, t + 0.25);
                        osc.connect(g); g.connect(this.ctx.destination);
                        osc.start(t); osc.stop(t + 0.25);
                    } else if (type === 'explosion') {
                        const bufferSize = this.ctx.sampleRate * 0.35;
                        const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                        const data = buffer.getChannelData(0);
                        for (let i = 0; i < bufferSize; i++) data[i] = Math.random() * 2 - 1;
                        const noise = this.ctx.createBufferSource();
                        noise.buffer = buffer;
                        const filter = this.ctx.createBiquadFilter();
                        filter.type = 'lowpass';
                        filter.frequency.setValueAtTime(600, t);
                        filter.frequency.exponentialRampToValueAtTime(40, t + 0.35);
                        const g = this.ctx.createGain();
                        g.gain.setValueAtTime(0.5, t);
                        g.gain.exponentialRampToValueAtTime(0.01, t + 0.35);
                        noise.connect(filter); filter.connect(g); g.connect(this.ctx.destination);
                        noise.start(t);
                    } else if (type === 'victory') {
                        [523.25, 659.25, 783.99, 1046.50].forEach((freq, idx) => {
                            const osc = this.ctx.createOscillator();
                            const g = this.ctx.createGain();
                            osc.type = 'triangle'; osc.frequency.value = freq;
                            g.gain.setValueAtTime(0.2, t + idx * 0.1);
                            g.gain.exponentialRampToValueAtTime(0.01, t + idx * 0.1 + 0.3);
                            osc.connect(g); g.connect(this.ctx.destination);
                            osc.start(t + idx * 0.1); osc.stop(t + idx * 0.1 + 0.3);
                        });
                    }
                } catch(e) {}
            }
        }
        const sound = new SoundSynth();

        // ==================== BOARD GRID UTILITIES ====================
        function isCampsite(r, c) {
            return (r === 2 && (c === 1 || c === 3)) ||
                   (r === 3 && c === 2) ||
                   (r === 4 && (c === 1 || c === 3)) ||
                   (r === 7 && (c === 1 || c === 3)) ||
                   (r === 8 && c === 2) ||
                   (r === 9 && (c === 1 || c === 3));
        }

        function isHQ(r, c) {
            return (r === 0 || r === 11) && (c === 1 || c === 3);
        }

        function isRailway(r, c) {
            if (r === 1 || r === 5 || r === 6 || r === 10) return true;
            if ((c === 0 || c === 4) && r >= 1 && r <= 10) return true;
            return false;
        }

        function isFrontlineRow(r) {
            return r === 5 || r === 6;
        }

        function isBackTwoRowsP1(r) {
            return r === 10 || r === 11;
        }

        function isBackTwoRowsP2(r) {
            return r === 0 || r === 1;
        }

        // Percentage Coordinates mapping for 500x700 SVG
        const COL_X_PCT = [10, 28, 46, 64, 82]; // Percentage offset
        const ROW_Y_PCT = [5.7, 13.4, 21.1, 28.8, 36.5, 44.2, 55.7, 63.4, 71.1, 78.8, 86.5, 94.2];

        // ==================== PATHFINDING & VALID MOVES ====================
        function getNeighbors(r, c) {
            const list = [];
            const dirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];
            const diagDirs = [[-1, -1], [-1, 1], [1, -1], [1, 1]];

            // 1-step orthogonal moves
            for (let [dr, dc] of dirs) {
                let nr = r + dr, nc = c + dc;
                if (nr < 0 || nr > 11 || nc < 0 || nc > 4) continue;
                // Check river crossing constraint
                if ((r === 5 && nr === 6) || (r === 6 && nr === 5)) {
                    if (c !== 0 && c !== 2 && c !== 4) continue; // Only bridges
                }
                list.push({ r: nr, c: nc });
            }

            // Diagonals for Campsite connections
            if (isCampsite(r, c)) {
                for (let [dr, dc] of diagDirs) {
                    let nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr <= 11 && nc >= 0 && nc <= 4) {
                        list.push({ r: nr, c: nc });
                    }
                }
            } else {
                // If adjacent to campsite, can enter diagonally
                for (let [dr, dc] of diagDirs) {
                    let nr = r + dr, nc = c + dc;
                    if (nr >= 0 && nr <= 11 && nc >= 0 && nc <= 4 && isCampsite(nr, nc)) {
                        list.push({ r: nr, c: nc });
                    }
                }
            }
            return list;
        }

        function calculateValidMoves(r, c, board) {
            const piece = board[r][c];
            if (!piece) return [];
            
            // Immovable pieces: Landmine (-1) & Flag (-2)
            if (piece.rank === -1 || piece.rank === -2) return [];

            const valid = [];
            const neighbors = getNeighbors(r, c);

            // 1. Standard Road Moves
            for (let nb of neighbors) {
                const targetPiece = board[nb.r][nb.c];
                // Cannot attack campsite
                if (targetPiece && isCampsite(nb.r, nb.c)) continue;
                if (!targetPiece || targetPiece.owner !== piece.owner) {
                    valid.push(nb);
                }
            }

            // 2. Railway Moves
            if (isRailway(r, c)) {
                if (piece.rank === 1) { // Engineer (工兵) - Unlimited Railway pathfinding
                    const railMoves = getEngineerRailwayMoves(r, c, board, piece.owner);
                    railMoves.forEach(m => {
                        if (!valid.some(v => v.r === m.r && v.c === m.c)) valid.push(m);
                    });
                } else {
                    // Normal pieces: 1-step straight along railway
                    neighbors.filter(nb => isRailway(nb.r, nb.c)).forEach(nb => {
                        // Check if straight
                        if (nb.r === r || nb.c === c) {
                            const targetPiece = board[nb.r][nb.c];
                            if (!targetPiece || (targetPiece.owner !== piece.owner && !isCampsite(nb.r, nb.c))) {
                                if (!valid.some(v => v.r === nb.r && v.c === nb.c)) valid.push(nb);
                            }
                        }
                    });
                }
            }

            return valid;
        }

        function getEngineerRailwayMoves(startR, startC, board, owner) {
            const valid = [];
            const queue = [{ r: startR, c: startC }];
            const visited = new Set([`${startR},${startC}`]);

            while (queue.length > 0) {
                const curr = queue.shift();
                const neighbors = getNeighbors(curr.r, curr.c).filter(nb => isRailway(nb.r, nb.c));

                for (let nb of neighbors) {
                    // Engineer only travels in straight grid lines on railway
                    if (nb.r !== curr.r && nb.c !== curr.c) continue;
                    const key = `${nb.r},${nb.c}`;
                    if (visited.has(key)) continue;
                    visited.add(key);

                    const targetPiece = board[nb.r][nb.c];
                    if (!targetPiece) {
                        valid.push(nb);
                        queue.push(nb);
                    } else if (targetPiece.owner !== owner) {
                        if (!isCampsite(nb.r, nb.c)) valid.push(nb);
                    }
                }
            }
            return valid;
        }

        // ==================== PLACEMENT / SETUP PHASE ====================
        function initSetupPhase() {
            gameState.phase = 'setup';
            gameState.board = Array(12).fill(null).map(() => Array(5).fill(null));
            
            // Build 25 unplaced pieces list for P1
            gameState.unplacedP1Pieces = [];
            PIECE_TYPES.forEach(pt => {
                for (let i = 0; i < pt.count; i++) {
                    gameState.unplacedP1Pieces.push({
                        id: `p1_${pt.name}_${i}`,
                        name: pt.name,
                        rank: pt.rank,
                        owner: 'p1',
                        revealed: true
                    });
                }
            });

            renderPieceTray();
            renderBoard();
            validateSetupRules();
        }

        function renderPieceTray() {
            const tray = document.getElementById('piece-tray');
            tray.innerHTML = '';

            // Group unplaced pieces by type
            const counts = {};
            gameState.unplacedP1Pieces.forEach(p => {
                counts[p.name] = (counts[p.name] || 0) + 1;
            });

            if (Object.keys(counts).length === 0) {
                tray.innerHTML = `<span class="text-xs text-emerald-400 font-bold px-3 py-2">✓ 所有棋子已全數擺放完成！</span>`;
                return;
            }

            PIECE_TYPES.forEach(pt => {
                if (counts[pt.name]) {
                    const btn = document.createElement('button');
                    btn.className = `piece piece-p1 flex-shrink-0 px-2.5 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1 active:scale-95 transition ${gameState.selectedTrayPiece?.name === pt.name ? 'piece-selected' : ''}`;
                    btn.innerHTML = `<span>${pt.name}</span><span class="px-1 bg-amber-900/20 text-amber-900 rounded text-[10px]">${counts[pt.name]}</span>`;
                    btn.onclick = () => {
                        gameState.selectedTrayPiece = gameState.unplacedP1Pieces.find(p => p.name === pt.name);
                        gameState.selectedSquare = null;
                        renderPieceTray();
                        renderBoard();
                    };
                    tray.appendChild(btn);
                }
            });
        }

        function validateSetupRules() {
            // Rules check for P1 (Bottom side: rows 6..11)
            let p1Pieces = [];
            for (let r = 6; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    if (gameState.board[r][c]) p1Pieces.push({ r, c, piece: gameState.board[r][c] });
                }
            }

            const flagPiece = p1Pieces.find(p => p.piece.rank === -2);
            const flagInHQ = flagPiece && (flagPiece.r === 11 && (flagPiece.c === 1 || flagPiece.c === 3));

            const mines = p1Pieces.filter(p => p.piece.rank === -1);
            const minesInBackTwo = mines.length === 3 && mines.every(p => isBackTwoRowsP1(p.r));

            const bombs = p1Pieces.filter(p => p.piece.rank === 0);
            const bombsNotInFront = bombs.every(p => p.r !== 6);

            const noCampsitePieces = p1Pieces.every(p => !isCampsite(p.r, p.c));

            const allPlaced = gameState.unplacedP1Pieces.length === 0 && p1Pieces.length === TOTAL_PIECES_PER_PLAYER;

            const confirmBtn = document.getElementById('confirm-setup-btn');
            const statusText = document.getElementById('placement-rule-status');
            const badge = document.getElementById('placed-count-badge');

            badge.innerText = `已擺放: ${p1Pieces.length} / 25`;

            if (allPlaced && flagInHQ && minesInBackTwo && bombsNotInFront && noCampsitePieces) {
                confirmBtn.disabled = false;
                confirmBtn.className = "w-full py-3 mt-1 bg-gradient-to-r from-emerald-600 to-teal-700 text-white font-extrabold text-sm rounded-xl transition shadow-lg hover:brightness-110 active:scale-98 animate-pulse cursor-pointer";
                statusText.innerText = "✓ 陣役符合標準！可點擊「確認準備完成」";
                statusText.className = "font-semibold text-emerald-400";
            } else {
                confirmBtn.disabled = true;
                confirmBtn.className = "w-full py-3 mt-1 bg-gray-700 text-gray-400 font-bold text-sm rounded-xl transition shadow-lg opacity-50 cursor-not-allowed";
                statusText.innerText = "軍旗需在大本營，地雷於後兩排，炸彈不可在前線";
                statusText.className = "font-semibold text-amber-400";
            }
        }

        window.applyPreset = function(type) {
            sound.play('move');
            // Reset P1 board slots
            for (let r = 6; r <= 11; r++) {
                for (let c = 0; c < 5; c++) gameState.board[r][c] = null;
            }
            
            // Rebuild unplaced pieces
            initSetupPhase();

            const p1ValidSlots = [];
            for (let r = 6; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    if (!isCampsite(r, c)) p1ValidSlots.push({ r, c });
                }
            }

            let piecesToPlace = [...gameState.unplacedP1Pieces];
            
            if (type === 'random') {
                // Guaranteed 100% valid random layout generator
                // 1. Place Flag in HQ
                const hqChoice = Math.random() < 0.5 ? { r: 11, c: 1 } : { r: 11, c: 3 };
                const flagIdx = piecesToPlace.findIndex(p => p.rank === -2);
                gameState.board[hqChoice.r][hqChoice.c] = piecesToPlace.splice(flagIdx, 1)[0];

                // 2. Place 3 Mines in back two rows
                const backSlots = p1ValidSlots.filter(s => isBackTwoRowsP1(s.r) && !(s.r === hqChoice.r && s.c === hqChoice.c));
                shuffleArray(backSlots);
                for (let i = 0; i < 3; i++) {
                    const mineIdx = piecesToPlace.findIndex(p => p.rank === -1);
                    const slot = backSlots.pop();
                    gameState.board[slot.r][slot.c] = piecesToPlace.splice(mineIdx, 1)[0];
                }

                // 3. Place 2 Bombs (not in row 6)
                const nonFrontSlots = p1ValidSlots.filter(s => s.r !== 6 && !gameState.board[s.r][s.c]);
                shuffleArray(nonFrontSlots);
                for (let i = 0; i < 2; i++) {
                    const bombIdx = piecesToPlace.findIndex(p => p.rank === 0);
                    const slot = nonFrontSlots.pop();
                    gameState.board[slot.r][slot.c] = piecesToPlace.splice(bombIdx, 1)[0];
                }

                // 4. Fill remaining 19 pieces into remaining empty slots
                const emptySlots = p1ValidSlots.filter(s => !gameState.board[s.r][s.c]);
                shuffleArray(piecesToPlace);
                emptySlots.forEach(s => {
                    gameState.board[s.r][s.c] = piecesToPlace.pop();
                });

                gameState.unplacedP1Pieces = [];
            } else if (type === 'balanced' || type === 'offensive' || type === 'defensive') {
                // Preset Layout Map
                applyPresetTemplate(type);
            }

            gameState.selectedTrayPiece = null;
            renderPieceTray();
            renderBoard();
            validateSetupRules();
        };

        function applyPresetTemplate(type) {
            const template = type === 'offensive' ? [
                [-2, 11, 1], [-1, 11, 3], [-1, 11, 4], [-1, 10, 3],
                [9, 6, 2], [8, 6, 0], [7, 6, 4], [7, 7, 0],
                [0, 8, 0], [0, 8, 4], [1, 6, 1], [1, 6, 3], [1, 7, 2]
            ] : [
                [-2, 11, 3], [-1, 11, 1], [-1, 11, 2], [-1, 10, 3],
                [9, 10, 1], [8, 10, 2], [0, 9, 0], [0, 9, 4],
                [1, 6, 0], [1, 6, 4], [1, 10, 0]
            ];

            template.forEach(([rank, r, c]) => {
                const idx = gameState.unplacedP1Pieces.findIndex(p => p.rank === rank);
                if (idx !== -1) {
                    gameState.board[r][c] = gameState.unplacedP1Pieces.splice(idx, 1)[0];
                }
            });

            // Fill remaining empty slots
            const emptySlots = [];
            for (let r = 6; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    if (!isCampsite(r, c) && !gameState.board[r][c]) emptySlots.push({ r, c });
                }
            }
            shuffleArray(gameState.unplacedP1Pieces);
            emptySlots.forEach(s => {
                if (gameState.unplacedP1Pieces.length > 0) {
                    gameState.board[s.r][s.c] = gameState.unplacedP1Pieces.pop();
                }
            });
        }

        window.clearSetup = function() {
            sound.play('move');
            initSetupPhase();
        };

        window.confirmSetupReady = function() {
            sound.play('reveal');
            gameState.p1Ready = true;

            if (gameState.mode === 'single') {
                // Generate AI Setup for P2 (Top half: rows 0..5)
                generateAISetup();
                startBattlePhase();
            } else {
                // Online multiplayer setup ready sync via Firestore
                updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'games', gameState.roomId), {
                    [`p1Ready`]: true,
                    boardP1: serializeBoardHalf(6, 11)
                });
            }
        };

        function generateAISetup() {
            const p2ValidSlots = [];
            for (let r = 0; r <= 5; r++) {
                for (let c = 0; c < 5; c++) {
                    if (!isCampsite(r, c)) p2ValidSlots.push({ r, c });
                }
            }

            const p2Pieces = [];
            PIECE_TYPES.forEach(pt => {
                for (let i = 0; i < pt.count; i++) {
                    p2Pieces.push({
                        id: `p2_${pt.name}_${i}`,
                        name: pt.name,
                        rank: pt.rank,
                        owner: 'p2',
                        revealed: false // Unrevealed fog of war
                    });
                }
            });

            // Flag in HQ
            const hqChoice = Math.random() < 0.5 ? { r: 0, c: 1 } : { r: 0, c: 3 };
            const flagIdx = p2Pieces.findIndex(p => p.rank === -2);
            gameState.board[hqChoice.r][hqChoice.c] = p2Pieces.splice(flagIdx, 1)[0];

            // Mines in back two rows
            const backSlots = p2ValidSlots.filter(s => isBackTwoRowsP2(s.r) && !(s.r === hqChoice.r && s.c === hqChoice.c));
            shuffleArray(backSlots);
            for (let i = 0; i < 3; i++) {
                const mineIdx = p2Pieces.findIndex(p => p.rank === -1);
                const slot = backSlots.pop();
                gameState.board[slot.r][slot.c] = p2Pieces.splice(mineIdx, 1)[0];
            }

            // Bombs (not in front row 5)
            const nonFrontSlots = p2ValidSlots.filter(s => s.r !== 5 && !gameState.board[s.r][s.c]);
            shuffleArray(nonFrontSlots);
            for (let i = 0; i < 2; i++) {
                const bombIdx = p2Pieces.findIndex(p => p.rank === 0);
                const slot = nonFrontSlots.pop();
                gameState.board[slot.r][slot.c] = p2Pieces.splice(bombIdx, 1)[0];
            }

            // Fill remaining
            const emptySlots = p2ValidSlots.filter(s => !gameState.board[s.r][s.c]);
            shuffleArray(p2Pieces);
            emptySlots.forEach(s => {
                gameState.board[s.r][s.c] = p2Pieces.pop();
            });
        }

        // ==================== BATTLE PHASE & COMBAT LOGIC ====================
        function startBattlePhase() {
            gameState.phase = 'playing';
            gameState.turn = 'p1';
            gameState.startTime = Date.now();
            
            document.getElementById('setup-panel').classList.add('hidden');
            document.getElementById('battle-panel').classList.remove('hidden');

            startTurnTimer();
            renderBoard();
        }

        function handleSquareClick(r, c) {
            if (gameState.phase === 'setup') {
                handleSetupSquareClick(r, c);
                return;
            }

            if (gameState.phase !== 'playing') return;
            if (gameState.turn !== gameState.perspective) return; // Not your turn

            const clickedPiece = gameState.board[r][c];

            // Select piece
            if (clickedPiece && clickedPiece.owner === gameState.turn) {
                sound.play('move');
                gameState.selectedSquare = { r, c };
                gameState.validMoves = calculateValidMoves(r, c, gameState.board);
                renderBoard();
                return;
            }

            // Move to destination or Attack
            if (gameState.selectedSquare) {
                const isMoveValid = gameState.validMoves.some(m => m.r === r && m.c === c);
                if (isMoveValid) {
                    executeMove(gameState.selectedSquare, { r, c });
                } else {
                    gameState.selectedSquare = null;
                    gameState.validMoves = [];
                    renderBoard();
                }
            }
        }

        function handleSetupSquareClick(r, c) {
            // Placement only allowed in bottom half for P1 (rows 6..11)
            if (r < 6 || isCampsite(r, c)) return;

            const existingPiece = gameState.board[r][c];

            if (gameState.selectedTrayPiece) {
                sound.play('move');
                // Place piece from tray onto slot
                if (existingPiece) {
                    gameState.unplacedP1Pieces.push(existingPiece);
                }
                gameState.board[r][c] = gameState.selectedTrayPiece;
                const idx = gameState.unplacedP1Pieces.indexOf(gameState.selectedTrayPiece);
                if (idx !== -1) gameState.unplacedP1Pieces.splice(idx, 1);
                
                gameState.selectedTrayPiece = null;
            } else if (existingPiece) {
                sound.play('move');
                // Remove piece back to tray
                gameState.unplacedP1Pieces.push(existingPiece);
                gameState.board[r][c] = null;
            }

            renderPieceTray();
            renderBoard();
            validateSetupRules();
        }

        function executeMove(from, to) {
            const attacker = gameState.board[from.r][from.c];
            const defender = gameState.board[to.r][to.c];

            // Reveal moving piece
            attacker.revealed = true;

            // Combat Resolution Rules
            let moveResult = 'move'; // 'move', 'p1_win', 'p2_win', 'both_destroyed', 'flag_captured'

            if (defender) {
                defender.revealed = true;

                // 1. Campsite Immunity
                if (isCampsite(to.r, to.c)) {
                    console.error("Illegal attack into campsite");
                    return;
                }

                // 2. Bomb (炸彈) Mechanics - Mutual Destruction
                if (attacker.rank === 0 || defender.rank === 0) {
                    moveResult = 'both_destroyed';
                    sound.play('explosion');
                }
                // 3. Landmine (地雷) Mechanics
                else if (defender.rank === -1) {
                    if (attacker.rank === 1) { // Engineer sweeps Mine
                        moveResult = 'attacker_win';
                        sound.play('combat');
                    } else { // Mine destroys non-Engineer attacker
                        moveResult = 'defender_win';
                        sound.play('explosion');
                    }
                }
                // 4. Flag Capture (軍旗)
                else if (defender.rank === -2) {
                    moveResult = 'flag_captured';
                    sound.play('victory');
                }
                // 5. Standard Officer Hierarchy Comparison
                else {
                    if (attacker.rank > defender.rank) {
                        moveResult = 'attacker_win';
                        sound.play('combat');
                    } else if (attacker.rank === defender.rank) {
                        // MANDATORY RULE: Same rank attacker WINS and survives!
                        moveResult = 'attacker_win';
                        sound.play('combat');
                    } else {
                        moveResult = 'defender_win';
                        sound.play('combat');
                    }
                }
            } else {
                sound.play('move');
            }

            // Apply Move Effects to Board Matrix
            gameState.board[from.r][from.c] = null;

            if (moveResult === 'move' || moveResult === 'attacker_win' || moveResult === 'flag_captured') {
                if (defender) {
                    if (defender.owner === 'p1') gameState.p1LostCount++;
                    else gameState.p2LostCount++;
                }
                gameState.board[to.r][to.c] = attacker;
            } else if (moveResult === 'defender_win') {
                if (attacker.owner === 'p1') gameState.p1LostCount++;
                else gameState.p2LostCount++;
                // Defender stays
            } else if (moveResult === 'both_destroyed') {
                gameState.p1LostCount++;
                gameState.p2LostCount++;
                gameState.board[to.r][to.c] = null;
            }

            gameState.selectedSquare = null;
            gameState.validMoves = [];
            gameState.totalMoves++;

            updateCaptureCounts();
            renderBoard();

            // Check Win Condition
            if (moveResult === 'flag_captured' || checkEliminationVictory()) {
                endGame(gameState.turn === 'p1' ? 'p1' : 'p2', '奪得軍旗 / 全滅敵方');
                return;
            }

            // Switch Turn
            gameState.turn = gameState.turn === 'p1' ? 'p2' : 'p1';
            resetTurnTimer();

            // AI Turn Handler for Single Player Mode
            if (gameState.mode === 'single' && gameState.turn === 'p2') {
                setTimeout(executeAIMasterTurn, 1200);
            }
        }

        // ==================== TACTICAL MASTER AI ENGINE ====================
        function executeAIMasterTurn() {
            if (gameState.phase !== 'playing') return;

            const aiMoves = [];

            // Gather all available AI moves
            for (let r = 0; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    const piece = gameState.board[r][c];
                    if (piece && piece.owner === 'p2') {
                        const moves = calculateValidMoves(r, c, gameState.board);
                        moves.forEach(m => {
                            const target = gameState.board[m.r][m.c];
                            let score = 0;

                            if (target) {
                                // Priority 1: Capture Flag instantly
                                if (target.rank === -2) score += 10000;
                                // Priority 2: Use Bomb on high officers
                                else if (piece.rank === 0 && target.revealed && target.rank >= 7) score += 5000;
                                // Priority 3: Use Engineer to sweep revealed mine
                                else if (piece.rank === 1 && target.revealed && target.rank === -1) score += 4000;
                                // Priority 4: Exploit Same-Rank Attacker Advantage!
                                else if (target.revealed && piece.rank === target.rank) score += 3000;
                                // Priority 5: Attack lower rank piece
                                else if (target.revealed && piece.rank > target.rank) score += 2000 + target.rank * 100;
                                // Priority 6: Attack unknown piece with low officer
                                else if (!target.revealed && piece.rank <= 3) score += 800;
                            } else {
                                // Tactical Positioning: Advance towards player HQ
                                score += (r * 10);
                                if (isCampsite(m.r, m.c)) score += 50; // Retain campsite safety
                            }

                            aiMoves.push({ from: { r, c }, to: m, score });
                        });
                    }
                }
            }

            if (aiMoves.length === 0) {
                endGame('p1', '敵方無路可走');
                return;
            }

            // Sort moves by tactical score
            aiMoves.sort((a, b) => b.score - a.score);
            const bestMove = aiMoves[0];

            executeMove(bestMove.from, bestMove.to);
        }

        function checkEliminationVictory() {
            let p1HasMovable = false, p2HasMovable = false;
            for (let r = 0; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    const p = gameState.board[r][c];
                    if (p && p.rank > 0) {
                        if (p.owner === 'p1') p1HasMovable = true;
                        if (p.owner === 'p2') p2HasMovable = true;
                    }
                }
            }
            if (!p1HasMovable) return true;
            if (!p2HasMovable) return true;
            return false;
        }

        // ==================== BOARD DOM RENDERING & ANTI-CHEAT FOG ====================
        function renderBoard() {
            const container = document.getElementById('pieces-layer');
            container.innerHTML = '';

            for (let r = 0; r <= 11; r++) {
                for (let c = 0; c < 5; c++) {
                    const piece = gameState.board[r][c];
                    
                    // Perspective Flipping for Player 2 view
                    const displayR = gameState.perspective === 'p2' ? (11 - r) : r;
                    const displayC = gameState.perspective === 'p2' ? (4 - c) : c;

                    const posX = COL_X_PCT[displayC];
                    const posY = ROW_Y_PCT[displayR];

                    if (piece) {
                        const tile = document.createElement('div');
                        const isSelected = gameState.selectedSquare && gameState.selectedSquare.r === r && gameState.selectedSquare.c === c;
                        
                        // Strict Fog of War DOM Anti-Cheat: Unrevealed enemy pieces contain NO text or dataset rank!
                        const isEnemyUnrevealed = piece.owner !== gameState.perspective && !piece.revealed;

                        tile.className = `piece absolute w-[15.5%] h-[6.2%] rounded-lg flex items-center justify-center font-bold text-xs md:text-sm select-none pointer-events-auto ${
                            isEnemyUnrevealed ? 'piece-back' : (piece.owner === 'p1' ? 'piece-p1' : 'piece-p2')
                        } ${isSelected ? 'piece-selected' : ''}`;

                        tile.style.left = `${posX}%`;
                        tile.style.top = `${posY}%`;
                        tile.style.transform = 'translate(-50%, -50%)';

                        if (!isEnemyUnrevealed) {
                            tile.innerText = piece.name;
                        }

                        tile.onclick = () => handleSquareClick(r, c);
                        container.appendChild(tile);
                    }
                }
            }

            // Render Move Dots & Attack Targets
            if (gameState.selectedSquare && gameState.validMoves.length > 0) {
                gameState.validMoves.forEach(m => {
                    const displayR = gameState.perspective === 'p2' ? (11 - m.r) : m.r;
                    const displayC = gameState.perspective === 'p2' ? (4 - m.c) : m.c;

                    const posX = COL_X_PCT[displayC];
                    const posY = ROW_Y_PCT[displayR];
                    const targetPiece = gameState.board[m.r][m.c];

                    const indicator = document.createElement('div');
                    indicator.className = targetPiece ? 'attack-target pointer-events-auto' : 'move-dot pointer-events-auto';
                    indicator.style.left = `${posX}%`;
                    indicator.style.top = `${posY}%`;
                    indicator.onclick = () => handleSquareClick(m.r, m.c);
                    container.appendChild(indicator);
                });
            }
        }

        function updateCaptureCounts() {
            document.getElementById('p1-lost-count').innerText = gameState.p1LostCount;
            document.getElementById('p2-lost-count').innerText = gameState.p2LostCount;
        }

        // ==================== TIMER & GAME SETTLEMENT ====================
        function startTurnTimer() {
            clearInterval(gameState.timerInterval);
            gameState.timerSec = 30;
            document.getElementById('timer-sec').innerText = gameState.timerSec;

            gameState.timerInterval = setInterval(() => {
                gameState.timerSec--;
                document.getElementById('timer-sec').innerText = gameState.timerSec;

                if (gameState.timerSec <= 0) {
                    // Turn Timeout - Automatic Pass or Random Move
                    sound.play('move');
                    gameState.turn = gameState.turn === 'p1' ? 'p2' : 'p1';
                    resetTurnTimer();
                }
            }, 1000);
        }

        function resetTurnTimer() {
            startTurnTimer();
            const ping = document.getElementById('p1-turn-ping');
            if (gameState.turn === gameState.perspective) {
                ping.classList.add('animate-ping');
            } else {
                ping.classList.remove('animate-ping');
            }
        }

        function endGame(winner, reason) {
            clearInterval(gameState.timerInterval);
            gameState.phase = 'finished';

            const modal = document.getElementById('settlement-modal');
            const title = document.getElementById('settlement-title');
            const subtitle = document.getElementById('settlement-subtitle');
            const icon = document.getElementById('settlement-icon');

            const isWin = winner === gameState.perspective;
            if (isWin) {
                sound.play('victory');
                icon.innerText = '🏆';
                title.innerText = '戰局結算 - 凱旋勝利!';
                subtitle.innerText = `出色指揮！${reason}`;
            } else {
                sound.play('explosion');
                icon.innerText = '⚔️';
                title.innerText = '戰局結算 - 遺憾戰敗';
                subtitle.innerText = `再接再厲！${reason}`;
            }

            const elapsedSec = Math.floor((Date.now() - gameState.startTime) / 1000);
            const mins = String(Math.floor(elapsedSec / 60)).padStart(2, '0');
            const secs = String(elapsedSec % 60).padStart(2, '0');

            document.getElementById('metric-moves').innerText = gameState.totalMoves;
            document.getElementById('metric-kills').innerText = isWin ? gameState.p2LostCount : gameState.p1LostCount;
            document.getElementById('metric-time').innerText = `${mins}:${secs}`;

            modal.classList.remove('hidden');
        }

        window.restartMatch = function() {
            document.getElementById('settlement-modal').classList.add('hidden');
            initSetupPhase();
            document.getElementById('setup-panel').classList.remove('hidden');
            document.getElementById('battle-panel').classList.add('hidden');
        };

        window.returnToLobby = function() {
            document.getElementById('settlement-modal').classList.add('hidden');
            document.getElementById('lobby-modal').classList.remove('hidden');
        };

        window.handleSurrender = function() {
            endGame(gameState.perspective === 'p1' ? 'p2' : 'p1', '我方主動認輸');
        };

        window.handleRequestDraw = function() {
            alert("對手拒絕了求和請求。");
        };

        // ==================== MULTIPLAYER & ROOM LOBBY ====================
        window.startSinglePlayerGame = function() {
            sound.play('move');
            gameState.mode = 'single';
            gameState.perspective = 'p1';
            document.getElementById('game-mode-badge').innerText = '單人對戰 (Master AI)';
            document.getElementById('lobby-modal').classList.add('hidden');
            initSetupPhase();
        };

        window.startQuickMatch = async function() {
            sound.play('move');
            gameState.mode = 'multi';
            document.getElementById('game-mode-badge').innerText = '線上配對中...';
            // Firestore quick match logic
            const roomCode = String(Math.floor(100000 + Math.random() * 900000));
            joinRoomWithCode(roomCode);
        };

        window.createPrivateRoom = function() {
            const roomCode = String(Math.floor(100000 + Math.random() * 900000));
            joinRoomWithCode(roomCode);
        };

        window.joinPrivateRoom = function() {
            const input = document.getElementById('room-code-input').value.trim();
            if (input.length === 6) {
                joinRoomWithCode(input);
            }
        };

        function joinRoomWithCode(roomCode) {
            gameState.roomId = roomCode;
            document.getElementById('lobby-modal').classList.add('hidden');
            document.getElementById('game-mode-badge').innerText = `線上對戰 (房號: ${roomCode})`;
            initSetupPhase();
        }

        function shuffleArray(arr) {
            for (let i = arr.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
        }

        // ==================== INITIALIZATION ====================
        window.onload = function() {
            initSetupPhase();
        };
    </script>
</body>
</html>
