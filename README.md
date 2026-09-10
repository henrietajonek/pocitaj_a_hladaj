<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Počítaj a hľadaj</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body, button, input, select, textarea, div, span, h1, p {
            font-family: Arial, Helvetica, sans-serif !important;
        }

        /* Presné farebné pásy podľa priloženého obrázka */
        .band-1, .band-2 { background-color: #dbe3fc; } /* svetlomodrá / fialkavá */
        .band-3          { background-color: #d3f5f8; } /* cyan / tyrkysová */
        .band-4          { background-color: #e1f7d5; } /* svetlozelená */
        .band-5          { background-color: #fffecb; } /* svetložltá */
        .band-6          { background-color: #fde3d0; } /* pastelová oranžová/broskyňová */
        .band-7          { background-color: #fff2c7; } /* tepložltá */
        .band-8          { background-color: #fce1ed; } /* ružová */
        .band-9          { background-color: #eae0fd; } /* svetlofialová */
        .band-10         { background-color: #dbe3fc; } /* svetlomodrá */

        .grid-cell {
            transition: transform 0.15s ease, filter 0.15s ease;
            user-select: none;
        }

        .grid-cell:hover {
            transform: scale(1.05);
            z-index: 10;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
            filter: brightness(0.96);
        }

        /* Schované čísla - farba pozadia ostáva, text je priesvitný */
        .cell-hidden {
            color: transparent !important;
            cursor: pointer;
        }

        .cell-hidden:hover {
            filter: brightness(0.93);
        }

        /* Výrazné deliace čiary ako na obrázku */
        .thick-border-r {
            border-right: 2.5px solid #000000 !important;
        }
        .thick-border-b {
            border-bottom: 2.5px solid #000000 !important;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-between p-2 sm:p-6 bg-slate-50 text-slate-900">

    <!-- Hlavný kontajner -->
    <div class="w-full max-w-4xl flex flex-col items-center">
       <!-- Nadpis -->
<header class="text-center my-4">
    
    <h1 class="text-3xl sm:text-5xl font-bold tracking-tight"
        style="color: #000000;">
        Počítaj a hľadaj
    </h1>

    <p class="mt-2 text-sm sm:text-base max-w-lg"
       style="color: #000000;">
        Klikni na políčko pre odhalenie alebo schovanie výsledku. 
        Čísla sa odhalia naraz pre 
        <span class="font-bold" style="color: #000000">
            A × B
        </span> 
        aj 
        <span class="font-bold" style="color: #000000">
            B × A
        </span>.
    </p>

</header>

        <!-- Ovládacie tlačidlá -->
<div class="flex flex-wrap justify-center gap-2 sm:gap-4 mb-6 w-full">

    <button id="btn-show-all" 
            style="background-color: #9D69B6;"
            class="hover:opacity-80 text-white font-bold py-2.5 px-5 rounded-xl shadow transition active:scale-95 text-sm sm:text-base">
        Odhaliť všetko
    </button>

    <button id="btn-hide-all" 
            style="background-color: #69B6AD;"
            class="hover:opacity-80 text-white font-bold py-2.5 px-5 rounded-xl shadow transition active:scale-95 text-sm sm:text-base">
        Schovať všetko
    </button>

    <button id="btn-quiz" 
            style="background-color: #6983B6;"
            class="hover:opacity-80 text-white font-bold py-2.5 px-5 rounded-xl shadow transition active:scale-95 text-sm sm:text-base">
        🎯 Vyskúšaj sa!
    </button>

</div>

        <!-- Panel pre kvíz -->
        <div id="quiz-panel" class="hidden w-full max-w-md bg-emerald-50 border-2 border-emerald-400 rounded-2xl p-4 mb-6 text-center shadow-md">
            <span id="quiz-question" class="text-xl sm:text-2xl font-bold text-emerald-900">
                Nájdi políčko s výsledkom: 4 × 3
            </span>
            <p id="quiz-feedback" class="text-sm mt-1 font-bold h-6 text-emerald-700"></p>
        </div>

        <!-- Tabuľka násobilky -->
        <div class="w-full overflow-x-auto pb-4 flex justify-center">
            <div class="inline-block border-2 border-black rounded-sm shadow-2xl overflow-hidden bg-black">
                <div id="multiplication-grid" class="grid grid-cols-11 gap-px bg-slate-400">
                    <!-- Mriežka vygenerovaná skriptom -->
                </div>
            </div>
        </div>

        <!-- Štatistika odhalených políčok -->
        <div class="mt-4 text-slate-600 text-sm font-bold flex items-center gap-2">
            <span>Odhalené políčka:</span>
            <span id="revealed-count" class="bg-indigo-100 text-indigo-800 px-3 py-1 rounded-full text-base font-bold">0 / 100</span>
        </div>

    </div>

    <!-- Päta -->
    <footer class="mt-8 text-center text-xs text-slate-400 font-bold">
        Počítaj a hľadaj &bull; Interaktívna malá násobilka
    </footer>

    <!-- JS Logika -->
    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const grid = document.getElementById('multiplication-grid');
            const revealedCountEl = document.getElementById('revealed-count');
            const btnShowAll = document.getElementById('btn-show-all');
            const btnHideAll = document.getElementById('btn-hide-all');
            const btnQuiz = document.getElementById('btn-quiz');
            const quizPanel = document.getElementById('quiz-panel');
            const quizQuestion = document.getElementById('quiz-question');
            const quizFeedback = document.getElementById('quiz-feedback');

            // Stav odhalenia [row][col] (1..10)
            const gridState = {};
            for (let r = 1; r <= 10; r++) {
                gridState[r] = {};
                for (let c = 1; c <= 10; c++) {
                    gridState[r][c] = false;
                }
            }

            let quizActive = false;
            let currentQuiz = { a: 0, b: 0, result: 0 };

            // Pomocná funkcia pre získanie triedy pásu podľa max(r, c)
            function getBandClass(r, c) {
                const bandIndex = Math.max(r, c);
                return `band-${bandIndex}`;
            }

            // Vytvorenie mriežky
            function createGrid() {
                grid.innerHTML = '';

                // Políčko vľavo hore [0, 0] s bodkou
                const topLeft = document.createElement('div');
                topLeft.className = 'w-8 h-8 sm:w-12 sm:h-12 flex items-center justify-center font-bold text-lg sm:text-2xl text-black band-1 thick-border-r thick-border-b select-none';
                topLeft.innerHTML = '•';
                grid.appendChild(topLeft);

                // Prvý riadok - činitele (stĺpce 1-10)
                for (let c = 1; c <= 10; c++) {
                    const headerCol = document.createElement('div');
                    const isLastCol = (c === 10);
                    headerCol.className = `w-8 h-8 sm:w-12 sm:h-12 flex items-center justify-center font-bold text-base sm:text-xl text-black band-${c} thick-border-b select-none`;
                    headerCol.innerText = c;
                    grid.appendChild(headerCol);
                }

                // Riadky 1 až 10
                for (let r = 1; r <= 10; r++) {
                    // Prvý stĺpec v riadku - činiteľ
                    const headerRow = document.createElement('div');
                    headerRow.className = `w-8 h-8 sm:w-12 sm:h-12 flex items-center justify-center font-bold text-base sm:text-xl text-black band-${r} thick-border-r select-none`;
                    headerRow.innerText = r;
                    grid.appendChild(headerRow);

                    // Políčka so súčinmi
                    for (let c = 1; c <= 10; c++) {
                        const cell = document.createElement('div');
                        cell.id = `cell-${r}-${c}`;
                        cell.dataset.row = r;
                        cell.dataset.col = c;
                        
                        const bandClass = getBandClass(r, c);
                        cell.className = `grid-cell w-8 h-8 sm:w-12 sm:h-12 flex items-center justify-center font-bold text-sm sm:text-xl text-black ${bandClass} cell-hidden`;
                        cell.innerText = r * c;

                        cell.addEventListener('click', () => handleCellClick(r, c));
                        grid.appendChild(cell);
                    }
                }
            }

            // Kliknutie na políčko
            function handleCellClick(r, c) {
                const newState = !gridState[r][c];
                
                // Zmena stavu pre A x B aj B x A
                gridState[r][c] = newState;
                gridState[c][r] = newState;

                updateCellVisual(r, c);
                updateCellVisual(c, r);

                updateCount();

                if (quizActive && newState) {
                    checkQuizAnswer(r, c);
                }
            }

            // Aktualizácia vzhľadu
            function updateCellVisual(r, c) {
                const cell = document.getElementById(`cell-${r}-${c}`);
                if (!cell) return;

                if (gridState[r][c]) {
                    cell.classList.remove('cell-hidden');
                } else {
                    cell.classList.add('cell-hidden');
                }
            }

            // Počítadlo odhalených
            function updateCount() {
                let count = 0;
                for (let r = 1; r <= 10; r++) {
                    for (let c = 1; c <= 10; c++) {
                        if (gridState[r][c]) count++;
                    }
                }
                revealedCountEl.innerText = `${count} / 100`;
            }

            // Odhaliť všetko
            btnShowAll.addEventListener('click', () => {
                for (let r = 1; r <= 10; r++) {
                    for (let c = 1; c <= 10; c++) {
                        gridState[r][c] = true;
                        updateCellVisual(r, c);
                    }
                }
                updateCount();
            });

            // Schovať všetko
            btnHideAll.addEventListener('click', () => {
                for (let r = 1; r <= 10; r++) {
                    for (let c = 1; c <= 10; c++) {
                        gridState[r][c] = false;
                        updateCellVisual(r, c);
                    }
                }
                updateCount();
            });

            // Kvíz
            btnQuiz.addEventListener('click', () => {
                quizActive = !quizActive;
                if (quizActive) {
                    quizPanel.classList.remove('hidden');
                    btnQuiz.classList.replace('bg-emerald-600', 'bg-amber-600');
                    btnQuiz.classList.replace('hover:bg-emerald-700', 'hover:bg-amber-700');
                    btnQuiz.innerText = '✖ Ukončiť cvičenie';
                    nextQuizQuestion();
                } else {
                    quizPanel.classList.add('hidden');
                    btnQuiz.classList.replace('bg-amber-600', 'bg-emerald-600');
                    btnQuiz.classList.replace('hover:bg-amber-700', 'hover:bg-emerald-700');
                    btnQuiz.innerText = '🎯 Vyskúšaj sa!';
                }
            });

            function nextQuizQuestion() {
                currentQuiz.a = Math.floor(Math.random() * 10) + 1;
                currentQuiz.b = Math.floor(Math.random() * 10) + 1;
                currentQuiz.result = currentQuiz.a * currentQuiz.b;

                quizQuestion.innerText = `Nájdi výsledok pre: ${currentQuiz.a} × ${currentQuiz.b}`;
                quizFeedback.innerText = '';
                quizFeedback.className = 'text-sm mt-1 font-bold h-6 text-emerald-700';
            }

            function checkQuizAnswer(r, c) {
                if ((r === currentQuiz.a && c === currentQuiz.b) || (r === currentQuiz.b && c === currentQuiz.a)) {
                    quizFeedback.innerText = '🎉 Výborne! Správne políčko!';
                    quizFeedback.className = 'text-sm mt-1 font-bold h-6 text-emerald-700';
                    setTimeout(() => {
                        nextQuizQuestion();
                    }, 1400);
                } else if (r * c === currentQuiz.result) {
                    quizFeedback.innerText = `👏 Výborne, ${r} × ${c} je tiež ${currentQuiz.result}!`;
                    quizFeedback.className = 'text-sm mt-1 font-bold h-6 text-emerald-700';
                    setTimeout(() => {
                        nextQuizQuestion();
                    }, 1400);
                } else {
                    quizFeedback.innerText = `Toto políčko je ${r * c}. Hľadáme ${currentQuiz.result}!`;
                    quizFeedback.className = 'text-sm mt-1 font-bold h-6 text-rose-600';
                }
            }

            // Štart
            createGrid();
        });
    </script>
</body>
</html>
