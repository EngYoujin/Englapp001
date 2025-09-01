<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DUOセレクト英単語クイズ (例文 21-30)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+JP:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Noto Sans JP', 'Inter', sans-serif; }
        .quiz-container { max-width: 800px; width: 95%; }
        .step-indicator.active { background-color: #3b82f6; color: white; font-weight: bold; }
        .btn-option:hover:not(:disabled) { transform: translateY(-2px); box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .flashcard { perspective: 1000px; cursor: pointer; }
        .flashcard-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .flashcard.flipped .flashcard-inner { transform: rotateY(180deg); }
        .flashcard-front, .flashcard-back { position: absolute; width: 100%; height: 100%; -webkit-backface-visibility: hidden; backface-visibility: hidden; display: flex; flex-direction: column; justify-content: center; align-items: center; padding: 1rem; border-radius: 0.75rem; border: 1px solid #d1d5db; }
        .flashcard-front { background-color: white; }
        .flashcard-back { background-color: #f0f9ff; transform: rotateY(180deg); }
        .speaker-btn { position: absolute; top: 10px; right: 10px; color: #6b7280; }
        .speaker-btn:hover { color: #3b82f6; }
        .tooltip-trigger { border-bottom: 2px dotted #3b82f6; cursor: pointer; position: relative; }
        .tooltip-content { visibility: hidden; width: max-content; max-width: 250px; background-color: #1f2937; color: #fff; text-align: center; border-radius: 6px; padding: 8px; position: absolute; z-index: 1; bottom: 125%; left: 50%; transform: translateX(-50%); opacity: 0; transition: opacity 0.3s; pointer-events: none; }
        .tooltip-trigger:hover .tooltip-content { visibility: visible; opacity: 1; }
        .toc-item.needs-review .review-mark { display: inline; }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="quiz-container mx-auto bg-white rounded-2xl shadow-lg p-6 md:p-8">
        <!-- Start Screen -->
        <div id="start-screen">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-800 text-center mb-4">DUOセレクト記憶定着クイズ</h1>
            <p class="text-gray-600 text-center mb-2">本での学習を「使える知識」に変えよう！</p>
            <p class="text-gray-500 text-center text-sm mb-8">新・4ステップ学習法で完璧マスター</p>
            <div class="text-center">
                <button id="start-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-8 rounded-full text-lg transition-transform transform hover:scale-105">
                    スタート (例文 21-30)
                </button>
            </div>
        </div>
        
        <!-- Table of Contents Screen -->
        <div id="toc-screen" class="hidden">
            <h2 class="text-2xl font-bold text-gray-800 text-center mb-6">目次 (例文 21-30)</h2>
            <p class="text-sm text-gray-500 text-center mb-4">挑戦したい問題を選ぶか、苦手な問題から復習しましょう。</p>
            <div id="toc-list" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4"></div>
             <div class="text-center mt-8">
                <button id="start-over-btn" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-6 rounded-full">最初から全ての問題をやる</button>
            </div>
        </div>

        <!-- Main Quiz Area -->
        <div id="main-quiz-area" class="hidden">
            <!-- Progress and Title -->
            <div class="flex justify-between items-center mb-2">
                <h2 id="quiz-set-title" class="text-xl font-bold text-blue-600"></h2>
                <p id="question-counter" class="text-gray-500 font-medium"></p>
            </div>
            <div class="w-full bg-gray-200 rounded-full h-2.5 mb-4">
                <div id="progress-bar" class="bg-blue-500 h-2.5 rounded-full" style="width: 0%; transition: width 0.3s ease-in-out;"></div>
            </div>
            <!-- Step Indicators -->
            <div class="flex justify-center space-x-1 md:space-x-2 mb-6 text-xs md:text-sm">
                <div id="step-indicator-1" class="step-indicator border border-gray-300 rounded-full px-3 py-1 text-gray-600 transition-all duration-300">1:クイズ</div>
                <div id="step-indicator-2" class="step-indicator border border-gray-300 rounded-full px-3 py-1 text-gray-600 transition-all duration-300">2:応用</div>
                <div id="step-indicator-3" class="step-indicator border border-gray-300 rounded-full px-3 py-1 text-gray-600 transition-all duration-300">3:復習</div>
                <div id="step-indicator-4" class="step-indicator border border-gray-300 rounded-full px-3 py-1 text-gray-600 transition-all duration-300 hidden">4:最終チェック</div>
            </div>

            <!-- Step Views -->
            <div id="step-views">
                <div id="step1-quiz" class="step-view"></div>
                <div id="step2-application" class="step-view hidden"></div>
                <div id="step3-flashcards" class="step-view hidden"></div>
                <div id="step4-final-check" class="step-view hidden"></div>
            </div>
            
            <!-- Navigation -->
            <div class="text-center mt-6">
                <button id="next-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-8 rounded-full transition-colors hidden">次へ</button>
            </div>
        </div>
        
        <!-- Result Screen -->
        <div id="result-screen" class="hidden text-center">
            <h2 class="text-3xl font-bold text-gray-800 mb-4">お疲れ様でした！</h2>
            <p class="text-lg text-gray-700 mb-6">セクション (例文 21-30) を完了しました。</p>
            <div id="self-assessment-summary"></div>
            <div class="mt-8">
                <button id="review-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-8 rounded-full text-lg transition-transform transform hover:scale-105 mr-4">苦手な問題を復習する</button>
            </div>
        </div>
    </div>

<script>
// --- DATA ---
const quizData = [
    {
        id: 21,
        originalSentence: "Take your time, and deal with the problems one by one.",
        translation: "慌てないで。一つひとつ問題に対処していきなさい。",
        quiz: { target: "deal with", choices: ["deal with", "get over", "look for", "turn down"], meaning: "～に対処する" },
        application: {
            situation: "山積みのタスクリストを見て、どこから手をつけていいか分からずパニックになっている同僚に。",
            sentence: "Don't <span class='tooltip-trigger'>panic<span class='tooltip-content'>panic = パニックになる</span></span>. Let's just handle these tasks systematically.",
            translation: "パニックにならないで。一つひとつ、体系的にこれらのタスクを片付けていこう。"
        },
        flashcards: [
            { en: "take one's time", ja: "ゆっくりやる、慌てない", kana: "**テイ**ク・ワンズ・**タ**イム", phonetic: "/teɪk wʌnz taɪm/", hint: "t" },
            { en: "one by one", ja: "一つひとつ", kana: "**ワ**ン・バイ・**ワ**ン", phonetic: "/wʌn baɪ wʌn/", hint: "o" }
        ]
    },
    {
        id: 22,
        originalSentence: "Take it easy. It's no use arguing about it.",
        translation: "いいじゃないか。そのことで口論したって無駄だよ。",
        quiz: { target: "It's no use arguing", choices: ["It's no use arguing", "It's worth trying", "It's fun learning", "It's hard saying"], meaning: "口論しても無駄だ" },
        application: {
            situation: "試合の結果に納得できず、ずっと文句を言っている友達に「もう終わったことだよ」と声をかける時。",
            sentence: "The game is over. It's no use arguing about the <span class='tooltip-trigger'>referee's call<span class='tooltip-content'>call = (審判などの)判定</span></span> now.",
            translation: "試合は終わったんだ。今さら審判の判定について口論したって無駄だよ。"
        },
        flashcards: [
            { en: "Take it easy.", ja: "気楽にね、落ち着いて", kana: "**テイ**ク・イット・**イー**ジー", phonetic: "/teɪk ɪt ˈiːzi/", hint: "T" },
            { en: "argue", ja: "口論する", kana: "**アー**ギュー", phonetic: "/ˈɑːrɡjuː/", hint: "a" }
        ]
    },
    {
        id: 23,
        originalSentence: "We were amazed that the device was tiny but smart.",
        translation: "その装置がとても小さいけれど賢いことにびっくりした。",
        quiz: { target: "amazed", choices: ["amazed", "bored", "frightened", "excited"], meaning: "（とても）驚いて" },
        application: {
            situation: "最新のワイヤレスイヤホンを買ったら、あまりの音質の良さと機能の多さに感動した。",
            sentence: "I was honestly amazed by how good these new <span class='tooltip-trigger'>earbuds<span class='tooltip-content'>earbuds = (耳に入れるタイプの)イヤホン</span></span> sound. The noise-canceling is incredible.",
            translation: "この新しいイヤホンの音の良さには正直、本当に驚いたよ。ノイズキャンセリングが信じられないくらいすごい。"
        },
        flashcards: [
            { en: "device", ja: "装置", kana: "ディ**ヴァ**イス", phonetic: "/dɪˈvaɪs/", hint: "d" },
            { en: "tiny", ja: "とても小さい", kana: "**タ**イニー", phonetic: "/ˈtaɪni/", hint: "t" },
            { en: "smart", ja: "賢い", kana: "ス**マー**ト", phonetic: "/smɑːrt/", hint: "s" }
        ]
    },
    {
        id: 24,
        originalSentence: "Although Bob is frightened of insects, he studies biology.",
        translation: "ボブは昆虫恐怖症だが、生物学を勉強している。",
        quiz: { target: "frightened", choices: ["frightened", "amazed", "bored", "ashamed"], meaning: "おびえて、怖がって" },
        application: {
            situation: "ホラー映画は苦手だけど、なぜか見てしまう友達。",
            sentence: "It's funny, she's frightened of <span class='tooltip-trigger'>ghosts<span class='tooltip-content'>ghost = 幽霊</span></span>, but she loves watching horror movies.",
            translation: "面白いよね、彼女、お化けを怖がるのにホラー映画を見るのは大好きなんだ。"
        },
        flashcards: [
            { en: "although", ja: "～だけれども", kana: "オール**ゾ**ウ", phonetic: "/ɔːlˈðoʊ/", hint: "a" },
            { en: "insect", ja: "昆虫", kana: "**イ**ンセクト", phonetic: "/ˈɪnsekt/", hint: "i" },
            { en: "biology", ja: "生物学", kana: "バイ**オ**ロジー", phonetic: "/baɪˈɑːlədʒi/", hint: "b" }
        ]
    },
    {
        id: 25,
        originalSentence: "No wonder the audience was bored to death. The topic was boring.",
        translation: "聴衆が死ぬほど退屈したのも当然だ。話題が退屈だったのだ。",
        quiz: { target: "No wonder", choices: ["No wonder", "No problem", "No way", "No doubt"], meaning: "～なのも当然だ" },
        application: {
            situation: "いつも行列ができているラーメン屋さん。実際に食べてみたら、ものすごく美味しかった。",
            sentence: "This ramen is incredible! No wonder there's always a <span class='tooltip-trigger'>line out the door<span class='tooltip-content'>line out the door = ドアの外まで続く行列</span></span>.",
            translation: "このラーメン、信じられないくらい美味しい！いつも行列ができているのも当然だね。"
        },
        flashcards: [
            { en: "audience", ja: "聴衆", kana: "**オー**ディエンス", phonetic: "/ˈɔːdiəns/", hint: "a" },
            { en: "bored", ja: "（人が）退屈して", kana: "**ボー**ド", phonetic: "/bɔːrd/", hint: "b" },
            { en: "topic", ja: "話題", kana: "**ト**ピック", phonetic: "/ˈtɑːpɪk/", hint: "t" }
        ]
    },
    {
        id: 26,
        originalSentence: "Now that you've gotten married, respect each other.",
        translation: "あなたたちは結婚したのだから、お互いを尊重し合いなさい。",
        quiz: { target: "Now that", choices: ["Now that", "Even if", "As long as", "Just because"], meaning: "今や～なので" },
        application: {
            situation: "運転免許を取ったばかりの友達に、ドライブに連れて行ってもらう約束をする。",
            sentence: "Now that you have your <span class='tooltip-trigger'>driver's license<span class='tooltip-content'>driver's license = 運転免許証</span></span>, you have to take me to the beach!",
            translation: "もう運転免許を取ったんだから、私をビーチに連れて行ってくれなきゃ！"
        },
        flashcards: [
            { en: "get married", ja: "結婚する", kana: "**ゲ**ット・**マ**リード", phonetic: "/ɡet ˈmærid/", hint: "g" },
            { en: "respect", ja: "～を尊重する", kana: "リ**スペ**クト", phonetic: "/rɪˈspekt/", hint: "r" },
            { en: "each other", ja: "お互い", kana: "**イー**チ・**ア**ザー", phonetic: "/iːtʃ ˈʌðər/", hint: "e" }
        ]
    },
    {
        id: 27,
        originalSentence: "I got lost. To make matters worse, my car broke down.",
        translation: "道に迷ってしまった。さらに悪いことに、車が故障した。",
        quiz: { target: "broke down", choices: ["broke down", "gave up", "turned off", "got lost"], meaning: "故障した" },
        application: {
            situation: "大事なプレゼンの日に寝坊。急いで準備をしていたら、さらに悪いことが…。",
            sentence: "I <span class='tooltip-trigger'>overslept<span class='tooltip-content'>oversleep = 寝坊する</span></span> on the day of my big presentation. To make matters worse, my laptop wouldn't turn on.",
            translation: "大事なプレゼンの日に寝坊しちゃったんだ。さらに悪いことに、ノートパソコンが起動しなかったんだ。"
        },
        flashcards: [
            { en: "get lost", ja: "道に迷う", kana: "**ゲ**ット・**ロ**スト", phonetic: "/ɡet lɔːst/", hint: "g" },
            { en: "to make matters worse", ja: "さらに悪いことに", kana: "トゥ・**メイ**ク・**マ**ターズ・**ワー**ス", phonetic: "/tu meɪk ˈmætərz wɜːrs/", hint: "t" }
        ]
    },
    {
        id: 28,
        originalSentence: "I'm excited about getting together.",
        translation: "みんなに会えるから、わくわくしているよ。",
        quiz: { target: "excited", choices: ["excited", "amazed", "bored", "frightened"], meaning: "わくわくして" },
        application: {
            situation: "好きなアーティストのライブが来週に迫っている。",
            sentence: "The concert is next week! I'm so excited, I can <span class='tooltip-trigger'>hardly wait<span class='tooltip-content'>hardly wait = 待ちきれない</span></span>.",
            translation: "ライブは来週だ！すっごくわくわくして、待ちきれないよ。"
        },
        flashcards: [
            { en: "get together", ja: "集まる", kana: "**ゲ**ット・トゥ**ゲ**ザー", phonetic: "/ɡet təˈɡeðər/", hint: "g" },
            { en: "So am I.", ja: "私もです。", kana: "**ソ**ウ・アム・**ア**イ", phonetic: "/soʊ æm aɪ/", hint: "S" }
        ]
    },
    {
        id: 29,
        originalSentence: "The negotiations have failed, so the tension will get worse.",
        translation: "交渉が失敗に終わったので、緊張は悪化するだろう。",
        quiz: { target: "tension", choices: ["tension", "attention", "solution", "tradition"], meaning: "緊張（状態）" },
        application: {
            situation: "グループプロジェクトで意見が対立。締め切りが近づくにつれて、メンバー間の雰囲気が悪くなってきた。",
            sentence: "You could feel the tension in the room as the project <span class='tooltip-trigger'>deadline<span class='tooltip-content'>deadline = 締め切り</span></span> got closer.",
            translation: "プロジェクトの締め切りが近づくにつれて、部屋の緊張感が感じられたよ。"
        },
        flashcards: [
            { en: "negotiation", ja: "交渉", kana: "ネゴシ**エイ**ション", phonetic: "/nəˌɡoʊʃiˈeɪʃn/", hint: "n" },
            { en: "fail", ja: "失敗する", kana: "**フェ**イル", phonetic: "/feɪl/", hint: "f" },
            { en: "get worse", ja: "悪化する", kana: "**ゲ**ット・**ワー**ス", phonetic: "/ɡet wɜːrs/", hint: "g" }
        ]
    },
    {
        id: 30,
        originalSentence: "Our hostility toward him grew more and more intense.",
        translation: "彼に対する私たちの敵意は、ますます激しくなった。",
        quiz: { target: "intense", choices: ["intense", "solid", "vague", "rare"], meaning: "激しい" },
        application: {
            situation: "応援しているスポーツチームが、ライバルチームと接戦を繰り広げている。",
            sentence: "The <span class='tooltip-trigger'>rivalry<span class='tooltip-content'>rivalry = ライバル関係</span></span> between the two teams is so intense. Every match is a <span class='tooltip-trigger'>must-win<span class='tooltip-content'>must-win = 絶対に勝たなければならない試合</span></span>.",
            translation: "あの2チームのライバル関係はすごく激しい。どの試合も絶対に負けられないんだ。"
        },
        flashcards: [
            { en: "hostility", ja: "敵意", kana: "ホス**ティ**リティ", phonetic: "/hɑːˈstɪləti/", hint: "h" },
            { en: "grow", ja: "～になる；育つ", kana: "グ**ロ**ウ", phonetic: "/ɡroʊ/", hint: "g" },
            { en: "more and more", ja: "ますます", kana: "**モー**・アンド・**モー**", phonetic: "/mɔːr ænd mɔːr/", hint: "m" }
        ]
    }
];

// --- STATE ---
let currentSentenceIndex = 0;
let currentStep = 1;
let selfAssessments = new Array(quizData.length).fill(null);
let quizResults = new Array(quizData.length).fill(null);
let isFirstRun = true;
let speechSynthesis = window.speechSynthesis;
let englishVoice = null;
let quizQueue = [];

// --- DOM ELEMENTS ---
const startScreen = document.getElementById('start-screen');
const tocScreen = document.getElementById('toc-screen');
const mainQuizArea = document.getElementById('main-quiz-area');
const resultScreen = document.getElementById('result-screen');
const startBtn = document.getElementById('start-btn');
const nextBtn = document.getElementById('next-btn');
const reviewBtn = document.getElementById('review-btn');
const startOverBtn = document.getElementById('start-over-btn');

const stepViews = {
    1: document.getElementById('step1-quiz'),
    2: document.getElementById('step2-application'),
    3: document.getElementById('step3-flashcards'),
    4: document.getElementById('step4-final-check')
};
const stepIndicators = {
    1: document.getElementById('step-indicator-1'),
    2: document.getElementById('step-indicator-2'),
    3: document.getElementById('step-indicator-3'),
    4: document.getElementById('step-indicator-4')
};

// --- EVENT LISTENERS ---
startBtn.addEventListener('click', () => {
    isFirstRun = true;
    quizResults.fill(null);
    selfAssessments.fill(null);
    startQuizSession(Array.from(Array(quizData.length).keys()));
});
nextBtn.addEventListener('click', handleNext);
reviewBtn.addEventListener('click', showTableOfContents);
startOverBtn.addEventListener('click', () => {
    isFirstRun = false;
    startQuizSession(Array.from(Array(quizData.length).keys()));
});


// --- Voice Initialization Logic ---
function loadVoices() {
    const voices = speechSynthesis.getVoices();
    englishVoice = voices.find(voice => voice.lang.startsWith('en-US')) || voices.find(voice => voice.lang.startsWith('en-'));
}
speechSynthesis.onvoiceschanged = loadVoices;
loadVoices();

// --- FUNCTIONS ---

function speak(text, lang = 'en-US') {
    if (speechSynthesis.speaking) speechSynthesis.cancel();
    const utterance = new SpeechSynthesisUtterance(text);
    if (englishVoice) utterance.voice = englishVoice;
    utterance.lang = lang;
    utterance.rate = 0.9;
    speechSynthesis.speak(utterance);
}

function startQuizSession(indices) {
    quizQueue = indices;
    if (quizQueue.length === 0) {
        showTableOfContents();
        return;
    }
    currentSentenceIndex = quizQueue.shift();
    currentStep = 1;
    startScreen.classList.add('hidden');
    tocScreen.classList.add('hidden');
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.remove('hidden');
    nextBtn.classList.add('hidden');
    renderCurrentStep();
}

function handleNext() {
    if (currentStep === 4) {
        showResult();
        return;
    }

    currentStep++;

    if (currentStep > 3) {
        if (quizQueue.length > 0) {
            currentSentenceIndex = quizQueue.shift();
            currentStep = 1;
            renderCurrentStep();
        } else {
            if (isFirstRun) {
                currentStep = 4;
                renderCurrentStep();
            } else {
                showTableOfContents();
            }
        }
    } else {
        renderCurrentStep();
    }
}


function updateProgress() {
    const currentData = quizData[currentSentenceIndex];
    if (!currentData) return; 

    document.getElementById('question-counter').textContent = `例文 ${currentData.id}`;
    
    const progress = isFirstRun ? ((currentSentenceIndex) / quizData.length) * 100 : 0;
    document.getElementById('progress-bar').style.width = `${progress}%`;

    Object.values(stepIndicators).forEach(el => el.classList.remove('active'));
    stepIndicators[4].classList.toggle('hidden', isFirstRun === false);
    if (stepIndicators[currentStep]) {
        stepIndicators[currentStep].classList.add('active');
    }
}

function renderCurrentStep() {
    updateProgress();
    Object.values(stepViews).forEach(view => view.classList.add('hidden'));
    stepViews[currentStep].classList.remove('hidden');
    nextBtn.classList.add('hidden');

    if (currentStep === 4) {
        renderStep4();
        return;
    }

    const data = quizData[currentSentenceIndex];
    switch (currentStep) {
        case 1: renderStep1(data); break;
        case 2: renderStep2(data); break;
        case 3: renderStep3(data); break;
    }
}

// --- RENDER FUNCTIONS FOR EACH STEP ---

function renderStep1(data) {
    const { quiz, originalSentence } = data;
    const questionText = originalSentence.replace(new RegExp(quiz.target, 'i'), '______');
    const shuffledChoices = [...quiz.choices].sort(() => Math.random() - 0.5);
    
    let optionsHTML = shuffledChoices.map(choice => 
        `<button class="btn-option w-full p-4 bg-white border border-gray-300 rounded-lg text-left text-gray-700 font-medium transition-all duration-200">${choice}</button>`
    ).join('');

    stepViews[1].innerHTML = `
        <p class="text-sm text-gray-500 text-center mb-4">あてはまる選択肢を選びましょう。</p>
        <div class="mb-6 relative">
            <p class="text-xl md:text-2xl text-gray-800 text-center p-4 bg-gray-50 rounded-lg">${questionText}</p>
            <button class="speaker-btn text-xl" onclick="speak('${originalSentence.replace(/'/g, "\\'")}')"><i class="fas fa-volume-up"></i></button>
        </div>
        <div id="step1-options" class="grid grid-cols-1 md:grid-cols-2 gap-4">${optionsHTML}</div>
        <div id="step1-feedback" class="p-4 rounded-lg text-center font-medium hidden my-4"></div>
    `;

    document.getElementById('step1-options').addEventListener('click', e => {
        if (e.target.tagName === 'BUTTON') {
            handleQuizAnswer(e.target, quiz, data.translation);
        }
    });
}

function handleQuizAnswer(selectedButton, quizInfo, translation) {
    const isCorrect = selectedButton.innerText.toLowerCase() === quizInfo.target.toLowerCase();
    quizResults[currentSentenceIndex] = isCorrect;
    
    const feedbackEl = document.getElementById('step1-feedback');
    
    if (isCorrect) {
        feedbackEl.innerHTML = `正解！ <span class="font-bold">${quizInfo.target}</span> は「${quizInfo.meaning}」という意味です。`;
        feedbackEl.className = 'p-4 rounded-lg text-center font-medium my-4 bg-green-100 border border-green-300 text-green-800';
    } else {
        feedbackEl.innerHTML = `不正解。正解は <span class="font-bold">${quizInfo.target}</span> です。<br>「${quizInfo.meaning}」という意味になります。`;
        feedbackEl.className = 'p-4 rounded-lg text-center font-medium my-4 bg-red-100 border border-red-300 text-red-800';
    }
    feedbackEl.classList.remove('hidden');
    
    const translationEl = document.createElement('p');
    translationEl.className = 'text-center text-gray-600 mt-2';
    translationEl.textContent = `文全体の訳：${translation}`;
    feedbackEl.appendChild(translationEl);

    Array.from(document.getElementById('step1-options').children).forEach(button => {
        button.disabled = true;
        if (button.innerText.toLowerCase() === quizInfo.target.toLowerCase()) button.classList.add('bg-green-200', 'border-green-500');
        else if (button === selectedButton) button.classList.add('bg-red-200', 'border-red-500');
    });

    nextBtn.textContent = '応用例文へ →';
    nextBtn.classList.remove('hidden');
}

function renderStep2(data) {
    const { application } = data;
    stepViews[2].innerHTML = `
        <p class="text-sm text-gray-500 text-center mb-4">応用例文で使い方を確認しましょう。</p>
        <div class="bg-sky-50 border-l-4 border-sky-500 p-4 rounded-lg mb-4">
            <p class="font-semibold text-sky-800 mb-2">💡 こんな場面で使える！</p>
            <p class="text-gray-700">${application.situation}</p>
        </div>
        <div class="mb-4 relative p-4 bg-gray-50 rounded-lg">
            <p class="text-lg md:text-xl text-gray-800">${application.sentence}</p>
            <button class="speaker-btn text-xl" onclick="speak(this.previousElementSibling.innerText.replace(/'/g, &quot;\\'&quot;))"><i class="fas fa-volume-up"></i></button>
        </div>
        <div class="text-center mb-6">
            <button id="show-translation-btn" class="text-blue-600 hover:underline">日本語訳を見る</button>
        </div>
        <p id="translation-text" class="text-center text-gray-600 hidden mb-6">${application.translation}</p>
        <div id="self-assessment-buttons" class="flex justify-center space-x-4">
            <button data-cleared="false" class="self-assess-btn border border-gray-400 text-gray-700 font-bold py-2 px-6 rounded-full hover:bg-gray-100">もう一度</button>
            <button data-cleared="true" class="self-assess-btn bg-green-500 text-white font-bold py-2 px-6 rounded-full hover:bg-green-600">理解できた！</button>
        </div>
    `;

    document.getElementById('show-translation-btn').addEventListener('click', e => {
        document.getElementById('translation-text').classList.toggle('hidden');
        e.target.textContent = document.getElementById('translation-text').classList.contains('hidden') ? '日本語訳を見る' : '日本語訳を隠す';
    });

    document.getElementById('self-assessment-buttons').addEventListener('click', e => {
        if (e.target.classList.contains('self-assess-btn')) {
            const cleared = e.target.dataset.cleared === 'true';
            selfAssessments[currentSentenceIndex] = cleared;
            
            if (!isFirstRun && cleared) {
                if(quizResults[currentSentenceIndex] === true) {
                    // This logic is now handled correctly by the `needsReview` check in `showTableOfContents`
                }
            }

            nextBtn.textContent = '単語の復習へ →';
            nextBtn.classList.remove('hidden');
            document.querySelectorAll('.self-assess-btn').forEach(btn => {
                btn.disabled = true;
                btn.classList.add('opacity-50');
            });
            e.target.classList.remove('opacity-50');
        }
    });
}

function renderFlashcards(container, cards, defaultMode) {
    container.innerHTML = '';
    cards.forEach(card => {
        const cardEl = document.createElement('div');
        cardEl.className = 'flashcard h-48';
        
        const frontContent = (defaultMode === 'en-ja') 
            ? `<h3 class="text-2xl font-bold">${card.en}</h3>`
            : `<h3 class="text-xl font-bold">${card.ja}</h3><p class="text-gray-500 mt-2">ヒント: ${card.hint}</p>`;
        
        const backContent = (defaultMode === 'en-ja')
            ? `<h3 class="text-xl font-bold">${card.ja}</h3>`
            : `<h3 class="text-2xl font-bold">${card.en}</h3>`;

        cardEl.innerHTML = `
            <div class="flashcard-inner">
                <div class="flashcard-front">${frontContent}<button class="speaker-btn" onclick="event.stopPropagation(); speak('${card.en.replace(/'/g, "\\'")}')"><i class="fas fa-volume-up"></i></button></div>
                <div class="flashcard-back">
                    ${backContent}
                    <div class="text-center mt-2">
                        <p class="text-gray-500">${card.kana.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>')}</p>
                        <p class="text-gray-400 text-sm">${card.phonetic}</p>
                    </div>
                    <button class="speaker-btn" onclick="event.stopPropagation(); speak('${card.en.replace(/'/g, "\\'")}')"><i class="fas fa-volume-up"></i></button>
                </div>
            </div>
        `;
        cardEl.addEventListener('click', () => cardEl.classList.toggle('flipped'));
        container.appendChild(cardEl);
    });
}

function renderStep3(data) {
    const { flashcards } = data;
    stepViews[3].innerHTML = `
        <p class="text-sm text-gray-500 text-center mb-4">カードをタップして関連単語も覚えましょう。</p>
        <div class="flex justify-center mb-4">
            <div class="inline-flex rounded-md shadow-sm" role="group">
                <button type="button" id="mode-en-ja-step3" class="mode-btn bg-blue-500 text-white px-4 py-2 text-sm font-medium border border-gray-200 rounded-l-lg">認識 (英→日)</button>
                <button type="button" id="mode-ja-en-step3" class="mode-btn bg-white text-gray-900 px-4 py-2 text-sm font-medium border border-gray-200 rounded-r-lg hover:bg-gray-100">想起 (日→英)</button>
            </div>
        </div>
        <div id="flashcard-container-step3" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
    `;

    const container = document.getElementById('flashcard-container-step3');
    renderFlashcards(container, flashcards, 'en-ja');

    const modeBtnEnJa = document.getElementById('mode-en-ja-step3');
    const modeBtnJaEn = document.getElementById('mode-ja-en-step3');
    
    modeBtnEnJa.addEventListener('click', () => {
        modeBtnEnJa.classList.add('bg-blue-500', 'text-white');
        modeBtnJaEn.classList.remove('bg-blue-500', 'text-white');
        renderFlashcards(container, flashcards, 'en-ja');
    });
    
    modeBtnJaEn.addEventListener('click', () => {
        modeBtnJaEn.classList.add('bg-blue-500', 'text-white');
        modeBtnEnJa.classList.remove('bg-blue-500', 'text-white');
        renderFlashcards(container, flashcards, 'ja-en');
    });

    const nextText = isFirstRun ? 
        (quizQueue.length === 0 ? '最終チェックへ →' : '次の問題へ →') :
        (quizQueue.length === 0 ? '目次に戻る' : '次の問題へ →');
    nextBtn.textContent = nextText;
    nextBtn.classList.remove('hidden');
}

function renderStep4() {
    let allFlashcards = quizData.flatMap(d => d.flashcards).sort(() => Math.random() - 0.5);

    stepViews[4].innerHTML = `
        <p class="text-sm text-gray-500 text-center mb-4">セクションの総復習！日本語から英語を思い出してみましょう。</p>
        <div class="flex justify-center mb-4">
            <div class="inline-flex rounded-md shadow-sm" role="group">
                <button type="button" id="mode-en-ja-step4" class="mode-btn bg-white text-gray-900 px-4 py-2 text-sm font-medium border border-gray-200 rounded-l-lg hover:bg-gray-100">認識 (英→日)</button>
                <button type="button" id="mode-ja-en-step4" class="mode-btn bg-blue-500 text-white px-4 py-2 text-sm font-medium border border-gray-200 rounded-r-lg">想起 (日→英)</button>
            </div>
        </div>
        <div id="flashcard-container-step4" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
    `;

    const container = document.getElementById('flashcard-container-step4');
    renderFlashcards(container, allFlashcards, 'ja-en');

    const modeBtnEnJa = document.getElementById('mode-en-ja-step4');
    const modeBtnJaEn = document.getElementById('mode-ja-en-step4');

    modeBtnEnJa.addEventListener('click', () => {
        modeBtnEnJa.classList.add('bg-blue-500', 'text-white');
        modeBtnJaEn.classList.remove('bg-blue-500', 'text-white');
        renderFlashcards(container, allFlashcards, 'en-ja');
    });
    
    modeBtnJaEn.addEventListener('click', () => {
        modeBtnJaEn.classList.add('bg-blue-500', 'text-white');
        modeBtnEnJa.classList.remove('bg-blue-500', 'text-white');
        renderFlashcards(container, allFlashcards, 'ja-en');
    });

    nextBtn.textContent = '結果を見る';
    nextBtn.classList.remove('hidden');
}

function showResult() {
    mainQuizArea.classList.add('hidden');
    resultScreen.classList.remove('hidden');
    
    const clearedCount = selfAssessments.filter(sa => sa === true).length;
    const totalAssessed = selfAssessments.filter(sa => sa !== null).length;
    
    const summaryEl = document.getElementById('self-assessment-summary');
    summaryEl.innerHTML = `
        <p class="text-xl">応用例文の理解度: <span class="font-bold text-green-600">${clearedCount}</span> / ${totalAssessed}</p>
        <p class="text-gray-600 mt-2">「もう一度」を選んだ問題やクイズで間違えた問題は、復習リストに追加されています。</p>
    `;
    isFirstRun = false;
}

function showTableOfContents() {
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.add('hidden');
    startScreen.classList.add('hidden');
    tocScreen.classList.remove('hidden');
    const tocList = document.getElementById('toc-list');
    tocList.innerHTML = '';
    quizData.forEach((data, index) => {
        const needsReview = quizResults[index] === false || selfAssessments[index] === false;
        const item = document.createElement('button');
        item.className = 'toc-item p-4 border rounded-lg text-center hover:bg-gray-100 transition-colors';
        if (needsReview) item.classList.add('needs-review', 'border-red-400', 'bg-red-50');
        
        item.innerHTML = `
            例文 ${data.id}
            <span class="review-mark text-red-500 ${needsReview ? '' : 'hidden'}">🔴</span>
        `;
        item.onclick = () => {
            isFirstRun = false;
            startQuizSession([index]);
        };
        tocList.appendChild(item);
    });
}
</script>
</body>
</html>
