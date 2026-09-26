# number-touch-app
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta property="og:title" content="ナンバータッチ & TMT トレーニング">
    <meta property="og:description" content="数字や文字を順番にタップして動体視力と脳のスピードを鍛えるトレーニングアプリ">
    <meta property="og:type" content="website">
    <title>ナンバータッチ & TMT トレーニング</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#4F46E5',
                        secondary: '#10B981',
                        accent: '#F59E0B',
                        dark: '#1F2937'
                    },
                    keyframes: {
                        shake: {
                            '0%, 100%': { transform: 'translateX(0)' },
                            '20%, 60%': { transform: 'translateX(-6px)' },
                            '40%, 80%': { transform: 'translateX(6px)' },
                        },
                        pop: {
                            '0%': { transform: 'scale(1)' },
                            '50%': { transform: 'scale(1.2)' },
                            '100%': { transform: 'scale(0)' }
                        }
                    },
                    animation: {
                        shake: 'shake 0.3s ease-in-out',
                        pop: 'pop 0.25s ease-out forwards'
                    }
                }
            }
        }
    </script>
    <style>
        /* タッチデバイスでの連打拡大、スクロール、長押し選択などを完全に防止 */
        html, body {
            touch-action: none;
            overscroll-behavior: none;
            user-select: none;
            -webkit-user-select: none;
            -webkit-touch-callout: none;
            overflow: hidden;
            width: 100%;
            height: 100%;
            position: fixed;
        }
        .bubble-btn {
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2), inset 0 2px 4px rgba(255, 255, 255, 0.6);
            transition: transform 0.1s ease, background-color 0.15s ease, opacity 0.2s ease;
            touch-action: none;
        }
        .bubble-btn:active {
            transform: scale(0.92);
        }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex flex-col justify-between font-sans overflow-hidden select-none">

    <!-- Header / Navbar -->
    <header id="mainHeader" class="bg-slate-800/80 backdrop-blur border-b border-slate-700 px-4 py-3 sticky top-0 z-20 transition-all duration-300">
        <div class="max-w-4xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-2">
                <div class="bg-indigo-600 text-white p-2 rounded-xl shadow-lg">
                    <i class="fa-solid fa-bolt text-lg"></i>
                </div>
                <div>
                    <h1 class="font-bold text-lg md:text-xl text-white leading-tight">Touch Speed</h1>
                    <p class="text-xs text-slate-400">ナンバータッチ & TMT</p>
                </div>
            </div>
            
            <div class="flex items-center gap-3">
                <button id="soundToggleBtn" class="p-2.5 rounded-full bg-slate-700 hover:bg-slate-600 text-amber-400 transition" title="効果音切り替え">
                    <i class="fa-solid fa-volume-high text-lg"></i>
                </button>
                <button id="statsOpenBtn" class="p-2.5 rounded-full bg-slate-700 hover:bg-slate-600 text-indigo-400 transition" title="ベスト記録">
                    <i class="fa-solid fa-trophy text-lg"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="max-w-4xl w-full mx-auto p-3 md:p-4 flex-1 flex flex-col gap-3 overflow-hidden">
        
        <!-- Controls & Mode Selector -->
        <section id="controlsSection" class="bg-slate-800 rounded-2xl p-4 shadow-xl border border-slate-700 flex flex-col gap-3 items-center transition-all duration-300">
            <!-- 1行目: 数字モードボタン -->
            <div class="w-full flex flex-wrap gap-2 justify-center items-center">
                <button data-mode="10" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-indigo-600 text-white shadow-md transition">1〜10</button>
                <button data-mode="15" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-slate-700 text-slate-300 hover:bg-slate-600 transition">1〜15</button>
                <button data-mode="20" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-slate-700 text-slate-300 hover:bg-slate-600 transition">1〜20</button>
                <button data-mode="30" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-slate-700 text-slate-300 hover:bg-slate-600 transition">1〜30</button>
            </div>

            <!-- 2行目: TMTモードボタン -->
            <div class="w-full flex flex-wrap gap-2 justify-center items-center">
                <button data-mode="tmt-10" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-slate-700 text-pink-400 border border-pink-500/30 hover:bg-slate-600 transition">
                    <i class="fa-solid fa-brain mr-1"></i>1→あ→2→い (10)
                </button>
                <button data-mode="tmt-20" class="mode-btn px-3.5 py-2 rounded-xl font-bold text-sm bg-slate-700 text-pink-400 border border-pink-500/30 hover:bg-slate-600 transition">
                    <i class="fa-solid fa-brain mr-1"></i>1→あ→2→い (20)
                </button>
            </div>

            <!-- 3行目: スタート / ストップ ボタン -->
            <div class="w-full flex justify-center gap-3 pt-2 border-t border-slate-700/60">
                <button id="startBtn" class="px-6 py-2.5 bg-indigo-600 hover:bg-indigo-500 text-white font-bold rounded-xl shadow-lg transition flex items-center gap-2 text-sm active:scale-95 cursor-pointer">
                    <i class="fa-solid fa-play"></i> スタート
                </button>
                <button id="stopBtn" class="px-6 py-2.5 bg-rose-600 hover:bg-rose-500 text-white font-bold rounded-xl shadow-lg transition flex items-center gap-2 text-sm opacity-40 cursor-not-allowed active:scale-95" disabled>
                    <i class="fa-solid fa-stop"></i> ストップ
                </button>
            </div>
        </section>

        <!-- Dynamic Game Status Header -->
        <section class="grid grid-cols-4 gap-2 md:gap-3">
            <div class="bg-slate-800 rounded-xl p-2.5 md:p-3 border border-slate-700 flex flex-col items-center justify-center">
                <span class="text-[10px] md:text-xs text-slate-400 font-medium">つぎのターゲット</span>
                <span id="nextTargetDisplay" class="text-xl md:text-3xl font-black text-amber-400 mt-0.5">-</span>
            </div>
            <div class="bg-slate-800 rounded-xl p-2.5 md:p-3 border border-slate-700 flex flex-col items-center justify-center">
                <span class="text-[10px] md:text-xs text-slate-400 font-medium">タイム</span>
                <span id="timerDisplay" class="text-xl md:text-3xl font-black text-emerald-400 font-mono mt-0.5">0.00</span>
            </div>
            <div class="bg-slate-800 rounded-xl p-2.5 md:p-3 border border-slate-700 flex flex-col items-center justify-center">
                <span class="text-[10px] md:text-xs text-slate-400 font-medium">自己ベスト</span>
                <span id="bestScoreDisplay" class="text-lg md:text-2xl font-bold text-indigo-400 font-mono mt-0.5">--.--</span>
            </div>
            <!-- ゲーム集中モード時の緊急ストップ（中断）ボタン -->
            <button id="inGameStopBtn" class="bg-rose-600/20 hover:bg-rose-600 text-rose-400 hover:text-white rounded-xl p-2.5 md:p-3 border border-rose-500/40 flex flex-col items-center justify-center transition active:scale-95 cursor-pointer">
                <i class="fa-solid fa-stop text-base md:text-xl mb-0.5"></i>
                <span class="text-[10px] md:text-xs font-bold">中断</span>
            </button>
        </section>

        <!-- Play Area (Game Board) -->
        <div id="boardContainer" class="relative bg-slate-800/60 rounded-3xl border-2 border-slate-700/80 flex-1 overflow-hidden shadow-2xl flex items-center justify-center p-3 touch-none">
            <!-- 描画領域とボタン領域を独立させたゲームエリア -->
            <div id="gameArea" class="relative w-full h-full min-h-[360px] touch-none">
                <!-- 軌跡描画用SVGキャンバス -->
                <svg id="svgOverlay" class="absolute inset-0 w-full h-full pointer-events-none z-0"></svg>
                <!-- タッチボタン用コンテナ -->
                <div id="gameBoard" class="absolute inset-0 w-full h-full z-10 touch-none"></div>
            </div>

            <!-- Overlay Start Screen / Guidance -->
            <div id="startOverlay" class="absolute inset-0 bg-slate-900/90 backdrop-blur-sm z-20 flex flex-col items-center justify-center p-6 text-center transition-opacity duration-300">
                <div class="bg-indigo-500/10 text-indigo-400 rounded-full p-4 mb-3 border border-indigo-500/20">
                    <i class="fa-solid fa-hand-pointer text-4xl animate-bounce"></i>
                </div>
                <h2 class="text-2xl font-bold text-white mb-2">準備はいいですか？</h2>
                <p id="modeInstructions" class="text-sm text-slate-300 max-w-md mb-6">
                    「1」から順番に「10」まで素早くタップしてください！
                </p>
                <button id="overlayStartBtn" class="px-8 py-3 bg-indigo-600 hover:bg-indigo-500 text-white text-lg font-bold rounded-2xl shadow-xl transition transform active:scale-95 flex items-center gap-2 cursor-pointer">
                    <i class="fa-solid fa-play"></i> ゲーム開始
                </button>
            </div>
        </div>

    </main>

    <!-- Result Modal -->
    <div id="resultModal" class="fixed inset-0 bg-black/70 backdrop-blur-md z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl max-w-sm w-full p-6 text-center shadow-2xl transform transition-all scale-95 opacity-0" id="resultModalCard">
            <div class="w-16 h-16 bg-emerald-500/20 text-emerald-400 rounded-2xl flex items-center justify-center mx-auto mb-4 border border-emerald-500/30">
                <i class="fa-solid fa-trophy text-3xl"></i>
            </div>
            <h3 class="text-2xl font-bold text-white mb-1">ステージクリア！</h3>
            <p id="resultModeName" class="text-xs text-slate-400 mb-4">1〜10 モード</p>
            
            <div class="bg-slate-900/80 rounded-2xl p-4 mb-4 border border-slate-700/50">
                <div class="text-xs text-slate-400 mb-1">クリアタイム</div>
                <div id="finalTime" class="text-4xl font-black text-emerald-400 font-mono mb-2">0.00 秒</div>
                <div id="newRecordBadge" class="hidden inline-flex items-center gap-1 bg-amber-500/20 text-amber-300 text-xs font-bold px-3 py-1 rounded-full border border-amber-500/40 animate-pulse">
                    <i class="fa-solid fa-star"></i> 自己新記録達成！
                </div>
            </div>

            <div class="flex gap-3">
                <button id="modalHomeBtn" class="flex-1 py-3 bg-slate-700 hover:bg-slate-600 text-slate-200 font-bold rounded-xl transition shadow-lg cursor-pointer">
                    <i class="fa-solid fa-house mr-1"></i> 戻る
                </button>
                <button id="modalRetryBtn" class="flex-1 py-3 bg-indigo-600 hover:bg-indigo-500 text-white font-bold rounded-xl transition shadow-lg cursor-pointer">
                    <i class="fa-solid fa-rotate-right mr-1"></i> もう一度
                </button>
            </div>
        </div>
    </div>

    <!-- Stats Modal -->
    <div id="statsModal" class="fixed inset-0 bg-black/70 backdrop-blur-md z-50 flex items-center justify-center p-4 hidden">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl max-w-md w-full p-6 shadow-2xl">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-xl font-bold text-white flex items-center gap-2">
                    <i class="fa-solid fa-medal text-amber-400"></i> ベスト記録一覧
                </h3>
                <button id="statsCloseBtn" class="text-slate-400 hover:text-white p-1 cursor-pointer">
                    <i class="fa-solid fa-xmark text-xl"></i>
                </button>
            </div>
            
            <div id="statsList" class="space-y-2 max-h-60 overflow-y-auto pr-1 mb-6">
                <!-- JavaScriptで動的生成 -->
            </div>

            <button id="resetStatsBtn" class="w-full py-2 bg-slate-700 hover:bg-rose-600 text-slate-300 hover:text-white text-xs font-bold rounded-xl transition cursor-pointer">
                <i class="fa-solid fa-trash-can mr-1"></i> 全記録をクリア
            </button>
        </div>
    </div>

    <!-- Footer -->
    <footer id="mainFooter" class="py-3 text-center text-xs text-slate-500 transition-all duration-300">
        Touch Speed & TMT Training App &copy; 2026
    </footer>

    <script>
        // --- タッチによる画面固定/スクロール/ズーム動作の完全ブロック ---
        document.addEventListener('touchmove', function(e) {
            e.preventDefault();
        }, { passive: false });

        document.addEventListener('gesturestart', function(e) {
            e.preventDefault();
        });

        // --- Web Audio API (効果音合成) ---
        class SoundManager {
            constructor() {
                this.audioCtx = null;
                this.enabled = true;
            }

            init() {
                if (!this.audioCtx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    this.audioCtx = new AudioContext();
                }
                if (this.audioCtx.state === 'suspended') {
                    this.audioCtx.resume();
                }
            }

            playCorrect() {
                if (!this.enabled) return;
                this.init();
                const now = this.audioCtx.currentTime;
                const osc = this.audioCtx.createOscillator();
                const gain = this.audioCtx.createGain();

                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, now); // D5
                osc.frequency.exponentialRampToValueAtTime(880, now + 0.1); // A5

                gain.gain.setValueAtTime(0.15, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.15);

                osc.connect(gain);
                gain.connect(this.audioCtx.destination);

                osc.start(now);
                osc.stop(now + 0.15);
            }

            playWrong() {
                if (!this.enabled) return;
                this.init();
                const now = this.audioCtx.currentTime;
                const osc = this.audioCtx.createOscillator();
                const gain = this.audioCtx.createGain();

                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(180, now);
                osc.frequency.linearRampToValueAtTime(120, now + 0.2);

                gain.gain.setValueAtTime(0.2, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.2);

                osc.connect(gain);
                gain.connect(this.audioCtx.destination);

                osc.start(now);
                osc.stop(now + 0.2);
            }

            playComplete() {
                if (!this.enabled) return;
                this.init();
                const notes = [523.25, 659.25, 783.99, 1046.50]; // C5, E5, G5, C6
                notes.forEach((freq, idx) => {
                    const now = this.audioCtx.currentTime + idx * 0.08;
                    const osc = this.audioCtx.createOscillator();
                    const gain = this.audioCtx.createGain();

                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now);

                    gain.gain.setValueAtTime(0.2, now);
                    gain.gain.exponentialRampToValueAtTime(0.01, now + 0.3);

                    osc.connect(gain);
                    gain.connect(this.audioCtx.destination);

                    osc.start(now);
                    osc.stop(now + 0.3);
                });
            }
        }

        const sound = new SoundManager();

        // --- ひらがなリスト (TMT用) ---
        const HIRAGANA = ['あ','い','う','え','お','か','き','く','け','こ','さ','し','す','せ','そ','た','ち','つ','て','と'];

        let currentMode = '10'; // 10, 15, 20, 30, tmt-10, tmt-20
        let targetSequence = []; // 正解の順番配列表
        let currentStepIndex = 0;
        let isPlaying = false;
        let startTime = 0;
        let timerInterval = null;
        let elapsedTime = 0;
        let tappedPositions = []; // 軌跡線描画用

        // DOM Element Cache
        const gameBoard = document.getElementById('gameBoard');
        const boardContainer = document.getElementById('boardContainer');
        const svgOverlay = document.getElementById('svgOverlay');
        const timerDisplay = document.getElementById('timerDisplay');
        const nextTargetDisplay = document.getElementById('nextTargetDisplay');
        const bestScoreDisplay = document.getElementById('bestScoreDisplay');
        const startOverlay = document.getElementById('startOverlay');
        const modeInstructions = document.getElementById('modeInstructions');
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const inGameStopBtn = document.getElementById('inGameStopBtn');
        const mainHeader = document.getElementById('mainHeader');
        const controlsSection = document.getElementById('controlsSection');
        const mainFooter = document.getElementById('mainFooter');
        const overlayStartBtn = document.getElementById('overlayStartBtn');
        const soundToggleBtn = document.getElementById('soundToggleBtn');
        const resultModal = document.getElementById('resultModal');
        const resultModalCard = document.getElementById('resultModalCard');
        const modalRetryBtn = document.getElementById('modalRetryBtn');
        const modalHomeBtn = document.getElementById('modalHomeBtn');
        const statsOpenBtn = document.getElementById('statsOpenBtn');
        const statsCloseBtn = document.getElementById('statsCloseBtn');
        const statsModal = document.getElementById('statsModal');
        const statsList = document.getElementById('statsList');
        const resetStatsBtn = document.getElementById('resetStatsBtn');

        // モード別シーケンス生成関数
        function generateSequence(mode) {
            let seq = [];
            if (mode === '10') {
                for (let i = 1; i <= 10; i++) seq.push(i.toString());
            } else if (mode === '15') {
                for (let i = 1; i <= 15; i++) seq.push(i.toString());
            } else if (mode === '20') {
                for (let i = 1; i <= 20; i++) seq.push(i.toString());
            } else if (mode === '30') {
                for (let i = 1; i <= 30; i++) seq.push(i.toString());
            } else if (mode === 'tmt-10') {
                for (let i = 0; i < 5; i++) {
                    seq.push((i + 1).toString());
                    seq.push(HIRAGANA[i]);
                }
            } else if (mode === 'tmt-20') {
                for (let i = 0; i < 10; i++) {
                    seq.push((i + 1).toString());
                    seq.push(HIRAGANA[i]);
                }
            }
            return seq;
        }

        // ガイドメッセージ取得
        function getModeInstructionText(mode) {
            if (mode.startsWith('tmt')) {
                return '数字とひらがなを交互にタップしてください！<br><span class="text-pink-400 font-bold">例: 1 → あ → 2 → い → 3 → う...</span>';
            }
            return `「1」から順番に「${mode}」まで素早くタップしてください！`;
        }

        // ボタン配置の生成（枠からはみ出さない完全計算）
        function createBoardButtons() {
            gameBoard.innerHTML = '';
            svgOverlay.innerHTML = '';
            tappedPositions = [];

            const width = gameBoard.clientWidth || boardContainer.clientWidth || 340;
            const height = gameBoard.clientHeight || boardContainer.clientHeight || 360;

            const count = targetSequence.length;
            let btnSize = 56;
            if (width < 380) {
                btnSize = count > 20 ? 40 : count > 15 ? 44 : 48;
            } else {
                btnSize = count > 20 ? 44 : count > 15 ? 48 : 54;
            }

            const padding = 20; // 外枠からの安全マージン
            const placedRects = [];

            const shuffledItems = [...targetSequence].sort(() => Math.random() - 0.5);

            shuffledItems.forEach((val) => {
                let attempts = 0;
                let x = 0, y = 0;
                let overlap = false;

                const maxX = width - btnSize - padding;
                const maxY = height - btnSize - padding;
                const minX = padding;
                const minY = padding;

                do {
                    overlap = false;
                    x = minX + Math.random() * (maxX - minX);
                    y = minY + Math.random() * (maxY - minY);

                    for (const r of placedRects) {
                        const dist = Math.hypot(x - r.x, y - r.y);
                        if (dist < btnSize + 8) {
                            overlap = true;
                            break;
                        }
                    }
                    attempts++;
                } while (overlap && attempts < 300);

                placedRects.push({ x, y });

                const btn = document.createElement('button');
                btn.className = `bubble-btn absolute rounded-full flex items-center justify-center font-black shadow-lg cursor-pointer border-2 select-none text-slate-800 touch-none`;
                btn.style.width = `${btnSize}px`;
                btn.style.height = `${btnSize}px`;
                btn.style.left = `${x}px`;
                btn.style.top = `${y}px`;

                btn.textContent = val;
                if (isNaN(val)) {
                    btn.classList.add('bg-gradient-to-br', 'from-pink-300', 'to-rose-400', 'border-rose-200', 'text-slate-900');
                    btn.style.fontSize = btnSize > 48 ? '1.2rem' : '0.95rem';
                } else {
                    btn.classList.add('bg-gradient-to-br', 'from-cyan-200', 'to-indigo-300', 'border-indigo-100', 'text-slate-900');
                    btn.style.fontSize = btnSize > 48 ? '1.3rem' : '1.05rem';
                }

                btn.dataset.value = val;

                btn.addEventListener('pointerdown', (e) => {
                    e.preventDefault();
                    handleTargetTap(btn, val);
                });

                gameBoard.appendChild(btn);
            });
        }

        function handleTargetTap(btn, val) {
            if (!isPlaying) return;

            const expected = targetSequence[currentStepIndex];

            if (val === expected) {
                sound.playCorrect();
                
                // 画面上の物理的な位置から正確な中心座標を計算
                const svgRect = svgOverlay.getBoundingClientRect();
                const btnRect = btn.getBoundingClientRect();
                const cx = (btnRect.left + btnRect.width / 2) - svgRect.left;
                const cy = (btnRect.top + btnRect.height / 2) - svgRect.top;

                tappedPositions.push({ x: cx, y: cy });
                drawConnectingLines();

                btn.style.pointerEvents = 'none';
                btn.classList.add('animate-pop');
                setTimeout(() => {
                    btn.style.opacity = '0.15';
                    btn.classList.remove('animate-pop');
                    btn.classList.add('scale-75', 'grayscale');
                }, 200);

                currentStepIndex++;

                if (currentStepIndex < targetSequence.length) {
                    nextTargetDisplay.textContent = targetSequence[currentStepIndex];
                } else {
                    completeGame();
                }
            } else {
                sound.playWrong();
                btn.classList.remove('animate-shake');
                void btn.offsetWidth;
                btn.classList.add('animate-shake');
            }
        }

        function drawConnectingLines() {
            if (tappedPositions.length < 2) return;
            
            const prev = tappedPositions[tappedPositions.length - 2];
            const curr = tappedPositions[tappedPositions.length - 1];

            const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
            line.setAttribute('x1', prev.x);
            line.setAttribute('y1', prev.y);
            line.setAttribute('x2', curr.x);
            line.setAttribute('y2', curr.y);
            line.setAttribute('stroke', '#6366F1');
            line.setAttribute('stroke-width', '4');
            line.setAttribute('stroke-dasharray', '6 4');
            line.setAttribute('stroke-linecap', 'round');
            line.setAttribute('opacity', '0.7');

            svgOverlay.appendChild(line);
        }

        // 集中プレイモードの表示切替
        function setFocusMode(active) {
            if (active) {
                mainHeader?.classList.add('hidden');
                controlsSection?.classList.add('hidden');
                mainFooter?.classList.add('hidden');
            } else {
                mainHeader?.classList.remove('hidden');
                controlsSection?.classList.remove('hidden');
                mainFooter?.classList.remove('hidden');
            }
        }

        function startGame() {
            targetSequence = generateSequence(currentMode);
            currentStepIndex = 0;
            elapsedTime = 0;

            // 全画面（集中モード）の有効化
            setFocusMode(true);

            // ボード生成
            createBoardButtons();

            nextTargetDisplay.textContent = targetSequence[0];
            timerDisplay.textContent = '0.00';
            
            startOverlay.classList.add('opacity-0', 'pointer-events-none');
            setTimeout(() => {
                startOverlay.classList.add('hidden');
            }, 300);

            isPlaying = true;
            startTime = performance.now();

            if (stopBtn) {
                stopBtn.disabled = false;
                stopBtn.classList.remove('opacity-40', 'cursor-not-allowed');
            }

            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                if (!isPlaying) return;
                const now = performance.now();
                elapsedTime = (now - startTime) / 1000;
                timerDisplay.textContent = elapsedTime.toFixed(2);
            }, 30);
        }

        function stopGame() {
            isPlaying = false;
            clearInterval(timerInterval);

            // 集中モード解除（設定画面やヘッダーを復活）
            setFocusMode(false);

            if (stopBtn) {
                stopBtn.disabled = true;
                stopBtn.classList.add('opacity-40', 'cursor-not-allowed');
            }

            startOverlay.classList.remove('hidden');
            setTimeout(() => {
                startOverlay.classList.remove('opacity-0', 'pointer-events-none');
            }, 10);

            gameBoard.innerHTML = '';
            svgOverlay.innerHTML = '';
            nextTargetDisplay.textContent = '-';
            timerDisplay.textContent = '0.00';
        }

        function completeGame() {
            isPlaying = false;
            clearInterval(timerInterval);
            sound.playComplete();

            if (stopBtn) {
                stopBtn.disabled = true;
                stopBtn.classList.add('opacity-40', 'cursor-not-allowed');
            }

            nextTargetDisplay.textContent = 'CLEAR!';
            
            confetti({
                particleCount: 100,
                spread: 70,
                origin: { y: 0.6 }
            });

            const isNewRecord = saveBestScore(currentMode, elapsedTime);
            updateBestDisplay();

            document.getElementById('finalTime').textContent = `${elapsedTime.toFixed(2)} 秒`;
            document.getElementById('resultModeName').textContent = getModeTitle(currentMode);

            const recordBadge = document.getElementById('newRecordBadge');
            if (isNewRecord) {
                recordBadge.classList.remove('hidden');
            } else {
                recordBadge.classList.add('hidden');
            }

            setTimeout(() => {
                resultModal.classList.remove('hidden');
                setTimeout(() => {
                    resultModalCard.classList.remove('scale-95', 'opacity-0');
                    resultModalCard.classList.add('scale-100', 'opacity-100');
                }, 50);
            }, 400);
        }

        function getModeTitle(mode) {
            if (mode === 'tmt-10') return '1 → あ → 2 → い (10項目)';
            if (mode === 'tmt-20') return '1 → あ → 2 → い (20項目)';
            return `1 〜 ${mode}`;
        }

        function getBestScores() {
            try {
                return JSON.parse(localStorage.getItem('touch_speed_best_scores') || '{}');
            } catch (e) {
                return {};
            }
        }

        function saveBestScore(mode, time) {
            const scores = getBestScores();
            const currentBest = scores[mode];

            if (!currentBest || time < currentBest) {
                scores[mode] = time;
                localStorage.setItem('touch_speed_best_scores', JSON.stringify(scores));
                return true;
            }
            return false;
        }

        function updateBestDisplay() {
            const scores = getBestScores();
            const best = scores[currentMode];
            if (best) {
                bestScoreDisplay.textContent = `${best.toFixed(2)}s`;
            } else {
                bestScoreDisplay.textContent = '--.--';
            }
        }

        function renderStatsModal() {
            const scores = getBestScores();
            const modes = ['10', '15', '20', '30', 'tmt-10', 'tmt-20'];
            
            statsList.innerHTML = '';
            modes.forEach((m) => {
                const row = document.createElement('div');
                row.className = 'flex justify-between items-center bg-slate-900/60 px-4 py-2.5 rounded-xl border border-slate-700/50';
                
                const name = document.createElement('span');
                name.className = 'text-sm font-medium text-slate-300';
                name.textContent = getModeTitle(m);

                const val = document.createElement('span');
                val.className = 'text-sm font-bold font-mono text-emerald-400';
                val.textContent = scores[m] ? `${scores[m].toFixed(2)} 秒` : '--.--';

                row.appendChild(name);
                row.appendChild(val);
                statsList.appendChild(row);
            });
        }

        document.querySelectorAll('.mode-btn').forEach((btn) => {
            btn.addEventListener('click', () => {
                stopGame();

                document.querySelectorAll('.mode-btn').forEach(b => {
                    b.classList.remove('bg-indigo-600', 'text-white');
                    b.classList.add('bg-slate-700', 'text-slate-300');
                });
                btn.classList.remove('bg-slate-700', 'text-slate-300');
                btn.classList.add('bg-indigo-600', 'text-white');

                currentMode = btn.dataset.mode;
                targetSequence = generateSequence(currentMode);

                modeInstructions.innerHTML = getModeInstructionText(currentMode);
                updateBestDisplay();
            });
        });

        // ボタンのクリックイベントバインド
        startBtn?.addEventListener('click', startGame);
        overlayStartBtn?.addEventListener('click', startGame);
        stopBtn?.addEventListener('click', stopGame);
        inGameStopBtn?.addEventListener('click', stopGame);

        modalRetryBtn?.addEventListener('click', () => {
            resultModalCard.classList.remove('scale-100', 'opacity-100');
            resultModalCard.classList.add('scale-95', 'opacity-0');
            setTimeout(() => {
                resultModal.classList.add('hidden');
                startGame();
            }, 200);
        });

        modalHomeBtn?.addEventListener('click', () => {
            resultModalCard.classList.remove('scale-100', 'opacity-100');
            resultModalCard.classList.add('scale-95', 'opacity-0');
            setTimeout(() => {
                resultModal.classList.add('hidden');
                stopGame();
            }, 200);
        });

        soundToggleBtn?.addEventListener('click', () => {
            sound.enabled = !sound.enabled;
            if (sound.enabled) {
                soundToggleBtn.classList.remove('text-slate-500');
                soundToggleBtn.classList.add('text-amber-400');
                soundToggleBtn.innerHTML = '<i class="fa-solid fa-volume-high text-lg"></i>';
            } else {
                soundToggleBtn.classList.remove('text-amber-400');
                soundToggleBtn.classList.add('text-slate-500');
                soundToggleBtn.innerHTML = '<i class="fa-solid fa-volume-xmark text-lg"></i>';
            }
        });

        statsOpenBtn?.addEventListener('click', () => {
            renderStatsModal();
            statsModal.classList.remove('hidden');
        });

        statsCloseBtn?.addEventListener('click', () => {
            statsModal.classList.add('hidden');
        });

        let confirmResetState = false;
        resetStatsBtn?.addEventListener('click', () => {
            if (!confirmResetState) {
                confirmResetState = true;
                resetStatsBtn.textContent = '本当に消去しますか？（タップで確定）';
                resetStatsBtn.classList.remove('bg-slate-700', 'hover:bg-rose-600');
                resetStatsBtn.classList.add('bg-rose-600', 'text-white');
                setTimeout(() => {
                    confirmResetState = false;
                    resetStatsBtn.innerHTML = '<i class="fa-solid fa-trash-can mr-1"></i> 全記録をクリア';
                    resetStatsBtn.classList.remove('bg-rose-600', 'text-white');
                    resetStatsBtn.classList.add('bg-slate-700', 'hover:bg-rose-600');
                }, 3000);
            } else {
                localStorage.removeItem('touch_speed_best_scores');
                renderStatsModal();
                updateBestDisplay();
                confirmResetState = false;
                resetStatsBtn.innerHTML = '<i class="fa-solid fa-trash-can mr-1"></i> 全記録をクリア';
                resetStatsBtn.classList.remove('bg-rose-600', 'text-white');
                resetStatsBtn.classList.add('bg-slate-700', 'hover:bg-rose-600');
            }
        });

        window.addEventListener('resize', () => {
            if (isPlaying) {
                createBoardButtons();
            }
        });

        window.addEventListener('DOMContentLoaded', () => {
            targetSequence = generateSequence(currentMode);
            if (modeInstructions) modeInstructions.innerHTML = getModeInstructionText(currentMode);
            updateBestDisplay();
        });
    </script>
</body>
</html>
