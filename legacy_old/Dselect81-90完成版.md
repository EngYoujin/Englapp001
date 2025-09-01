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
        "id": 81,
        "originalSentence": "She decided to set up a popular blog after she graduated from college.",
        "translation": "彼女は大学を卒業した後、人気のブログを立ち上げることにした。",
        "quiz": {
            "target": "set up",
            "choices": ["set up", "took over", "worked for", "invested in"],
            "meaning": "…を設立する"
        },
        "application": {
            "situation": "友達と文化祭で何をやるか話し合っているとき。",
            "sentence": "Let's set up a small café for the school festival! We can sell coffee and homemade cookies. It'll be <span class='tooltip-trigger'>a huge hit<span class='tooltip-content'>「大成功」「大人気」という意味の口語表現。</span></span>!",
            "translation": "文化祭で小さなカフェを立ち上げようよ！コーヒーと手作りクッキーを売るんだ。絶対に大人気になるよ！"
        },
        "flashcards": [
            { "en": "set up", "ja": "…を設立する", "kana": "セット **ア**ップ", "phonetic": "/set ʌp/", "hint": "s" },
            { "en": "investment firm", "ja": "投資会社", "kana": "イン**ヴェ**ストメント **フ**ァーム", "phonetic": "/ɪnˈvestmənt fɜːrm/", "hint": "i" },
            { "en": "graduate from", "ja": "…を卒業する", "kana": "グ**ラ**ジュエイト フロム", "phonetic": "/ˈɡrædʒuət frʌm/", "hint": "g" },
            { "en": "establish", "ja": "…を設立する（同意語）", "kana": "イス**タ**ブリッシュ", "phonetic": "/ɪˈstæblɪʃ/", "hint": "e" }
        ]
    },
    {
        "id": 82,
        "originalSentence": "We need to establish a team to investigate this strange incident immediately.",
        "translation": "この奇妙な事件を直ちに調査するため、我々はチームを立ち上げる必要がある。",
        "quiz": {
            "target": "investigate",
            "choices": ["investigate", "forget", "hide", "announce"],
            "meaning": "…を調査する"
        },
        "application": {
            "situation": "部屋に置いてあったお菓子が消えた！誰が食べたのか、探偵ごっこを始めるとき。",
            "sentence": "My snacks are gone! I'm going to investigate this case like a detective and find the <span class='tooltip-trigger'>culprit<span class='tooltip-content'>「犯人」「罪人」のこと。ミステリー小説などでよく使われる。</span></span>.",
            "translation": "僕のお菓子がない！探偵みたいにこの事件を調査して、犯人を見つけてやる。"
        },
        "flashcards": [
            { "en": "committee", "ja": "委員会", "kana": "コ**ミ**ティ", "phonetic": "/kəˈmɪti/", "hint": "c" },
            { "en": "establish", "ja": "…を設立する", "kana": "イス**タ**ブリッシュ", "phonetic": "/ɪˈstæblɪʃ/", "hint": "e" },
            { "en": "investigate", "ja": "…を調査する", "kana": "イン**ヴェ**スティゲイト", "phonetic": "/ɪnˈvestɪɡeɪt/", "hint": "i" },
            { "en": "incident", "ja": "出来事、事件", "kana": "**イ**ンスィデント", "phonetic": "/ˈɪnsɪdənt/", "hint": "i" }
        ]
    },
    {
        "id": 83,
        "originalSentence": "My phone warned me that the battery was about to die completely.",
        "translation": "私のスマホは、バッテリーが完全になくなりそうだと警告してきた。",
        "quiz": {
            "target": "was about to",
            "choices": ["was about to", "had just", "was unlikely to", "wanted to"],
            "meaning": "（間もなく）…するところである"
        },
        "application": {
            "situation": "映画館で、ポップコーンを買いに行こうとしたら本編が始まりそうなとき。",
            "sentence": "I was just about to go get more popcorn, but it looks like the movie is <span class='tooltip-trigger'>starting<span class='tooltip-content'>'be about to'の後には動詞の原形が来る。ここでは'start'。</span></span>. I'll wait.",
            "translation": "ポップコーンのおかわりを買いに行こうとしたところだったけど、映画が始まりそうだ。待つことにするよ。"
        },
        "flashcards": [
            { "en": "warn", "ja": "…に警告する", "kana": "ウォーン", "phonetic": "/wɔːrn/", "hint": "w" },
            { "en": "horrible", "ja": "恐ろしい", "kana": "**ホ**ラブル", "phonetic": "/ˈhɔːrəbl/", "hint": "h" },
            { "en": "be about to do", "ja": "まさに…するところだ", "kana": "ビー ア**バ**ウト トゥ ドゥ", "phonetic": "/biː əˈbaʊt tuː duː/", "hint": "b" },
            { "en": "happen", "ja": "起こる", "kana": "**ハ**プン", "phonetic": "/ˈhæpən/", "hint": "h" }
        ]
    },
    {
        "id": 84,
        "originalSentence": "The annual company party will take place at a fancy hotel this year.",
        "translation": "毎年恒例の会社のパーティーは、今年は高級ホテルで行われる。",
        "quiz": {
            "target": "take place",
            "choices": ["take place", "be canceled", "be postponed", "be finished"],
            "meaning": "（行事などが）行われる"
        },
        "application": {
            "situation": "楽しみにしている地域の夏祭りについて話すとき。",
            "sentence": "The annual summer festival is scheduled to take place this weekend. I can't wait for the <span class='tooltip-trigger'>fireworks<span class='tooltip-content'>「花火」のこと。通常、複数形で使われる。</span></span>!",
            "translation": "毎年恒例の夏祭りが今週末に行われる予定なんだ。花火が待ちきれないよ！"
        },
        "flashcards": [
            { "en": "pass away", "ja": "亡くなる", "kana": "パス ア**ウェ**イ", "phonetic": "/pæs əˈweɪ/", "hint": "p" },
            { "en": "funeral", "ja": "葬式", "kana": "**フ**ューナラル", "phonetic": "/ˈfjuːnərəl/", "hint": "f" },
            { "en": "take place", "ja": "行われる、起こる", "kana": "テイク プ**レ**イス", "phonetic": "/teɪk pleɪs/", "hint": "t" },
            { "en": "be held", "ja": "開催される（同意語）", "kana": "ビー **ヘ**ルド", "phonetic": "/biː held/", "hint": "b" }
        ]
    },
    {
        "id": 85,
        "originalSentence": "I tried to contact customer service, but I couldn't get through at all.",
        "translation": "カスタマーサービスに連絡しようとしたが、全く電話が繋がらなかった。",
        "quiz": {
            "target": "get through",
            "choices": ["get through", "hang up", "call back", "hold on"],
            "meaning": "（電話が）つながる"
        },
        "application": {
            "situation": "人気アーティストのライブチケットの予約電話が、全然つながらないとき。",
            "sentence": "The ticket line is always busy. I've been trying for an hour, but I can't get through!",
            "translation": "チケットの電話窓口、いつも混んでるんだ。もう1時間もかけてるけど、全然つながらないよ！"
        },
        "flashcards": [
            { "en": "contact", "ja": "…に連絡を取る", "kana": "**コ**ンタクト", "phonetic": "/ˈkɑːntækt/", "hint": "c" },
            { "en": "directly", "ja": "直接に", "kana": "ディ**レ**クトリィ", "phonetic": "/dəˈrektli/", "hint": "d" },
            { "en": "get through", "ja": "（電話が）つながる", "kana": "ゲット ス**ル**ー", "phonetic": "/ɡet θruː/", "hint": "g" }
        ]
    },
    {
        "id": 86,
        "originalSentence": "Please get in touch with me if you have any trouble on your business trip.",
        "translation": "出張中に何か問題があったら、私に連絡してください。",
        "quiz": {
            "target": "get in touch with",
            "choices": ["get in touch with", "catch up with", "make friends with", "hang out with"],
            "meaning": "…と連絡を取る"
        },
        "application": {
            "situation": "久しぶりに会った旧友と、これからも連絡を取り合いたいとき。",
            "sentence": "It was so great seeing you! Let's <span class='tooltip-trigger'>keep in touch<span class='tooltip-content'>「連絡を取り合おうね」という別れ際の挨拶。'get in touch'は連絡を取る一回の行動を指すことが多い。</span></span> from now on.",
            "translation": "会えて本当に良かったよ！これからは連絡を取り合おうね。"
        },
        "flashcards": [
            { "en": "get in touch with", "ja": "…と連絡を取る", "kana": "ゲット イン **タ**ッチ ウィズ", "phonetic": "/ɡet ɪn tʌtʃ wɪð/", "hint": "g" },
            { "en": "be away", "ja": "留守にしている", "kana": "ビー ア**ウェ**イ", "phonetic": "/biː əˈweɪ/", "hint": "b" },
            { "en": "on business", "ja": "仕事で、出張で", "kana": "オン **ビ**ズネス", "phonetic": "/ɑːn ˈbɪznəs/", "hint": "o" },
            { "en": "contact", "ja": "…に連絡する（同意語）", "kana": "**コ**ンタクト", "phonetic": "/ˈkɑːntækt/", "hint": "c" }
        ]
    },
    {
        "id": 87,
        "originalSentence": "In fact, many people suffer from terrible pollen allergies every single spring.",
        "translation": "実は、非常に多くの人々が毎年春になるとひどい花粉症に苦しんでいる。",
        "quiz": {
            "target": "suffer",
            "choices": ["suffer", "enjoy", "avoid", "ignore"],
            "meaning": "（苦痛など）を受ける、苦しむ"
        },
        "application": {
            "situation": "ひどい風邪をひいて、何日も寝込んでいる友人の様子を話すとき。",
            "sentence": "He's been suffering from a terrible cold all week. He has a high fever and a <span class='tooltip-trigger'>nasty cough<span class='tooltip-content'>「ひどい咳」という意味。'nasty'は「不快な」「ひどい」を表す。</span></span>.",
            "translation": "彼は一週間ずっとひどい風邪に苦しんでいるんだ。高熱とひどい咳が出てる。"
        },
        "flashcards": [
            { "en": "in fact", "ja": "実は", "kana": "イン **ファ**クト", "phonetic": "/ɪn fækt/", "hint": "i" },
            { "en": "suffer", "ja": "…で苦しむ", "kana": "**サ**ファー", "phonetic": "/ˈsʌfər/", "hint": "s" },
            { "en": "physical abuse", "ja": "肉体的虐待", "kana": "**フィ**ズィカル ア**ビュ**ース", "phonetic": "/ˈfɪzɪkl əˈbjuːs/", "hint": "p" },
            { "en": "go through", "ja": "（嫌なこと）を経験する（同意語）", "kana": "ゴウ ス**ル**ー", "phonetic": "/ɡoʊ θruː/", "hint": "g" }
        ]
    },
    {
        "id": 88,
        "originalSentence": "I have sympathy for her because I went through a similar situation.",
        "translation": "私も似たような状況を経験したので、彼女に同情します。",
        "quiz": {
            "target": "sympathy",
            "choices": ["sympathy", "anger", "joy", "interest"],
            "meaning": "同情、共感"
        },
        "application": {
            "situation": "友人がペットを亡くして落ち込んでいる。慰めの言葉をかけるとき。",
            "sentence": "I have so much sympathy for you right now. Losing a pet is like losing a <span class='tooltip-trigger'>family member<span class='tooltip-content'>「家族の一員」のこと。ペットなど、大切な存在を指す。</span></span>.",
            "translation": "あなたの気持ち、本当によくわかるよ。ペットを失うのは家族を失うのと同じくらい辛いよね。"
        },
        "flashcards": [
            { "en": "sympathy", "ja": "同情、共感", "kana": "**スィ**ンパスィー", "phonetic": "/ˈsɪmpəθi/", "hint": "s" },
            { "en": "go through", "ja": "…を経験する", "kana": "ゴウ ス**ル**ー", "phonetic": "/ɡoʊ θruː/", "hint": "g" },
            { "en": "similar", "ja": "類似の", "kana": "**スィ**マラー", "phonetic": "/ˈsɪmələr/", "hint": "s" },
            { "en": "pity", "ja": "同情、あわれみ（同意語）", "kana": "**ピ**ティ", "phonetic": "/ˈpɪti/", "hint": "p" }
        ]
    },
    {
        "id": 89,
        "originalSentence": "He is recovering from the flu at a slow but steady rate.",
        "translation": "彼はゆっくりだが着実なペースでインフルエンザから回復している。",
        "quiz": {
            "target": "recovering",
            "choices": ["recovering", "suffering", "starting", "declining"],
            "meaning": "回復する"
        },
        "application": {
            "situation": "風邪をひいて休んでいたが、だいぶ良くなってきたことを伝えるとき。",
            "sentence": "Thanks for asking. I'm slowly recovering from my cold. I should be <span class='tooltip-trigger'>back on my feet<span class='tooltip-content'>「病気や困難から回復する」「元気を取り戻す」という意味のイディオム。</span></span> in a few days.",
            "translation": "心配してくれてありがとう。風邪、ゆっくりだけど回復してきてるよ。あと数日で元気になるはず。"
        },
        "flashcards": [
            { "en": "recover from", "ja": "…から回復する", "kana": "リ**カ**ヴァー フロム", "phonetic": "/rɪˈkʌvər frʌm/", "hint": "r" },
            { "en": "recession", "ja": "不況", "kana": "リ**セ**ション", "phonetic": "/rɪˈseʃn/", "hint": "r" },
            { "en": "steady", "ja": "着実な、安定した", "kana": "**ス**テディ", "phonetic": "/ˈstedi/", "hint": "s" },
            { "en": "rate", "ja": "率、ペース", "kana": "レイト", "phonetic": "/reɪt/", "hint": "r" }
        ]
    },
    {
        "id": 90,
        "originalSentence": "It took him a long time to get over breaking up with his girlfriend.",
        "translation": "彼がガールフレンドと別れたことから立ち直るのには長い時間がかかった。",
        "quiz": {
            "target": "get over",
            "choices": ["get over", "start", "forget", "suffer from"],
            "meaning": "…を克服する、…から立ち直る"
        },
        "application": {
            "situation": "失恋した友達を励ますとき。「時間が解決してくれるよ」と伝えたい。",
            "sentence": "I know it hurts now, but <span class='tooltip-trigger'>you'll get over it<span class='tooltip-content'>「君なら乗り越えられるよ」「大丈夫だよ」と励ます決まり文句。</span></span>. Time heals all wounds.",
            "translation": "今は辛いと思うけど、きっと乗り越えられるよ。時間がすべてを癒してくれるから。"
        },
        "flashcards": [
            { "en": "get over", "ja": "…を克服する", "kana": "ゲット **オ**ーヴァー", "phonetic": "/ɡet ˈoʊvər/", "hint": "g" },
            { "en": "depression", "ja": "憂鬱、落ち込み", "kana": "ディプ**レ**ッション", "phonetic": "/dɪˈpreʃn/", "hint": "d" },
            { "en": "break up with", "ja": "…と別れる", "kana": "ブレイク **ア**ップ ウィズ", "phonetic": "/breɪk ʌp wɪð/", "hint": "b" },
            { "en": "overcome", "ja": "…を克服する（同意語）", "kana": "オーヴァー**カ**ム", "phonetic": "/ˌoʊvərˈkʌm/", "hint": "o" }
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
