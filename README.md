<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BB84 양자 암호화 시뮬레이터 (학생용)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
        }
        .main-container {
            transition: all 0.3s ease-in-out;
        }
        .basis-rect { background-color: #fca5a5; } /* Red-300 for Rectilinear (+) */
        .basis-diag { background-color: #93c5fd; } /* Blue-300 for Diagonal (x) */
        .bit-0 { color: #3b82f6; font-weight: bold; } /* Blue-600 */
        .bit-1 { color: #ef4444; font-weight: bold; } /* Red-600 */
        .key-match { background-color: #dcfce7; } /* Green-100 */
        .key-mismatch { background-color: #fee2e2; } /* Red-100 */
        .step-section {
            transition: all 0.5s ease-in-out;
        }
        .qubit, .qubit-input {
            display: inline-flex;
            justify-content: center;
            align-items: center;
            width: 2.5rem;
            height: 2.5rem;
            font-size: 1.5rem;
            font-weight: bold;
            border-radius: 50%;
            margin: 0.25rem;
        }
        .qubit-input {
            border: 2px dashed #9ca3af;
            padding: 0;
            text-align: center;
        }
        .input-area {
            background-color: #f9fafb;
        }
        .dark .input-area {
             background-color: #1f2937;
        }
        input[type="text"] {
            font-family: 'Courier New', Courier, monospace;
            letter-spacing: 0.5em;
            text-align: center;
        }
        .feedback-message {
            display: none;
            margin-top: 0.75rem;
            padding: 0.75rem;
            border-radius: 0.5rem;
        }
        .feedback-message.correct {
            display: block;
            background-color: #dcfce7; color: #166534;
        }
        .dark .feedback-message.correct {
            background-color: #14532d; color: #dcfce7;
        }
        .feedback-message.incorrect {
            display: block;
            background-color: #fee2e2; color: #991b1b;
        }
        .dark .feedback-message.incorrect {
            background-color: #7f1d1d; color: #fee2e2;
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-200">

    <div id="mainContainer" class="main-container container mx-auto p-4 sm:p-6 md:p-8 max-w-5xl border-2 border-transparent rounded-lg">
        <header class="text-center mb-8 relative overflow-hidden rounded-lg">
            <div class="absolute inset-0 opacity-10 bg-no-repeat bg-center bg-cover" style="background-image: url('https://images.unsplash.com/photo-1535223289827-42f1e9919769?q=80&w=1887&auto=format&fit=crop'); z-index: -1;"></div>
            <div class="bg-white/70 dark:bg-gray-900/70 backdrop-blur-sm py-6">
                <h1 class="text-3xl md:text-4xl font-bold text-sky-600 dark:text-sky-400">BB84 양자 암호화 시뮬레이션</h1>
                <p class="text-gray-700 dark:text-gray-300 mt-2 text-sm md:text-base">BB84 프로토콜을 단계별로 체험하며 양자 암호의 원리를 이해해 보세요.</p>
                <p class="text-gray-500 dark:text-gray-400 text-xs mt-1">(테이블이 화면보다 넓을 경우, 좌우로 스크롤하여 볼 수 있습니다.)</p>
            </div>
        </header>

        <!-- Global Controls -->
        <div class="bg-white dark:bg-gray-800 shadow-lg rounded-lg p-6 mb-8 flex flex-wrap items-center justify-center gap-6">
             <div class="flex items-center gap-3">
                <label for="eveToggle" class="font-medium text-red-500">도청자 (이브) 개입:</label>
                <input type="checkbox" id="eveToggle" class="h-6 w-6 rounded text-red-600 focus:ring-red-500">
            </div>
            <button id="resetButton" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-6 rounded-lg transition duration-300">
                처음부터 다시하기
            </button>
        </div>

        <!-- Simulation Steps -->
        <div id="simulationArea" class="space-y-6">
            <!-- Step 1: Alice's Input -->
            <section id="aliceInputStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6">
                <h2 class="text-2xl font-semibold mb-4 text-rose-500">1단계: 갑돌이 (앨리스) - 비트/기준 입력</h2>
                <p class="mb-4">공유할 원본 비트(0 또는 1)와 각 비트를 보낼 편광 기준(+, x)을 입력하세요.</p>
                <div class="input-area border dark:border-gray-700 rounded-lg p-4 space-y-4">
                    <input type="text" id="aliceBitsInput" placeholder="원본 비트 입력 (예: 0110)" class="w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg p-2">
                    <input type="text" id="aliceBasesInput" placeholder="편광 기준 입력 (예: +x+x)" class="w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg p-2">
                </div>
                <p id="aliceInputError" class="feedback-message incorrect"></p>
                <button id="alicePrepareButton" class="mt-4 w-full bg-rose-500 hover:bg-rose-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">다음: 큐비트 예측하기</button>
            </section>
            
            <!-- Step 1-2: Alice's Qubit Prediction -->
            <section id="alicePredictStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 text-rose-500">1-2단계: 갑돌이 (앨리스) - 큐비트 예측</h2>
                <p class="mb-4">입력한 비트와 기준에 따라 어떤 큐비트(광자)가 생성될지 예측하여 아래에 입력하세요.
                     <br><span class="text-sm text-gray-600 dark:text-gray-400 mt-1 block">(힌트: 0은 ↔ 또는 ⤢, 1은 ↕ 또는 ⤡ 입니다. 키보드의 <b>- | / \</b> 문자로도 입력할 수 있습니다.)</span>
                </p>
                <div id="aliceQubitPredictionArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="alicePredictError" class="feedback-message incorrect"></p>
                <p id="alicePredictCorrect" class="feedback-message correct"></p>
                <button id="aliceCheckButton" class="mt-4 w-full bg-rose-500 hover:bg-rose-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">정답 확인</button>
            </section>
            
            <!-- Eve's Steps -->
            <section id="eveInputStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden border-4 border-red-500">
                <h2 class="text-2xl font-semibold mb-4 text-red-500">도청자 (이브) 차례 - 측정 기준 입력</h2>
                <p class="mb-4">갑돌이가 보낸 큐비트를 엿듣기 위해 사용할 자신만의 편광 기준(+, x)을 입력하세요.</p>
                <div class="input-area border dark:border-gray-700 rounded-lg p-4">
                    <input type="text" id="eveBasesInput" placeholder="도청 기준 입력 (예: x++x)" class="w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg p-2">
                </div>
                <p id="eveInputError" class="feedback-message incorrect"></p>
                <button id="evePrepareButton" class="mt-4 w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">다음: 측정 후 큐비트 예측</button>
            </section>

            <section id="evePredictQubitStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden border-4 border-red-500">
                <h2 class="text-2xl font-semibold mb-4 text-red-500">도청자 (이브) 차례 - 측정 후 큐비트 상태 예측</h2>
                <p class="mb-4">자신의 기준으로 큐비트를 측정한 '후'의 큐비트 상태를 예측해보세요. <br><span class="text-sm text-gray-500">(힌트: 갑돌이와 기준이 <b>일치하는 곳</b>만 입력하고, 다른 곳은 비워두세요.)</span></p>
                <div id="eveQubitPredictionArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="eveQubitPredictError" class="feedback-message incorrect"></p>
                <p id="eveQubitPredictCorrect" class="feedback-message correct"></p>
                <button id="eveQubitCheckButton" class="mt-4 w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">정답 확인</button>
            </section>

            <section id="evePredictBitStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden border-4 border-red-500">
                <h2 class="text-2xl font-semibold mb-4 text-red-500">도청자 (이브) 차례 - 측정 결과 예측</h2>
                <p class="mb-4">위에서 확인된 **측정 후 큐비트 상태**가 어떤 비트(0,1)로 인식될지 예측해보세요.<br><span class="text-sm text-gray-500">(힌트: 갑돌이와 기준이 같았던 곳만 예측하면 됩니다.)</span></p>
                <div id="eveBitPredictionArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="eveBitPredictError" class="feedback-message incorrect"></p>
                <p id="eveBitPredictCorrect" class="feedback-message correct"></p>
                <button id="eveBitCheckButton" class="mt-4 w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">정답 확인</button>
            </section>

             <section id="eveResendStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden border-4 border-red-500">
                <h2 class="text-2xl font-semibold mb-4 text-red-500">도청자 (이브) 차례 - 큐비트 재생성</h2>
                <p class="mb-4">들키지 않기 위해, 자신이 측정한 비트와 기준으로 새로운 큐비트를 만들어 을순이에게 보냅니다. 생성될 큐비트를 예측해보세요.</p>
                <div id="eveQubitResendArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="eveResendError" class="feedback-message incorrect"></p>
                <p id="eveResendCorrect" class="feedback-message correct"></p>
                <button id="eveResendCheckButton" class="mt-4 w-full bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">큐비트 전송</button>
            </section>

            <!-- Bob's Steps -->
            <section id="bobInputStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 text-sky-500">2단계: 을순이 (밥) - 측정 기준 입력</h2>
                <p class="mb-4">전송받은 큐비트를 측정할 자신만의 편광 기준(+, x)을 갑돌이와 동일한 길이로 입력하세요.</p>
                <div class="input-area border dark:border-gray-700 rounded-lg p-4">
                    <input type="text" id="bobBasesInput" placeholder="측정 기준 입력 (예: ++xx)" class="w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg p-2">
                </div>
                 <p id="bobInputError" class="feedback-message incorrect"></p>
                <button id="bobPrepareButton" class="mt-4 w-full bg-sky-500 hover:bg-sky-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">다음: 측정 후 큐비트 예측</button>
            </section>

            <section id="bobPredictQubitStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 text-sky-500">2-1단계: 을순이 (밥) - 측정 후 큐비트 상태 예측</h2>
                <p class="mb-4">자신의 기준으로 큐비트를 측정한 '후'의 큐비트 상태를 예측해보세요. <br><span class="text-sm text-gray-500">(힌트: 갑돌이와 기준이 <b>일치하는 곳</b>의 큐비트 상태만 입력하세요. 기준이 다른 곳은 비워두시면 됩니다.)</span></p>
                <div id="bobQubitPredictionArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="bobQubitPredictError" class="feedback-message incorrect"></p>
                <p id="bobQubitPredictCorrect" class="feedback-message correct"></p>
                <button id="bobQubitCheckButton" class="mt-4 w-full bg-sky-500 hover:bg-sky-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">정답 확인</button>
            </section>

            <section id="bobPredictBitStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 text-sky-500">2-2단계: 을순이 (밥) - 측정 결과 예측</h2>
                <p class="mb-4">이제, 위에서 확인된 **측정 후 큐비트 상태**가 어떤 비트(0 또는 1)로 인식될지 예측해보세요.<br><span class="text-sm text-gray-500">(힌트: 갑돌이와 기준이 일치했던 곳의 비트만 예측하면 됩니다.)</span></p>
                <div id="bobBitPredictionArea" class="input-area border dark:border-gray-700 rounded-lg p-4 flex flex-wrap justify-center items-center"></div>
                <p id="bobBitPredictError" class="feedback-message incorrect"></p>
                <p id="bobBitPredictCorrect" class="feedback-message correct"></p>
                <button id="bobBitCheckButton" class="mt-4 w-full bg-sky-500 hover:bg-sky-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">정답 확인</button>
            </section>

            <!-- Step 3: Sifting -->
            <section id="siftingStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 border-b pb-2 text-green-500">3단계: 공개 채널 - 기준 비교 및 키 선별</h2>
                <p class="mb-4">두 사람은 이제 서로의 기준을 공개적으로 비교합니다. 기준이 일치했던 부분의 비트만 모아 **을순이 자신의 최종 키**를 만들어 아래에 입력하세요.</p>
                 <div class="overflow-x-auto mb-4">
                    <table class="w-full text-center">
                        <thead><tr class="bg-gray-100 dark:bg-gray-700"><th class="p-2">갑돌이(앨리스) 기준</th><th class="p-2">을순이(밥) 기준</th></tr></thead>
                        <tbody><tr><td id="siftingAliceBases" class="p-2 font-mono"></td><td id="siftingBobBases" class="p-2 font-mono"></td></tr></tbody>
                    </table>
                </div>
                <div class="input-area border dark:border-gray-700 rounded-lg p-4">
                     <input type="text" id="siftedKeyInput" placeholder="을순이의 최종 비밀 키 예측" class="w-full bg-white dark:bg-gray-800 border border-gray-300 dark:border-gray-600 rounded-lg p-2">
                </div>
                 <p id="siftingError" class="feedback-message incorrect"></p>
                 <p id="siftingCorrect" class="feedback-message correct"></p>
                <button id="siftCheckButton" class="mt-4 w-full bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-6 rounded-lg transition duration-300">최종 키 확인</button>
            </section>

            <!-- Step 4: Final Results -->
            <section id="finalResultStep" class="step-section bg-white dark:bg-gray-800 shadow-md rounded-lg p-6 hidden">
                <h2 class="text-2xl font-semibold mb-4 border-b pb-2">최종 결과</h2>
                <div id="siftingResultsTable" class="overflow-x-auto"></div>
                <div class="mt-6 text-center">
                    <p class="font-semibold">갑돌이의 키: <span id="aliceKey" class="font-mono text-lg tracking-widest bg-green-100 dark:bg-green-900 p-2 rounded"></span></p>
                    <p class="font-semibold mt-2">을순이의 키: <span id="bobKey" class="font-mono text-lg tracking-widest bg-green-100 dark:bg-green-900 p-2 rounded"></span></p>
                </div>
                 <div id="eveCheck" class="mt-6 text-center">
                    <h3 class="text-xl font-semibold mb-2">도청자 확인</h3>
                    <div id="keyCheckResult" class="p-4 rounded-lg text-lg font-bold"></div>
                </div>
            </section>
        </div>
    </div>

    <script>
    // --- DOM Elements ---
    const eveToggle = document.getElementById('eveToggle');
    const resetButton = document.getElementById('resetButton');
    const allSections = document.querySelectorAll('.step-section');
    const mainContainer = document.getElementById('mainContainer');

    // Alice
    const aliceInputStep = document.getElementById('aliceInputStep');
    const aliceBitsInput = document.getElementById('aliceBitsInput');
    const aliceBasesInput = document.getElementById('aliceBasesInput');
    const aliceInputError = document.getElementById('aliceInputError');
    const alicePrepareButton = document.getElementById('alicePrepareButton');
    const alicePredictStep = document.getElementById('alicePredictStep');
    const aliceQubitPredictionArea = document.getElementById('aliceQubitPredictionArea');
    const alicePredictError = document.getElementById('alicePredictError');
    const alicePredictCorrect = document.getElementById('alicePredictCorrect');
    const aliceCheckButton = document.getElementById('aliceCheckButton');
    
    // Eve
    const eveInputStep = document.getElementById('eveInputStep');
    const eveBasesInput = document.getElementById('eveBasesInput');
    const eveInputError = document.getElementById('eveInputError');
    const evePrepareButton = document.getElementById('evePrepareButton');
    const evePredictQubitStep = document.getElementById('evePredictQubitStep');
    const eveQubitPredictionArea = document.getElementById('eveQubitPredictionArea');
    const eveQubitPredictError = document.getElementById('eveQubitPredictError');
    const eveQubitPredictCorrect = document.getElementById('eveQubitPredictCorrect');
    const eveQubitCheckButton = document.getElementById('eveQubitCheckButton');
    const evePredictBitStep = document.getElementById('evePredictBitStep');
    const eveBitPredictionArea = document.getElementById('eveBitPredictionArea');
    const eveBitPredictError = document.getElementById('eveBitPredictError');
    const eveBitPredictCorrect = document.getElementById('eveBitPredictCorrect');
    const eveBitCheckButton = document.getElementById('eveBitCheckButton');
    const eveResendStep = document.getElementById('eveResendStep');
    const eveQubitResendArea = document.getElementById('eveQubitResendArea');
    const eveResendError = document.getElementById('eveResendError');
    const eveResendCorrect = document.getElementById('eveResendCorrect');
    const eveResendCheckButton = document.getElementById('eveResendCheckButton');

    // Bob
    const bobInputStep = document.getElementById('bobInputStep');
    const bobBasesInput = document.getElementById('bobBasesInput');
    const bobInputError = document.getElementById('bobInputError');
    const bobPrepareButton = document.getElementById('bobPrepareButton');
    const bobPredictQubitStep = document.getElementById('bobPredictQubitStep');
    const bobQubitPredictionArea = document.getElementById('bobQubitPredictionArea');
    const bobQubitPredictError = document.getElementById('bobQubitPredictError');
    const bobQubitPredictCorrect = document.getElementById('bobQubitPredictCorrect');
    const bobQubitCheckButton = document.getElementById('bobQubitCheckButton');
    const bobPredictBitStep = document.getElementById('bobPredictBitStep');
    const bobBitPredictionArea = document.getElementById('bobBitPredictionArea');
    const bobBitPredictError = document.getElementById('bobBitPredictError');
    const bobBitPredictCorrect = document.getElementById('bobBitPredictCorrect');
    const bobBitCheckButton = document.getElementById('bobBitCheckButton');
    
    // Sifting
    const siftingStep = document.getElementById('siftingStep');
    const siftingAliceBases = document.getElementById('siftingAliceBases');
    const siftingBobBases = document.getElementById('siftingBobBases');
    const siftedKeyInput = document.getElementById('siftedKeyInput');
    const siftingError = document.getElementById('siftingError');
    const siftingCorrect = document.getElementById('siftingCorrect');
    const siftCheckButton = document.getElementById('siftCheckButton');

    // Final
    const finalResultStep = document.getElementById('finalResultStep');
    const siftingResultsTable = document.getElementById('siftingResultsTable');
    const aliceKeyEl = document.getElementById('aliceKey');
    const bobKeyEl = document.getElementById('bobKey');
    const keyCheckResultEl = document.getElementById('keyCheckResult');

    let state = {};
    const QUBIT_MAP = { '+': { '0': '↔', '1': '↕' }, 'x': { '0': '⤢', '1': '⤡' } };
    const REPLACEMENT_MAP = { '-': '↔', '|': '↕', '/': '⤢', '\\': '⤡' };

    // --- EVENT LISTENERS ---
    resetButton.addEventListener('click', resetSimulation);
    eveToggle.addEventListener('change', (event) => {
        if (event.target.checked) {
            mainContainer.classList.add('border-red-500', 'shadow-lg', 'shadow-red-500/20');
        } else {
            mainContainer.classList.remove('border-red-500', 'shadow-lg', 'shadow-red-500/20');
        }
    });

    [
        alicePrepareButton, aliceCheckButton,
        evePrepareButton, eveQubitCheckButton, eveBitCheckButton, eveResendCheckButton,
        bobPrepareButton, bobQubitCheckButton, bobBitCheckButton,
        siftCheckButton
    ].forEach(btn => btn.addEventListener('click', handleButtonClick));

    // --- MASTER HANDLER ---
    function handleButtonClick(event) {
        const id = event.target.id;
        // Alice
        if (id === 'alicePrepareButton') handleAlicePrepare();
        else if (id === 'aliceCheckButton') handleAliceCheck();
        // Eve
        else if (id === 'evePrepareButton') handleEvePrepare();
        else if (id === 'eveQubitCheckButton') handleEveQubitCheck();
        else if (id === 'eveBitCheckButton') handleEveBitCheck();
        else if (id === 'eveResendCheckButton') handleEveResendCheck();
        // Bob
        else if (id === 'bobPrepareButton') handleBobPrepare();
        else if (id === 'bobQubitCheckButton') handleBobQubitCheck();
        else if (id === 'bobBitCheckButton') handleBobBitCheck();
        // Sifting
        else if (id === 'siftCheckButton') handleSiftCheck();
    }

    // --- STEP HANDLERS ---
    
    // 1. Alice
    function handleAlicePrepare() {
        const bits = aliceBitsInput.value.trim();
        const bases = aliceBasesInput.value.trim().toLowerCase();
        const error = validateInput(bits, /^[01]+$/, '비트') || validateInput(bases, /^[+x]+$/, '기준');
        if (error || bits.length !== bases.length) {
            showFeedback(aliceInputError, error || '비트와 기준의 길이는 같아야 합니다.');
            return;
        }
        hideFeedback(aliceInputError);
        state = { aliceBits: bits, aliceBases: bases };
        state.sentQubits = encodeQubits(bits, bases);
        createPredictionInputs(aliceQubitPredictionArea, bits.length);
        showSection(alicePredictStep);
        disableSection(aliceInputStep);
    }
    
    function handleAliceCheck() {
        const predicted = getNormalizedPrediction(aliceQubitPredictionArea, REPLACEMENT_MAP).join('');
        if (predicted === state.sentQubits.join('')) {
            showSuccess(alicePredictCorrect, '정답입니다! 큐비트가 성공적으로 생성되었습니다.', () => {
                const nextStep = eveToggle.checked ? eveInputStep : bobInputStep;
                showSection(nextStep);
                disableSection(alicePredictStep);
            });
        } else {
            showError(alicePredictError, '예측이 틀렸습니다. 비트와 기준에 따른 광자 편광 방향을 다시 확인해보세요.', alicePredictCorrect);
        }
    }

    // 2. Eve
    function handleEvePrepare() {
        const bases = eveBasesInput.value.trim().toLowerCase();
        const error = validateInput(bases, /^[+x]+$/, '도청 기준');
         if (error || bases.length !== state.aliceBits.length) {
            showFeedback(eveInputError, error || `기준의 길이는 갑돌이가 보낸 비트의 길이(${state.aliceBits.length})와 같아야 합니다.`);
            return;
        }
        hideFeedback(eveInputError);
        state.eveBases = bases;
        state.eveBits = measureQubits(state.aliceBits, state.aliceBases, state.eveBases);
        state.eveQubitsAfterMeasurement = encodeQubits(state.eveBits, state.eveBases);
        createPredictionInputs(eveQubitPredictionArea, state.eveBases.length);
        showSection(evePredictQubitStep);
        disableSection(eveInputStep);
    }

    function handleEveQubitCheck() {
        const predictedInputs = Array.from(eveQubitPredictionArea.children);
        const predictedValues = getNormalizedPrediction(eveQubitPredictionArea, REPLACEMENT_MAP);
        const checkResult = checkPartialPrediction(predictedValues, state.eveQubitsAfterMeasurement, state.aliceBases, state.eveBases);
        if (checkResult.isCorrect) {
            fillAndDisableInputs(predictedInputs, state.eveQubitsAfterMeasurement);
            showSuccess(eveQubitPredictCorrect, '정답입니다! 도청 후 큐비트 상태가 확인되었습니다.', () => {
                 createPredictionInputs(eveBitPredictionArea, state.eveBases.length);
                 showSection(evePredictBitStep);
                 disableSection(evePredictQubitStep);
            });
        } else {
            showError(eveQubitPredictError, checkResult.errorMessage, eveQubitPredictCorrect);
        }
    }
    
    function handleEveBitCheck() {
        const predictedInputs = Array.from(eveBitPredictionArea.children);
        const checkResult = checkPartialPrediction(predictedInputs.map(i=>i.value), state.eveBits, state.aliceBases, state.eveBases);

        if (checkResult.isCorrect) {
            fillAndDisableInputs(predictedInputs, state.eveBits);
            showSuccess(eveBitPredictCorrect, '정답입니다! 도청으로 얻은 비트가 확인되었습니다.', () => {
                state.resentQubits = encodeQubits(state.eveBits, state.eveBases);
                createPredictionInputs(eveQubitResendArea, state.eveBases.length);
                showSection(eveResendStep);
                disableSection(evePredictBitStep);
            });
        } else {
            showError(eveBitPredictError, checkResult.errorMessage, eveBitPredictCorrect);
        }
    }
    
    function handleEveResendCheck() {
        const predictedQubitsNormalized = getNormalizedPrediction(eveQubitResendArea, REPLACEMENT_MAP).join('');
        if(predictedQubitsNormalized === state.resentQubits.join('')) {
            showSuccess(eveResendCorrect, '정답입니다! 을순이를 속이기 위한 큐비트를 전송했습니다.', () => {
                showSection(bobInputStep);
                disableSection(eveResendStep);
            });
        } else {
            showError(eveResendError, '예측이 틀렸습니다. 자신이 측정한 비트와 기준으로 다시 생성해보세요.', eveResendCorrect);
        }
    }

    // 3. Bob
    function handleBobPrepare() {
        const bases = bobBasesInput.value.trim().toLowerCase();
        const error = validateInput(bases, /^[+x]+$/, '기준');
        if (error || bases.length !== state.aliceBits.length) {
            showFeedback(bobInputError, error || `기준의 길이는 갑돌이가 보낸 비트의 길이(${state.aliceBits.length})와 같아야 합니다.`);
            return;
        }
        hideFeedback(bobInputError);
        state.bobBases = bases;
        
        const senderBases = state.eveBases || state.aliceBases;
        const senderBits = state.eveBits || state.aliceBits;
        
        state.bobBits = measureQubits(senderBits, senderBases, state.bobBases);
        state.bobQubitsAfterMeasurement = encodeQubits(state.bobBits, state.bobBases);

        createPredictionInputs(bobQubitPredictionArea, bases.length);
        showSection(bobPredictQubitStep);
        disableSection(bobInputStep);
    }

    function handleBobQubitCheck() {
        const predictedInputs = Array.from(bobQubitPredictionArea.children);
        const predictedNormalized = getNormalizedPrediction(bobQubitPredictionArea, REPLACEMENT_MAP);
        const checkResult = checkPartialPrediction(predictedNormalized, state.bobQubitsAfterMeasurement, state.aliceBases, state.bobBases);
        
        if (checkResult.isCorrect) {
            fillAndDisableInputs(predictedInputs, state.bobQubitsAfterMeasurement);
            showSuccess(bobQubitPredictCorrect, '정답입니다! 기준이 다른 곳의 큐비트 상태가 자동으로 채워졌습니다.', () => {
                createPredictionInputs(bobBitPredictionArea, state.bobBases.length);
                showSection(bobPredictBitStep);
                disableSection(bobPredictQubitStep);
            });
        } else {
            showError(bobQubitPredictError, checkResult.errorMessage, bobQubitPredictCorrect);
        }
    }

    function handleBobBitCheck() {
        const predictedInputs = Array.from(bobBitPredictionArea.children);
        const checkResult = checkPartialPrediction(predictedInputs.map(i=>i.value), state.bobBits, state.aliceBases, state.bobBases);

        if (checkResult.isCorrect) {
            fillAndDisableInputs(predictedInputs, state.bobBits);
            showSuccess(bobBitPredictCorrect, '정답입니다! 측정된 비트가 모두 확인되었습니다.', () => {
                siftingAliceBases.innerHTML = formatStringWithBases(state.aliceBases);
                siftingBobBases.innerHTML = formatStringWithBases(state.bobBases);
                showSection(siftingStep);
                disableSection(bobPredictBitStep);
            });
        } else {
            showError(bobBitPredictError, checkResult.errorMessage, bobBitPredictCorrect);
        }
    }

    // 4. Sifting
    function handleSiftCheck() {
        const { aliceKey, bobKey, matches } = siftKeys(state.aliceBits, state.bobBits, state.aliceBases, state.bobBases);
        state.finalAliceKey = aliceKey;
        state.finalBobKey = bobKey;
        state.matches = matches;
        if (siftedKeyInput.value.trim() === state.finalBobKey) {
             showSuccess(siftingCorrect, '정답입니다! 최종 비밀 키가 생성되었습니다.', () => {
                renderFinalResults();
                showSection(finalResultStep);
                disableSection(siftingStep);
            });
        } else {
             showError(siftingError, '예측이 틀렸습니다. 기준이 일치하는 위치에서 자신이 측정한 비트만 모았는지 다시 확인해보세요.', siftingCorrect);
        }
    }
    
    // --- UTILITY FUNCTIONS ---
    function resetSimulation() {
        state = {};
        allSections.forEach(sec => {
            sec.classList.add('hidden');
            sec.classList.remove('opacity-50');
            sec.querySelectorAll('input, button').forEach(item => {
                if(item.type === 'text') item.value = '';
                item.disabled = false
            });
            sec.querySelectorAll('.feedback-message').forEach(fb => hideFeedback(fb));
        });
        showSection(aliceInputStep);
        eveToggle.checked = false;
        mainContainer.classList.remove('border-red-500', 'shadow-lg', 'shadow-red-500/20');
    }

    function validateInput(value, regex, name) {
        if (!value) return `${name}을(를) 입력해야 합니다.`;
        if (!regex.test(value)) return `${name}의 형식이 올바르지 않습니다.`;
        return null;
    }

    function createPredictionInputs(area, length) {
        area.innerHTML = '';
        for (let i = 0; i < length; i++) {
            const input = document.createElement('input');
            input.type = 'text';
            input.maxLength = 1;
            input.className = 'qubit-input bg-white dark:bg-gray-800';
            area.appendChild(input);
        }
    }

    function getNormalizedPrediction(area, replacementMap) {
        return Array.from(area.children).map(input => {
            const char = input.value;
            return replacementMap[char] || char;
        });
    }

    function checkPartialPrediction(predictedValues, correctValues, base1, base2) {
        let result = { isCorrect: true, errorMessage: '' };
        
        for (let i = 0; i < correctValues.length; i++) {
            const basesMatch = base1[i] === base2[i];
            const predicted = predictedValues[i];
            const correct = correctValues[i];

            if (basesMatch) {
                if (predicted === '') { result = { isCorrect: false, errorMessage: `기준이 일치하는 ${i + 1}번째 칸을 입력해야 합니다.`}; break; }
                if (predicted !== correct) { result = { isCorrect: false, errorMessage: `기준이 일치하는 ${i + 1}번째 예측이 틀렸습니다.`}; break; }
            } else {
                if (predicted !== '' && predicted !== undefined) { result = { isCorrect: false, errorMessage: `기준이 다른 ${i + 1}번째 칸은 비워두어야 합니다.`}; break; }
            }
        }
        return result;
    }

    function fillAndDisableInputs(inputs, correctValues) {
        inputs.forEach((input, i) => {
            input.value = correctValues[i];
            input.disabled = true;
        });
    }

    function encodeQubits(bits, bases) {
        return bits.split('').map((bit, i) => QUBIT_MAP[bases[i]][bit]);
    }

    function generateRandomBases(length) {
        return Array.from({length}, () => ['+','x'][Math.floor(Math.random() * 2)]).join('');
    }
    
    function measureQubits(senderBits, senderBases, receiverBases) {
        return senderBases.split('').map((sBase, i) => {
             return (sBase === receiverBases[i]) ? senderBits[i] : String(Math.round(Math.random()));
        }).join('');
    }

    function siftKeys(aliceBits, bobBits, aliceBases, bobBases) {
        let aliceKey = '', bobKey = '', matches = [];
        for (let i = 0; i < aliceBits.length; i++) {
            if (aliceBases[i] === bobBases[i]) {
                aliceKey += aliceBits[i];
                bobKey += bobBits[i];
                matches.push(true);
            } else { matches.push(false); }
        }
        return { aliceKey, bobKey, matches };
    }

    // --- Rendering Functions ---
    function renderFinalResults() {
        let table = `<table class="w-full text-center border-collapse"><thead>
            <tr class="bg-gray-200 dark:bg-gray-700">
                <th class="p-2 border dark:border-gray-600">정보</th>
                ${state.aliceBits.split('').map((_, i) => `<th class="p-2 border dark:border-gray-600">${i+1}</th>`).join('')}
            </tr></thead><tbody>
            <tr><td class="font-semibold p-2 border dark:border-gray-600">갑돌이 비트</td>${state.aliceBits.split('').map(b => `<td class="p-2 border dark:border-gray-600 font-mono bit-${b}">${b}</td>`).join('')}</tr>
            <tr><td class="font-semibold p-2 border dark:border-gray-600">갑돌이 기준</td>${state.aliceBases.split('').map(b => `<td class="p-2 border dark:border-gray-600">${getBasisSpan(b)}</td>`).join('')}</tr>`;
        
        if (eveToggle.checked) {
            table += `<tr><td class="font-semibold p-2 border dark:border-gray-600 text-red-500">이브 기준</td>${state.eveBases.split('').map(b => `<td class="p-2 border dark:border-gray-600">${getBasisSpan(b)}</td>`).join('')}</tr>`;
            table += `<tr><td class="font-semibold p-2 border dark:border-gray-600 text-red-500">이브 측정</td>${state.eveBits.split('').map(b => `<td class="p-2 border dark:border-gray-600 font-mono bit-${b}">${b}</td>`).join('')}</tr>`;
        }

        table += `<tr><td class="font-semibold p-2 border dark:border-gray-600">을순이 기준</td>${state.bobBases.split('').map(b => `<td class="p-2 border dark:border-gray-600">${getBasisSpan(b)}</td>`).join('')}</tr>
            <tr><td class="font-semibold p-2 border dark:border-gray-600">을순이 측정</td>${state.bobBits.split('').map(b => `<td class="p-2 border dark:border-gray-600 font-mono bit-${b}">${b}</td>`).join('')}</tr>
            <tr class="bg-gray-100 dark:bg-gray-700"><td class="font-semibold p-2 border dark:border-gray-600">기준 일치</td>${state.matches.map(m => `<td class="p-2 border dark:border-gray-600 font-bold">${m ? '✅' : '❌'}</td>`).join('')}</tr>
            </tbody></table>`;

        siftingResultsTable.innerHTML = table;
        aliceKeyEl.textContent = state.finalAliceKey || '(없음)';
        bobKeyEl.textContent = state.finalBobKey || '(없음)';

        let resultClass = '', resultHTML = '';
        if (eveToggle.checked) {
            resultClass = state.finalAliceKey === state.finalBobKey ? 'bg-yellow-100 dark:bg-yellow-900 text-yellow-800 dark:text-yellow-200' : 'bg-red-200 dark:bg-red-900 text-red-800 dark:text-red-200';
            resultHTML = state.finalAliceKey === state.finalBobKey ? '⚠️ 키 일치! 이브가 운 좋게 모든 측정에 성공했습니다 (매우 드문 경우).' : '🚨 키 불일치! 도청이 감지되었습니다! 이 키는 폐기해야 합니다.';
        } else {
            resultClass = state.finalAliceKey ? 'bg-green-200 dark:bg-green-900 text-green-800 dark:text-green-200' : 'bg-gray-200 dark:bg-gray-700';
            resultHTML = state.finalAliceKey ? '✅ 키 일치! 도청이 없었으므로 안전한 키가 공유되었습니다.' : '키가 생성되지 않았습니다. 기준이 모두 달라 키를 만들 수 없습니다.';
        }
        keyCheckResultEl.className = `p-4 rounded-lg text-lg font-bold ${resultClass}`;
        keyCheckResultEl.innerHTML = resultHTML;
    }

    // --- UI Helpers ---
    function showSection(el) { el.classList.remove('hidden'); }
    function disableSection(el) {
        el.classList.add('opacity-50');
        el.querySelectorAll('input, button').forEach(item => item.disabled = true);
    }
    function showFeedback(el, message) { el.textContent = message; el.style.display = 'block'; }
    function hideFeedback(el) { el.style.display = 'none'; }
    function showSuccess(el, message, callback) {
        showFeedback(el, message);
        el.nextElementSibling.disabled = true; // Disable check button
        setTimeout(callback, 2000);
    }
    function showError(el, message, correctFeedbackEl) {
        showFeedback(el, message);
        hideFeedback(correctFeedbackEl);
    }
    function getBasisSpan(basis) {
        const className = basis.toLowerCase() === '+' ? 'basis-rect' : 'basis-diag';
        return `<span class="inline-block px-2 py-1 rounded-md ${className}">${basis}</span>`;
    }
    function formatStringWithBases(str) { return str.split('').map(char => getBasisSpan(char)).join(' '); }

    // --- Initial setup ---
    resetSimulation();
    </script>
</body>
</html>
