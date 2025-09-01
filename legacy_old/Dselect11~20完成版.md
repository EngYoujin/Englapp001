<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>単語帳学習アプリ</title>
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
        .review-mark {
            display: inline-block;
            font-size: 0.7rem;
            font-weight: bold;
            color: white;
            background-color: #ef4444;
            border-radius: 9999px;
            width: 1.1rem;
            height: 1.1rem;
            line-height: 1.1rem;
            text-align: center;
            margin-left: 0.25rem;
        }
        .toc-item-content {
            display: flex;
            align-items: center;
            justify-content: center;
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen p-4">

    <div id="app-container" class="quiz-container mx-auto bg-white rounded-2xl shadow-lg p-6 md:p-8">
        
        <!-- Table of Contents Screen (Main Menu) -->
        <div id="toc-screen">
            <h1 class="text-3xl md:text-4xl font-bold text-gray-800 text-center mb-4">DUOセレクト学習</h1>
            <p class="text-gray-600 text-center mb-8">学習したい例文を選ぶか、最初から全ての問題に挑戦しましょう。</p>
            
            <div class="border-b pb-8 mb-8">
                <h2 class="text-lg font-bold text-gray-700 text-center mb-4">メイン学習</h2>
                <button id="start-sequential-btn" class="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-8 rounded-full text-lg transition-transform transform hover:scale-105">
                    最初から順番に全ての問題をやる
                </button>
            </div>

            <div>
                <h2 class="text-lg font-bold text-gray-700 text-center mb-4">個別学習・復習</h2>
                <div id="toc-list" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4"></div>
            </div>
            
            <div class="border-t pt-8 mt-8">
                 <h2 class="text-lg font-bold text-gray-700 text-center mb-4">腕試し</h2>
                 <button id="final-check-btn" class="w-full bg-white hover:bg-gray-100 text-gray-700 font-semibold py-2 px-6 border border-gray-300 rounded-full">
                    まとめて最終チェック (日→英)
                </button>
            </div>
        </div>

        <!-- Main Quiz Area -->
        <div id="main-quiz-area" class="hidden">
            <!-- Progress and Title -->
            <div class="flex justify-between items-center mb-2">
                 <button id="back-to-toc-btn" class="text-blue-500 hover:text-blue-700 font-semibold"><i class="fas fa-arrow-left mr-2"></i>目次に戻る</button>
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
            </div>

            <!-- Step Views -->
            <div id="step-views">
                <div id="step1-quiz" class="step-view"></div>
                <div id="step2-application" class="step-view hidden"></div>
                <div id="step3-flashcards" class="step-view hidden"></div>
            </div>
            
            <!-- Navigation -->
            <div class="text-center mt-6">
                <button id="next-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-8 rounded-full transition-colors hidden">次へ</button>
            </div>
        </div>
        
        <!-- Final Check Area -->
        <div id="final-check-area" class="hidden">
            <div class="flex justify-between items-center mb-4">
                 <button id="back-to-toc-from-final-btn" class="text-blue-500 hover:text-blue-700 font-semibold"><i class="fas fa-arrow-left mr-2"></i>目次に戻る</button>
            </div>
            <h2 class="text-2xl font-bold text-gray-800 text-center mb-4">まとめて最終チェック</h2>
            <p class="text-sm text-gray-500 text-center mb-6">日本語を見て、対応する英単語を思い出してみましょう。カードをタップすると答えが表示されます。</p>
            <div id="final-check-flashcards" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
        </div>

        <!-- Result Screen -->
        <div id="result-screen" class="hidden text-center">
            <h2 class="text-3xl font-bold text-gray-800 mb-4">全問終了！お疲れ様でした！</h2>
            <p class="text-lg text-gray-700 mb-6">学習結果を確認しましょう。</p>
            <div id="self-assessment-summary"></div>
            <div class="mt-8">
                <button id="review-btn" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-8 rounded-full text-lg transition-transform transform hover:scale-105 mr-4">目次で復習する</button>
            </div>
        </div>
    </div>

<script>
// --- LEARNING APP LOGIC ---
const quizData = [
    {
        "id": 11,
        "originalSentence": "Fame is a means to an end, not an end itself.",
        "translation": "名声は目的達成の手段であって、それ自体が目的ではない。",
        "quiz": { "target": "means", "choices": ["means", "end", "money", "way"], "meaning": "手段" },
        "application": {
            "situation": "「なんでそんなに必死に勉強するの？」と聞かれた時に、自分の目標を語る場面。",
            "sentence": "For me, getting into a good university isn't the <span class='tooltip-trigger'>end goal<span class='tooltip-content'>end goal = 最終目標</span></span>; it's just a means to becoming a great researcher.",
            "translation": "僕にとって、良い大学に入ることは最終目標じゃないんだ。偉大な研究者になるための、ただの手段なんだよ。"
        },
        "flashcards": [
            { "en": "make money", "ja": "お金を稼ぐ", "kana": "**メイ**ク・**マ**ネー", "phonetic": "/meɪk ˈmʌni/", "hint": "m" },
            { "en": "end", "ja": "目的；終わり", "kana": "**エ**ンド", "phonetic": "/end/", "hint": "e" },
            { "en": "in itself", "ja": "それ自体", "kana": "イン・イット**セ**ルフ", "phonetic": "/ɪn ɪtˈself/", "hint": "i" },
            { "en": "means", "ja": "手段", "kana": "ミーンズ", "phonetic": "/miːnz/", "hint": "m" }
        ]
    },
    {
        "id": 12,
        "originalSentence": "I can pick you up from the station on my way home.",
        "translation": "家に帰る途中で、駅まであなたを迎えに行けますよ。",
        "quiz": { "target": "pick you up", "choices": ["pick you up", "take you up", "look you up", "get you up"], "meaning": "（車で）君を拾う" },
        "application": {
            "situation": "友達と駅で待ち合わせ。少し早く着きそうなので、一つ手前の駅にいる友達に連絡する。",
            "sentence": "Hey, I'm almost at your station. I can pick you up if you want.",
            "translation": "もしもし、もうすぐ君の駅に着くよ。よかったら車で拾ってこうか？"
        },
        "flashcards": [
            { "en": "pick ... up", "ja": "…を（車で）拾う", "kana": "ピック **ア**ップ", "phonetic": "/pɪk ʌp/", "hint": "p" },
            { "en": "on the way", "ja": "途中で", "kana": "オン・ザ・**ウェ**イ", "phonetic": "/ɑːn ðə weɪ/", "hint": "o" }
        ]
    },
    {
        "id": 13,
        "originalSentence": "He is on another line, so I will take a message.",
        "translation": "彼は別の電話に出ておりますので、ご伝言を承ります。",
        "quiz": { "target": "take a message", "choices": ["take a message", "make sense", "make a promise", "take a risk"], "meaning": "伝言を承る" },
        "application": {
            "situation": "バイト先に電話がかかってきて、店長を呼び出してほしいと言われたが、店長は接客中だった。",
            "sentence": "I'm sorry, the manager is with a customer right now. Can I take a message for you?",
            "translation": "申し訳ありません、店長はただいま接客中です。ご伝言を承りましょうか？"
        },
        "flashcards": [
            { "en": "on another line", "ja": "別の電話に出て", "kana": "オン・アナザー・**ラ**イン", "phonetic": "/ɑːn əˈnʌðər laɪn/", "hint": "o" },
            { "en": "right now", "ja": "今すぐ、たった今", "kana": "**ラ**イト・**ナ**ウ", "phonetic": "/raɪt naʊ/", "hint": "r" },
            { "en": "take a message", "ja": "伝言を承る", "kana": "テイク ア **メ**ッセージ", "phonetic": "/teɪk ə ˈmesɪdʒ/", "hint": "t" }
        ]
    },
    {
        "id": 14,
        "originalSentence": "As far as I'm concerned, this is a risk worth taking.",
        "translation": "私に関する限り、これは取る価値のあるリスクです。",
        "quiz": { "target": "risk", "choices": ["risk", "path", "career", "cough"], "meaning": "危険" },
        "application": {
            "situation": "起業するかどうか迷っている友達の背中を押してあげたい時。",
            "sentence": "Starting a business always involves some risk, but I think you're more than ready to <span class='tooltip-trigger'>take it on<span class='tooltip-content'>「（困難などに）立ち向かう、引き受ける」という意味。</span></span>.",
            "translation": "起業にはいつも多少のリスクが伴うけど、君ならそれを受ける準備は十分できてると思うよ。"
        },
        "flashcards": [
            { "en": "as far as I'm concerned", "ja": "私に関する限り", "kana": "アズ・**ファー**・アズ・アイム・コン**サー**ンド", "phonetic": "/æz fɑːr æz aɪm kənˈsɜːrnd/", "hint": "a" },
            { "en": "be willing to do", "ja": "～するのをいとわない", "kana": "ビー・**ウィ**リング・トゥ・ドゥ", "phonetic": "/bi ˈwɪlɪŋ tu du/", "hint": "b" },
            { "en": "take a risk", "ja": "危険を冒す", "kana": "**テイ**ク・ア・**リ**スク", "phonetic": "/teɪk ə rɪsk/", "hint": "t" }
        ]
    },
    {
        "id": 15,
        "originalSentence": "You should take advantage of this rare opportunity to travel.",
        "translation": "あなたはこのめったにない旅行の機会を利用するべきです。",
        "quiz": { "target": "Take advantage of", "choices": ["Take advantage of", "Make use of", "Get rid of", "Make fun of"], "meaning": "～を利用する" },
        "application": {
            "situation": "期間限定のセールで、欲しかったものが半額になっている！",
            "sentence": "This is a great chance to get that jacket you wanted. You should take advantage of the sale!",
            "translation": "欲しかったジャケットを手に入れる絶好のチャンスだよ。このセールを利用すべきだって！"
        },
        "flashcards": [
            { "en": "take advantage of", "ja": "～を利用する", "kana": "テイク アド**ヴァ**ンテージ オヴ", "phonetic": "/teɪk ədˈvæntɪdʒ əv/", "hint": "t" },
            { "en": "rare", "ja": "めったにない", "kana": "**レ**ア", "phonetic": "/rer/", "hint": "r" },
            { "en": "opportunity", "ja": "機会", "kana": "オポ**テュー**ニティ", "phonetic": "/ˌɑːpərˈtuːnəti/", "hint": "o" }
        ]
    },
    {
        "id": 16,
        "originalSentence": "He took her by the hand and she simply nodded.",
        "translation": "彼が彼女の手を取ると、彼女はただうなずいた。",
        "quiz": { "target": "nodded", "choices": ["nodded", "led", "took", "went"], "meaning": "うなずいた" },
        "application": {
            "situation": "カフェで「ケーキも頼んじゃう？」と目で合図したら、友達がにっこりしてうなずいた。",
            "sentence": "I asked if she wanted dessert too, and she just smiled and gave a little <span class='tooltip-trigger'>nod<span class='tooltip-content'>「うなずき」という名詞。'give a nod'で「うなずく」という動作を表す。</span></span>.",
            "translation": "彼女もデザートが欲しいか尋ねたら、彼女はただ微笑んで小さくうなずいただけだった。"
        },
        "flashcards": [
            { "en": "nod", "ja": "うなずく", "kana": "ノッド", "phonetic": "/nɑːd/", "hint": "n" },
            { "en": "take ... by the hand", "ja": "～の手を取る", "kana": "**テイ**ク・…・バイ・ザ・**ハ**ンド", "phonetic": "/teɪk baɪ ðə hænd/", "hint": "t" },
            { "en": "lead", "ja": "～を導く", "kana": "**リー**ド", "phonetic": "/liːd/", "hint": "l" },
            { "en": "shore", "ja": "岸", "kana": "**ショ**ア", "phonetic": "/ʃɔːr/", "hint": "s" }
        ]
    },
    {
        "id": 17,
        "originalSentence": "You should not drink alcohol on an empty stomach.",
        "translation": "空腹時にお酒を飲むべきではありません。",
        "quiz": { "target": "stomach", "choices": ["stomach", "throat", "pill", "temperature"], "meaning": "胃、お腹" },
        "application": {
            "situation": "食べ放題に来たけど、緊張のせいか全然お腹がすいていない…。",
            "sentence": "I'm too nervous to eat. I feel like I have <span class='tooltip-trigger'>butterflies in my stomach<span class='tooltip-content'>「（緊張や興奮で）胸がドキドキする、そわそわする」という意味のイディオム。</span></span>.",
            "translation": "緊張しすぎて食べられないよ。お腹がそわそわする感じ。"
        },
        "flashcards": [
            { "en": "pill", "ja": "錠剤", "kana": "**ピ**ル", "phonetic": "/pɪl/", "hint": "p" },
            { "en": "empty", "ja": "空の", "kana": "**エ**ンプティ", "phonetic": "/ˈempti/", "hint": "e" },
            { "en": "stomach", "ja": "胃、腹", "kana": "**スタ**マック", "phonetic": "/ˈstʌmək/", "hint": "s" }
        ]
    },
    {
        "id": 18,
        "originalSentence": "Jane is in bed with a sore throat and a terrible cough.",
        "translation": "ジェーンは喉の痛みとひどい咳があり、寝ている。。",
        "quiz": { "target": "sore", "choices": ["sore", "rare", "wise", "solid"], "meaning": "痛い" },
        "application": {
            "situation": "カラオケで歌いすぎて、次の日声がガラガラに。",
            "sentence": "I think I sang too much at karaoke last night. I have a really sore throat today.",
            "translation": "昨日の夜、カラオケで歌いすぎたみたい。今日、のどがすごく痛いんだ。"
        },
        "flashcards": [
            { "en": "sore", "ja": "痛い", "kana": "ソー", "phonetic": "/sɔːr/", "hint": "s" },
            { "en": "throat", "ja": "のど", "kana": "ス**ロ**ウト", "phonetic": "/θroʊt/", "hint": "t" },
            { "en": "cough", "ja": "咳", "kana": "**コ**フ", "phonetic": "/kɔːf/", "hint": "c" },
            { "en": "temperature", "ja": "体温；温度", "kana": "**テ**ンパラチャー", "phonetic": "/ˈtemprətʃər/", "hint": "t" }
        ]
    },
    {
        "id": 19,
        "originalSentence": "It's about time you thought about your future career path.",
        "translation": "そろそろ将来の進路について考えてもいい頃だ。",
        "quiz": { "target": "career", "choices": ["career", "skill", "knowledge", "aspect"], "meaning": "経歴、職業" },
        "application": {
            "situation": "大学3年生の秋。周りが就活を始め、自分も将来について真剣に考え始める時期。",
            "sentence": "I need to start thinking seriously about my future career. I'm not sure what I want to do yet.",
            "translation": "将来のキャリアについて真剣に考え始めないと。まだ何がしたいか分からないんだ。"
        },
        "flashcards": [
            { "en": "it's about time...", "ja": "そろそろ～してもよい時間だ", "kana": "イッツ・ア**バ**ウト・**タ**イム…", "phonetic": "/ɪts əˈbaʊt taɪm/", "hint": "i" },
            { "en": "decide", "ja": "～を決める", "kana": "ディ**サ**イド", "phonetic": "/dɪˈsaɪd/", "hint": "d" },
            { "en": "path", "ja": "道、進路", "kana": "**パ**ス", "phonetic": "/pæθ/", "hint": "p" },
            { "en": "career", "ja": "経歴、職業", "kana": "キャ**リ**ア", "phonetic": "/kəˈrɪr/", "hint": "c" }
        ]
    },
    {
        "id": 20,
        "originalSentence": "You need both knowledge and skill to acquire a license.",
        "translation": "免許を取得するには知識と技能の両方が必要です。",
        "quiz": { "target": "acquire", "choices": ["acquire", "predict", "deceive", "offer"], "meaning": "～を習得する" },
        "application": {
            "situation": "新しいゲームを始めたけど、操作が複雑でなかなか上手くならない。",
            "sentence": "This game is pretty complex. It's going to take some time to acquire all the necessary skills.",
            "translation": "このゲーム、かなり複雑だね。必要なスキルを全部習得するには時間がかかりそう。"
        },
        "flashcards": [
            { "en": "take time", "ja": "時間がかかる", "kana": "**テイ**ク・**タ**イム", "phonetic": "/teɪk taɪm/", "hint": "t" },
            { "en": "acquire", "ja": "～を習得する", "kana": "アク**ワ**イア", "phonetic": "/əˈkwaɪər/", "hint": "a" },
            { "en": "skill", "ja": "技能", "kana": "**スキ**ル", "phonetic": "/skɪl/", "hint": "s" },
            { "en": "knowledge", "ja": "知識", "kana": "**ノ**ーリッジ", "phonetic": "/ˈnɑːlɪdʒ/", "hint": "k" },
            { "en": "aspect", "ja": "側面", "kana": "**ア**スペクト", "phonetic": "/ˈæspekt/", "hint": "a" }
        ]
    }
];

// --- STATE ---
let currentSentenceIndex = 0;
let currentStep = 1;
let selfAssessments = new Array(quizData.length).fill(null);
let quizResults = new Array(quizData.length).fill(null);
let isSequentialMode = false; 
let speechSynthesis = window.speechSynthesis;
let englishVoice = null;
let quizQueue = [];

// --- DOM ELEMENTS ---
const tocScreen = document.getElementById('toc-screen');
const mainQuizArea = document.getElementById('main-quiz-area');
const finalCheckArea = document.getElementById('final-check-area');
const resultScreen = document.getElementById('result-screen');
const startSequentialBtn = document.getElementById('start-sequential-btn');
const finalCheckBtn = document.getElementById('final-check-btn');
const nextBtn = document.getElementById('next-btn');
const reviewBtn = document.getElementById('review-btn');
const backToTocBtn = document.getElementById('back-to-toc-btn');
const backToTocFromFinalBtn = document.getElementById('back-to-toc-from-final-btn');

const stepViews = {
    1: document.getElementById('step1-quiz'),
    2: document.getElementById('step2-application'),
    3: document.getElementById('step3-flashcards'),
};
const stepIndicators = {
    1: document.getElementById('step-indicator-1'),
    2: document.getElementById('step-indicator-2'),
    3: document.getElementById('step-indicator-3'),
};

// --- INITIALIZATION ---
document.addEventListener('DOMContentLoaded', showMainMenu);

// --- EVENT LISTENERS ---
startSequentialBtn.addEventListener('click', () => {
    isSequentialMode = true;
    quizResults = new Array(quizData.length).fill(null);
    selfAssessments = new Array(quizData.length).fill(null);
    startQuizSession(Array.from(Array(quizData.length).keys()));
});

finalCheckBtn.addEventListener('click', () => {
    isSequentialMode = false;
    renderFinalCheck();
});

nextBtn.addEventListener('click', handleNext);
reviewBtn.addEventListener('click', showMainMenu);
backToTocBtn.addEventListener('click', showMainMenu);
backToTocFromFinalBtn.addEventListener('click', showMainMenu);

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

function showMainMenu() {
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.add('hidden');
    finalCheckArea.classList.add('hidden');
    tocScreen.classList.remove('hidden');
    
    const tocList = document.getElementById('toc-list');
    tocList.innerHTML = '';
    quizData.forEach((data, index) => {
        const quizNeedsReview = quizResults[index] === false;
        const appNeedsReview = selfAssessments[index] === false;

        const item = document.createElement('button');
        item.className = 'toc-item p-3 border rounded-lg text-center hover:bg-gray-100 transition-colors flex flex-col justify-center items-center h-full';
        if (quizNeedsReview || appNeedsReview) item.classList.add('needs-review', 'border-red-400', 'bg-red-50');
        
        let reviewMarksHTML = '<div class="flex items-center mt-1">';
        if (quizNeedsReview) {
            reviewMarksHTML += '<span class="review-mark" title="クイズを復習">Q</span>';
        }
        if (appNeedsReview) {
            reviewMarksHTML += '<span class="review-mark" title="応用を復習">A</span>';
        }
        reviewMarksHTML += '</div>';

        item.innerHTML = `
            <div class="toc-item-content">
                <span class="font-semibold">例文 ${data.id}</span>
                ${(quizNeedsReview || appNeedsReview) ? reviewMarksHTML : ''}
            </div>
        `;
        item.onclick = () => {
            isSequentialMode = false;
            startQuizSession([index]);
        };
        tocList.appendChild(item);
    });
}


function startQuizSession(indices) {
    quizQueue = indices.map(i => ({ index: i, originalIndex: i })); 
    if (quizQueue.length === 0) {
        showMainMenu();
        return;
    }
    const session = quizQueue.shift();
    currentSentenceIndex = session.originalIndex;
    
    currentStep = 1;
    tocScreen.classList.add('hidden');
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.remove('hidden');
    nextBtn.classList.add('hidden');
    renderCurrentStep();
}

function handleNext() {
    currentStep++;

    if (currentStep > 3) { // After flashcards
        if (quizQueue.length > 0) { // More questions in queue
            const session = quizQueue.shift();
            currentSentenceIndex = session.originalIndex;
            currentStep = 1;
            renderCurrentStep();
        } else { // No more questions
            if (isSequentialMode) {
                 showResult();
            } else {
                showMainMenu();
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
    
    if (isSequentialMode) {
        const totalQuestions = quizData.length;
        const completedQuestions = totalQuestions - quizQueue.length - 1;
        const progress = (completedQuestions / totalQuestions) * 100;
        document.getElementById('progress-bar').style.width = `${progress}%`;
        document.getElementById('progress-bar').classList.remove('hidden');

    } else {
        document.getElementById('progress-bar').classList.add('hidden');
    }


    Object.values(stepIndicators).forEach(el => el.classList.remove('active'));
    if (stepIndicators[currentStep]) {
        stepIndicators[currentStep].classList.add('active');
    }
}

function renderCurrentStep() {
    updateProgress();
    Object.values(stepViews).forEach(view => view.classList.add('hidden'));
    stepViews[currentStep].classList.remove('hidden');
    nextBtn.classList.add('hidden');

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
    const questionText = originalSentence.replace(new RegExp(quiz.target.replace(/([.*+?^=!:${}()|\[\]\/\\])/g, "\\$1"), 'i'), '______');
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

    const nextText = quizQueue.length === 0 ? 
        (isSequentialMode ? '結果を見る' : '目次に戻る') : 
        '次の問題へ →';
    nextBtn.textContent = nextText;
    nextBtn.classList.remove('hidden');
}

function renderFinalCheck() {
    tocScreen.classList.add('hidden');
    mainQuizArea.classList.add('hidden');
    resultScreen.classList.add('hidden');
    finalCheckArea.classList.remove('hidden');

    let allHeadwords = quizData.flatMap(d => d.flashcards);
    allHeadwords.sort(() => Math.random() - 0.5);
    const container = document.getElementById('final-check-flashcards');
    renderFlashcards(container, allHeadwords, 'ja-en');
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
}

</script>
</body>
</html>
