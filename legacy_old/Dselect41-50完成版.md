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
    "id": 41,
    "originalSentence": "In practice, his theory didn't necessarily work as he expected.",
    "translation": "実際には、彼の理論は必ずしも彼が期待した通りには機能しなかった。",
    "quiz": {
      "target": "practice",
      "choices": ["practice", "theory", "mind", "work"],
      "meaning": "実行、実際"
    },
    "application": {
      "situation": "計画や勉強したことが、実世界でどうなるかについて話すとき。",
      "sentence": "I loved my marketing class, but putting the theories into practice at my new job is <span class='tooltip-trigger'>a whole different challenge<span class='tooltip-content'>「全く別の挑戦」という意味。予想していたことと大きく違う難しさがあることを示す表現。</span></span>.",
      "translation": "マーケティングの授業は大好きだったけど、新しい仕事でその理論を実践するのは全く別の挑戦だよ。"
    },
    "flashcards": [
      { "en": "keep in mind", "ja": "…を覚えておく", "kana": "キープ イン **マ**インドゥ", "phonetic": "/kiːp ɪn maɪnd/", "hint": "k" },
      { "en": "theory", "ja": "理論、説", "kana": "**スィ**オリィ", "phonetic": "/ˈθɪəri/", "hint": "t" },
      { "en": "necessarily", "ja": "必ずしも（…ない）", "kana": "ネサ**セ**ラリィ", "phonetic": "/ˌnesəˈserəli/", "hint": "n" },
      { "en": "work", "ja": "（計画などが）うまくいく", "kana": "**ワ**ァク", "phonetic": "/wɜːrk/", "hint": "w" }
    ]
  },
  {
    "id": 42,
    "originalSentence": "They had to fire the new employee they just hired.",
    "translation": "彼らは雇ったばかりの新しい従業員を解雇しなければならなかった。",
    "quiz": {
      "target": "fire",
      "choices": ["fire", "hire", "keep", "promote"],
      "meaning": "…を解雇する"
    },
    "application": {
      "situation": "仕事の厳しい現実や、突然の解雇について話すとき。",
      "sentence": "He got fired for being late too many times. It's <span class='tooltip-trigger'>a tough lesson to learn<span class='tooltip-content'>「学ぶには厳しい教訓」という意味。失敗などから得られる、つらいが良い経験を指す。</span></span>.",
      "translation": "彼は何度も遅刻したせいでクビになった。それは学ぶには厳しい教訓だ。"
    },
    "flashcards": [
      { "en": "the other day", "ja": "このあいだ、先日", "kana": "ズィ **ア**ザー デイ", "phonetic": "/ðiː ˈʌðər deɪ/", "hint": "t" },
      { "en": "hire", "ja": "…を雇う", "kana": "**ハ**イあ", "phonetic": "/ˈhaɪər/", "hint": "h" }
    ]
  },
  {
    "id": 43,
    "originalSentence": "Let's take turns doing the household chores this week.",
    "translation": "今週は家事を交代でやりましょう。",
    "quiz": {
      "target": "take turns",
      "choices": ["take turns", "do the dishes", "my turn", "this week"],
      "meaning": "（…を）交代でする"
    },
    "application": {
      "situation": "家事や仕事の分担について決めるとき。",
      "sentence": "<span class='tooltip-trigger'>To make it fair<span class='tooltip-content'>「公平にするために」という意味。何かを決めるときの前置きとしてよく使われる。</span></span>, let's take turns choosing the movie we watch each Friday night.",
      "translation": "公平にするために、毎週金曜の夜に見る映画は交代で選ぶことにしよう。"
    },
    "flashcards": [
      { "en": "take turns", "ja": "（…を）交代でする", "kana": "テイク **タ**ーンズ", "phonetic": "/teɪk tɜːrnz/", "hint": "t" },
      { "en": "do the dishes", "ja": "（食後に）食器を洗う", "kana": "ドゥー ザ **ディ**シズ", "phonetic": "/duː ðə ˈdɪʃɪz/", "hint": "d" },
      { "en": "one's turn", "ja": "…の順番", "kana": "ワンズ **タ**ーン", "phonetic": "/wʌnz tɜːrn/", "hint": "o" }
    ]
  },
  {
    "id": 44,
    "originalSentence": "I envy his talent for making friends so easily.",
    "translation": "彼が簡単に友達を作る才能を私は羨ましく思う。",
    "quiz": {
      "target": "envy",
      "choices": ["envy", "ignore", "hate", "know"],
      "meaning": "…を羨む"
    },
    "application": {
      "situation": "自分にはない他人の才能や状況を羨ましく思うとき。",
      "sentence": "I really envy people who can speak multiple languages <span class='tooltip-trigger'>fluently<span class='tooltip-content'>「流暢に、すらすらと」という意味。言語やスピーチが滞りなく滑らかであることを表す。</span></span>.",
      "translation": "複数の言語を流暢に話せる人が本当に羨ましいよ。"
    },
    "flashcards": [
      { "en": "envy", "ja": "…を羨む", "kana": "**エ**ンヴィ", "phonetic": "/ˈenvi/", "hint": "e" },
      { "en": "be good at", "ja": "…が上手だ", "kana": "ビー **グ**ッド アット", "phonetic": "/biː ɡʊd æt/", "hint": "b" },
      { "en": "make friends", "ja": "（…と）親しくなる", "kana": "メイク フ**レ**ンズ", "phonetic": "/meɪk frendz/", "hint": "m" }
    ]
  },
  {
    "id": 45,
    "originalSentence": "After exchanging greetings, let's get down to business.",
    "translation": "挨拶を交わしたら、仕事に取り掛かりましょう。",
    "quiz": {
      "target": "get down to",
      "choices": ["get down to", "shake hands", "exchange greetings", "go over"],
      "meaning": "（腰を据えて）…に取りかかる"
    },
    "application": {
      "situation": "雑談を終えて、本題に入りたいとき。",
      "sentence": "Okay everyone, we've had our coffee. Let's get down to business and discuss the <span class='tooltip-trigger'>marketing plan<span class='tooltip-content'>「販売戦略」のこと。製品をどのように市場に売り出していくかの計画。</span></span>.",
      "translation": "よしみんな、コーヒーは飲んだね。さあ、本題に入ってマーケティング計画について議論しよう。"
    },
    "flashcards": [
      { "en": "shake hands", "ja": "握手する", "kana": "シェイク **ハ**ンズ", "phonetic": "/ʃeɪk hændz/", "hint": "s" },
      { "en": "exchange", "ja": "…を交換する", "kana": "イクス**チェ**インジ", "phonetic": "/ɪksˈtʃeɪndʒ/", "hint": "e" },
      { "en": "greeting", "ja": "挨拶", "kana": "**グ**リーティング", "phonetic": "/ˈɡriːtɪŋ/", "hint": "g" }
    ]
  },
  {
    "id": 46,
    "originalSentence": "We should encourage them to reach their full potential.",
    "translation": "私たちは彼らが潜在能力を最大限に発揮できるよう励ますべきだ。",
    "quiz": {
      "target": "potential",
      "choices": ["potential", "talent", "skill", "career"],
      "meaning": "潜在能力、潜在性"
    },
    "application": {
      "situation": "部下や後輩の成長を期待して、新しい挑戦を促すとき。",
      "sentence": "She's a <span class='tooltip-trigger'>new hire<span class='tooltip-content'>「新入社員」や「新しく雇われた人」を指す言葉。</span></span>, but I can see she has a lot of potential. We should give her more responsibilities to help her grow.",
      "translation": "彼女は新人だけど、大きな潜在能力を秘めているのがわかる。成長を促すためにも、もっと責任のある仕事を与えるべきだ。"
    },
    "flashcards": [
      { "en": "grownup", "ja": "大人", "kana": "**グ**ロウナップ", "phonetic": "/ˈɡroʊnʌp/", "hint": "g" },
      { "en": "encourage", "ja": "…を勇気づける、奨励する", "kana": "エン**カ**リッジ", "phonetic": "/ɪnˈkɜːrɪdʒ/", "hint": "e" },
      { "en": "reach", "ja": "（…に）達する、届く", "kana": "**リ**ーチ", "phonetic": "/riːtʃ/", "hint": "r" }
    ]
  },
  {
    "id": 47,
    "originalSentence": "We finally persuaded him not to give up on his dream.",
    "translation": "私たちはついに彼が夢をあきらめないよう説得した。",
    "quiz": {
      "target": "persuaded",
      "choices": ["persuaded", "forced", "ordered", "advised"],
      "meaning": "…（人）を納得させる"
    },
    "application": {
      "situation": "友人がなかなか決心できないでいるのを、根気強く説得するとき。",
      "sentence": "<span class='tooltip-trigger'>It took a while<span class='tooltip-content'>「少し時間がかかった」という意味。物事がすぐに終わらなかったことを示す。</span></span>, but I finally persuaded my friend to join our hiking trip this weekend.",
      "translation": "少し時間がかかったけど、ついに友人を説得して今週末のハイキング旅行に参加してもらうことにしたよ。"
    },
    "flashcards": [
      { "en": "at last", "ja": "ついに", "kana": "アット **ラ**スト", "phonetic": "/æt læst/", "hint": "a" },
      { "en": "persuade", "ja": "…を説得して…させる", "kana": "パァス**ウェ**イドゥ", "phonetic": "/pərˈsweɪd/", "hint": "p" },
      { "en": "give up", "ja": "（…を）あきらめる", "kana": "ギヴ **ア**ップ", "phonetic": "/ɡɪv ʌp/", "hint": "g" }
    ]
  },
  {
    "id": 48,
    "originalSentence": "Please let me know your flight's departure date.",
    "translation": "あなたのフライトの出発日を教えてください。",
    "quiz": {
      "target": "departure",
      "choices": ["departure", "open", "schedule", "ticket"],
      "meaning": "出発"
    },
    "application": {
      "situation": "旅行や出張のフライト情報を確認するとき。",
      "sentence": "Could you please check the departure time for <span class='tooltip-trigger'>flight<span class='tooltip-content'>飛行機の「便」のこと。JL006便のように航空会社と番号で示される。</span></span> JL006 to New York?",
      "translation": "ニューヨーク行きJL006便の出発時間を確認していただけますか？"
    },
    "flashcards": [
      { "en": "remember to do", "ja": "忘れずに…する", "kana": "リ**メ**ンバァ トゥ ドゥー", "phonetic": "/rɪˈmembər tuː duː/", "hint": "r" },
      { "en": "let A know", "ja": "（…を）Aに知らせる", "kana": "レット エイ **ノ**ウ", "phonetic": "/let eɪ noʊ/", "hint": "l" },
      { "en": "date", "ja": "日付", "kana": "デイト", "phonetic": "/deɪt/", "hint": "d" }
    ]
  },
  {
    "id": 49,
    "originalSentence": "Don't forget to reply to her wedding invitation.",
    "translation": "彼女の結婚式の招待状に返信するのを忘れないで。",
    "quiz": {
      "target": "invitation",
      "choices": ["invitation", "reply", "message", "application"],
      "meaning": "招待状、招待"
    },
    "application": {
      "situation": "友人からパーティーの招待状を受け取ったとき。",
      "sentence": "I just received an invitation to Sarah's wedding! I need to <span class='tooltip-trigger'>RSVP<span class='tooltip-content'>「お返事ください」というフランス語の略。出欠を「返信する」という動詞としても使われる。</span></span> by next week.",
      "translation": "サラの結婚式の招待状が届いた！来週までに返事しなきゃ。"
    },
    "flashcards": [
      { "en": "forget to do", "ja": "…するのを忘れる", "kana": "フォ**ゲ**ット トゥ ドゥー", "phonetic": "/fərˈɡet tuː duː/", "hint": "f" },
      { "en": "reply", "ja": "（…だと）返事をする", "kana": "リプ**ラ**イ", "phonetic": "/rɪˈplaɪ/", "hint": "r" },
      { "en": "invitation", "ja": "招待状、招待", "kana": "インヴィ**テ**イション", "phonetic": "/ˌɪnvɪˈteɪʃən/", "hint": "i" }
    ]
  },
  {
    "id": 50,
    "originalSentence": "The teacher expected every student to participate in the debate.",
    "translation": "先生は全ての生徒がその討論会に参加することを期待した。",
    "quiz": {
      "target": "participate",
      "choices": ["participate", "observe", "debate", "decline"],
      "meaning": "（…に）参加する"
    },
    "application": {
      "situation": "チームのプロジェクトやイベントへの参加を促すとき。",
      "sentence": "We encourage everyone to actively participate in the <span class='tooltip-trigger'>brainstorming session<span class='tooltip-content'>アイデアを自由に出し合う会議のこと。「ブレスト」と略されることも多い。</span></span> tomorrow.",
      "translation": "明日のブレインストーミング会議には、全員が積極的に参加することを推奨します。"
    },
    "flashcards": [
      { "en": "expect", "ja": "…を期待する", "kana": "イクス**ペ**クト", "phonetic": "/ɪkˈspekt/", "hint": "e" },
      { "en": "participate", "ja": "（…に）参加する", "kana": "パァ**ティ**シペイト", "phonetic": "/pɑːrˈtɪsɪpeɪt/", "hint": "p" },
      { "en": "debate", "ja": "討論", "kana": "ディ**ベ**イト", "phonetic": "/dɪˈbeɪt/", "hint": "d" }
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
