<script>
// --- LEARNING APP LOGIC ---
/*
================================================================================
TEMPLATE INSTRUCTION / テンプレート利用方法
================================================================================
以下の `const quizData = [ ... ];` の `[` と `]` の間に、
新しい例文データを貼り付けてください。
================================================================================
*/
const quizData = [
    // ここに例文データを挿入
];

// --- PERSISTENCE KEYS ---
const KEY_PREFIX = 'duoSelect_';
const KEY = `${KEY_PREFIX}progress_v1`;
const VOICE_MODE = `${KEY_PREFIX}voice_mode_v1`;
const CARD_MODE = `${KEY_PREFIX}card_mode_v1`;
const HINT_MODE = `${KEY_PREFIX}hint_mode_v1`;

// --- [FROM D3.0] PERSISTENCE & SRS ---
function loadProgress() {
    try {
        return JSON.parse(localStorage.getItem(KEY)) || { cards: {} };
    } catch (e) {
        console.error("Failed to load progress:", e);
        return { cards: {} };
    }
}

function saveProgress(p) {
    try {
        localStorage.setItem(KEY, JSON.stringify(p));
    } catch (e) {
        console.error("Failed to save progress:", e);
    }
}

function isDue(card) {
    return !card || !card.next_due || Date.now() >= card.next_due;
}

function tomorrowPlusDays(n = 0) {
    const d = new Date();
    const t = new Date(d.getFullYear(), d.getMonth(), d.getDate() + 1);
    return t.getTime() + n * 86400000;
}

function markWrong(id) {
    const p = loadProgress();
    const c = p.cards[id] || {};
    c.last_result = 'wrong';
    c.interval = 0;
    c.next_due = Date.now() - 1; // Mark as due immediately
    p.cards[id] = c;
    saveProgress(p);
    console.log(`Marked wrong: ${id}`, c);
}

function markCorrect(id) {
    const p = loadProgress();
    const c = p.cards[id] || {};
    c.last_result = 'correct';
    const currentInterval = c.interval || 0;
    const nextInterval = currentInterval === 0 ? 1 : currentInterval === 1 ? 3 : Math.min(currentInterval * 2, 30);
    c.interval = nextInterval;
    c.next_due = tomorrowPlusDays(nextInterval);
    c.done = true;
    p.cards[id] = c;
    saveProgress(p);
    console.log(`Marked correct: ${id}`, c);
}

// --- ID GENERATION ---
function getItemId(sentenceId, type, identifier = '') {
    let safeIdentifier = identifier.toLowerCase().replace(/[^a-z0-9]/g, '_').substring(0, 20);
    return `s${sentenceId}-${type}${identifier ? `-${safeIdentifier}` : ''}`;
}


// --- STATE ---
let currentSentenceIndex = 0;
let currentStep = 1;
let isSequentialMode = false; 

// --- [REPLACED] Voice/Audio related state
let speechSynthesis = window.speechSynthesis;
// This cache will be populated by the new pickVoices function.
let voiceCache = { male: null, female: null, auto: [], toggle: 0 };

let quizQueue = [];
let stepUnlocked = 1;
let finalCheckCards = [];

// --- DOM ELEMENTS ---
const tocScreen = document.getElementById('toc-screen');
const reviewArea = document.getElementById('review-area');
const mainQuizArea = document.getElementById('main-quiz-area');
const finalCheckArea = document.getElementById('final-check-area');
const resultScreen = document.getElementById('result-screen');
const startSequentialBtn = document.getElementById('start-sequential-btn');
const finalCheckBtn = document.getElementById('final-check-btn');
const nextBtn = document.getElementById('next-btn');
const reviewBtn = document.getElementById('review-btn');
const backToTocBtn = document.getElementById('back-to-toc-btn');
const backToTocFromFinalBtn = document.getElementById('back-to-toc-from-final-btn');
const prevSentenceBtn = document.getElementById('prev-sentence-btn');
const nextSentenceBtn = document.getElementById('next-sentence-btn');
const shuffleFinalCheckBtn = document.getElementById('shuffle-final-check-btn');

const stepViews = {
    1: document.getElementById('step1-quiz'),
    2: document.getElementById('step2-application'),
    3: document.getElementById('step3-flashcards'),
};
const stepIndicatorsContainer = document.getElementById('step-indicators-container');

// --- INITIALIZATION ---
document.addEventListener('DOMContentLoaded', async () => {
    // Asynchronously load and filter voices first
    if ('speechSynthesis' in window) {
        await pickVoices();
    }
    showMainMenu();
    initAudioControls();
});

// --- EVENT LISTENERS ---
startSequentialBtn.addEventListener('click', () => {
    isSequentialMode = true;
    quizQueue = Array.from(Array(quizData.length).keys());
    startQuizSession(quizQueue);
});

finalCheckBtn.addEventListener('click', () => {
    isSequentialMode = false;
    finalCheckCards = quizData.flatMap(d => 
        d.flashcards.map(fc => ({ ...fc, sentenceId: d.id }))
    );
    finalCheckCards.sort(() => Math.random() - 0.5);
    renderFinalCheck();
});

nextBtn.addEventListener('click', handleNext);
reviewBtn.addEventListener('click', showMainMenu);
backToTocBtn.addEventListener('click', showMainMenu);
backToTocFromFinalBtn.addEventListener('click', showMainMenu);
shuffleFinalCheckBtn.addEventListener('click', () => {
    finalCheckCards.sort(() => Math.random() - 0.5);
    renderFinalCheck();
});

prevSentenceBtn.addEventListener('click', () => {
    if (currentSentenceIndex > 0) {
        navigateToSentence(currentSentenceIndex - 1);
    }
});
nextSentenceBtn.addEventListener('click', () => {
    if (currentSentenceIndex < quizData.length - 1) {
        navigateToSentence(currentSentenceIndex + 1);
    }
});

// Add header button listeners
document.getElementById('header-toc-link').addEventListener('click', (e) => { e.preventDefault(); showMainMenu(); });
document.getElementById('header-review-link').addEventListener('click', (e) => { e.preventDefault(); renderReview(); });
document.getElementById('reset-data-btn').addEventListener('click', () => {
    if (confirm('本当に学習データをすべてリセットしますか？')) {
        localStorage.removeItem(KEY);
        alert('学習データをリセットしました。');
        showMainMenu();
    }
});

stepIndicatorsContainer.addEventListener('click', (e) => {
    if (e.target.matches('.step-indicator')) {
        const targetStep = parseInt(e.target.dataset.step, 10);
        if (targetStep <= stepUnlocked) {
            currentStep = targetStep;
            renderCurrentStep();
        }
    }
});


// --- [Ultimate Hybrid] Voice Initialization Logic ---

// --- Common Helpers ---
function listVoices() {
    return new Promise(resolve => {
        let voices = speechSynthesis.getVoices();
        if (voices.length) {
            return resolve(voices);
        }
        speechSynthesis.onvoiceschanged = () => {
            voices = speechSynthesis.getVoices();
            resolve(voices);
        };
    });
}

function isMobile() {
    const ua = navigator.userAgent;
    return /android|iphone|ipad|ipod/i.test(ua);
}

// --- Mobile-Specific Logic (Stability First) ---
async function pickVoicesForMobile() {
    console.log("Mobile device detected. Applying strict voice filtering for stability.");
    const allVoices = await listVoices();
    const safeVoices = allVoices.filter(v => v.lang.startsWith('en-') && !/hindi|india|googlee|generic|network|xiaomi/i.test(v.name));
    if (safeVoices.length === 0) {
        console.warn("Mobile: No safe English voices found.");
        return;
    }

    let mobileMale = safeVoices.find(v => v.name === "Daniel") || safeVoices.find(v => /male/i.test(v.name));
    let mobileFemale = safeVoices.find(v => v.name === "Samantha") || safeVoices.find(v => /female/i.test(v.name));

    // Fallbacks
    if (!mobileMale) mobileMale = safeVoices.find(v => !/female/i.test(v.name)) || safeVoices[0];
    if (!mobileFemale) mobileFemale = safeVoices.find(v => v !== mobileMale) || safeVoices[0];

    voiceCache.male = mobileMale || null;
    voiceCache.female = mobileFemale || null;
    voiceCache.auto = [mobileMale, mobileFemale].filter(v => v !== null && v !== undefined);
    if (voiceCache.auto.length === 0 && safeVoices.length > 0) {
      voiceCache.auto.push(safeVoices[0]);
    }
}

// --- PC-Specific Logic (Diversity First) ---
async function pickVoicesForPC() {
    console.log("Desktop PC detected. Reversing logic to find a male voice first.");
    const allVoices = await listVoices();
    if (!allVoices.length) { console.warn("PC: No voices available."); return; }

    let enVoices = allVoices.filter(v => v.lang.startsWith('en-'));
    if (enVoices.length === 0) { enVoices = allVoices.filter(v => /english/i.test(v.name)); }

    if (enVoices.length === 0) {
        console.warn("PC: No English voices found. Using any available voices.");
        voiceCache.male = allVoices[0] || null;
        voiceCache.female = allVoices[1] || allVoices[0] || null;
        voiceCache.auto = allVoices;
        return;
    }

    const femaleKeywords = /female|jenny|aria|zira|samantha|tessa|susan|catherine/i;
    const maleKeywords = /male|david|mark|daniel|paul|alan|guy/i;

    let femaleVoice, maleVoice;

    // --- Reversed Strategy: Prioritize getting a good MALE voice, then find a DIFFERENT female voice. ---

    // 1. Find the best possible male voice (or at least, the most "not-female" voice).
    maleVoice = enVoices.find(v => maleKeywords.test(v.name)) 
                || enVoices.find(v => maleKeywords.test(v.voiceURI || ''))
                || enVoices.find(v => !femaleKeywords.test(v.name) && !femaleKeywords.test(v.voiceURI || '')) // Find a voice that isn't explicitly female
                || enVoices[0]; // Ultimate fallback to the very first English voice

    // 2. Find a FEMALE voice that is explicitly DIFFERENT from the male voice.
    femaleVoice = enVoices.find(v => v !== maleVoice && femaleKeywords.test(v.name))
                  || enVoices.find(v => v !== maleVoice); // Absolute fallback: just get any other voice.

    // If we only found one voice total, femaleVoice might be null. In that case, use the male voice for both.
    if (!femaleVoice) {
        femaleVoice = maleVoice;
    }

    voiceCache.female = femaleVoice;
    voiceCache.male = maleVoice;

    console.log(`[Reversed Logic] Selected Male: ${voiceCache.male?.name} (${voiceCache.male?.lang})`);
    console.log(`[Reversed Logic] Selected Female: ${voiceCache.female?.name} (${voiceCache.female?.lang})`);

    // For 'auto' mode, use all available English voices for maximum diversity
    voiceCache.auto = Array.from(new Set(enVoices)).sort(() => 0.5 - Math.random());
    if (voiceCache.auto.length === 0 && allVoices.length > 0) {
        voiceCache.auto = allVoices;
    }
}

// --- Main Hybrid Voice Selector ---
async function pickVoices() {
    try {
        if (isMobile()) {
            await pickVoicesForMobile();
        } else {
            await pickVoicesForPC();
        }
        
        // Final sanity checks to prevent empty voice cache
        if (!voiceCache.auto || voiceCache.auto.length === 0) {
             const allVoices = await listVoices();
             const enVoices = allVoices.filter(v => v.lang.startsWith('en-'));
             voiceCache.auto = enVoices.length > 0 ? [enVoices[0]] : (allVoices.length > 0 ? [allVoices[0]] : []);
        }
        if (!voiceCache.male) voiceCache.male = voiceCache.auto[0] || null;
        if (!voiceCache.female) voiceCache.female = voiceCache.auto.find(v => v !== voiceCache.male) || voiceCache.auto[0] || null;

    } catch (e) {
        console.error('Error picking voices', e);
        // Ultimate fallback
        const allVoices = speechSynthesis.getVoices();
        const enVoice = allVoices.find(v => v.lang.startsWith('en-')) || allVoices[0];
        if (enVoice) {
            voiceCache.male = enVoice;
            voiceCache.female = enVoice;
            voiceCache.auto = [enVoice];
        }
    }
}


function navigateToSentence(index) {
    if (index < 0 || index >= quizData.length) return;

    // Using sentence nav implies individual study mode
    isSequentialMode = false;
    quizQueue = []; // Clear queue

    currentSentenceIndex = index;
    currentStep = 1;
    stepUnlocked = 3; // Unlock all steps immediately for free navigation
    renderCurrentStep();
}

function initAudioControls() {
    // Voice settings
    const voiceSeg = document.getElementById('voiceSeg');
    if (voiceSeg) {
        voiceSeg.addEventListener('click', (e) => {
            const v = e.target?.dataset?.v;
            if (!v) return;
            localStorage.setItem(VOICE_MODE, v);
            syncAppSettings();
        });
    }

    // Card Mode settings
    const cardModeSeg = document.getElementById('cardModeSeg');
    if (cardModeSeg) {
        cardModeSeg.addEventListener('click', (e) => {
            const v = e.target?.dataset?.v;
            if (!v) return;
            localStorage.setItem(CARD_MODE, v);
            syncAppSettings();
        });
    }

    // Hint Mode settings
    const hintModeSeg = document.getElementById('hintModeSeg');
    if (hintModeSeg) {
        hintModeSeg.addEventListener('click', (e) => {
            const v = e.target?.dataset?.v;
            if (!v) return;
            localStorage.setItem(HINT_MODE, v);
            syncAppSettings();
        });
    }

    syncAppSettings();
}

function syncAppSettings() {
    // Sync Voice
    const voiceSeg = document.getElementById('voiceSeg');
    if (voiceSeg) {
        const currentVoiceMode = localStorage.getItem(VOICE_MODE) || 'auto';
        [...voiceSeg.querySelectorAll('button')].forEach(b => b.classList.toggle('on', b.dataset.v === currentVoiceMode));
    }
    
    // Sync Card Mode
    const cardModeSeg = document.getElementById('cardModeSeg');
    const currentCardMode = localStorage.getItem(CARD_MODE) || 'en-ja';
    if (cardModeSeg) {
        [...cardModeSeg.querySelectorAll('button')].forEach(b => b.classList.toggle('on', b.dataset.v === currentCardMode));
    }

    // Sync final check description
    const finalCheckDescription = document.getElementById('final-check-description');
    if (finalCheckDescription) {
        if (currentCardMode === 'ja-en') {
            finalCheckDescription.textContent = '日本語を見て、対応する英単語を思い出してみましょう。カードをタップすると答えが表示されます。';
        } else {
            finalCheckDescription.textContent = '英語を見て、対応する日本語を思い出してみましょう。カードをタップすると答えが表示されます。';
        }
    }

    // Sync Hint Mode and its dependency on Card Mode
    const hintModeSeg = document.getElementById('hintModeSeg');
    if (hintModeSeg) {
        const currentHintMode = localStorage.getItem(HINT_MODE) || 'on';
        const isHintingDisabled = currentCardMode === 'en-ja';
        
        hintModeSeg.style.opacity = isHintingDisabled ? '0.5' : '1';
        [...hintModeSeg.querySelectorAll('button')].forEach(b => {
            b.classList.toggle('on', b.dataset.v === currentHintMode);
            b.disabled = isHintingDisabled;
        });
    }
}

// --- FUNCTIONS ---

function sanitizeForTTS(s) {
    if (!s) return s;
    let t = s.replace(/\([^)]*\)/g, '').replace(/（[^）]*）/g, '');
    t = t.replace(/<[^>]+>/g, '');
    t = t.replace(/[↔↮→—–…\/｜≠=]/g, ' ');
    t = t.replace(/[\u3040-\u30FF\u3400-\u9FFF\uF900-\uFAFF]/g, '');
    t = t.replace(/\s{2,}/g, ' ').trim();
    return t;
}

function getVoiceMode() { return localStorage.getItem(VOICE_MODE) || 'auto' }

// Main speech function.
function speak(text) {
    if (!('speechSynthesis' in window) || !text) return;
    try { speechSynthesis.cancel(); } catch (e) {}
    
    const prepared = sanitizeForTTS(text);
    if (!prepared) return;

    const rateEl = document.getElementById('rate');
    const rawRate = rateEl ? parseFloat(rateEl.value) : 1.0;
    const mode = getVoiceMode();

    const clamp = (val, min, max) => Math.max(min, Math.min(max, val));

    let voiceToUse = null;
    if (mode === 'male') {
        voiceToUse = voiceCache.male;
    } else if (mode === 'female') {
        voiceToUse = voiceCache.female;
    } else {
        if (voiceCache.auto && voiceCache.auto.length > 0) {
            voiceCache.toggle = (voiceCache.toggle + 1) % voiceCache.auto.length;
            voiceToUse = voiceCache.auto[voiceCache.toggle];
        }
    }
    
    const u = new SpeechSynthesisUtterance(prepared);
    u.lang = 'en-US';
    u.rate = clamp(rawRate, 0.8, 1.2);
    u.pitch = clamp(1.0, 0.85, 1.15);
    u.voice = voiceToUse;
    
    // Safety check for when voices are not yet loaded
    if (u.voice || speechSynthesis.getVoices().length > 0) {
        speechSynthesis.speak(u);
    } else {
        setTimeout(() => {
            // Re-attempt to assign a voice if it loaded late
            if (!u.voice) {
                 if (mode === 'male') voiceToUse = voiceCache.male;
                 else if (mode === 'female') voiceToUse = voiceCache.female;
                 else if (voiceCache.auto && voiceCache.auto.length) voiceToUse = voiceCache.auto[voiceCache.toggle];
                 u.voice = voiceToUse;
            }
            speechSynthesis.speak(u);
        }, 250);
    }
}

function showMainMenu() {
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.add('hidden');
    finalCheckArea.classList.add('hidden');
    reviewArea.classList.add('hidden');
    tocScreen.classList.remove('hidden');
    
    const tocList = document.getElementById('toc-list');
    tocList.innerHTML = '';
    const progress = loadProgress();

    // --- [FROM D3.0] Review Banner Logic ---
    const allDue = Object.keys(progress.cards).filter(id => isDue(progress.cards[id]));
    const urgent = allDue.filter(id => progress.cards[id].last_result === 'wrong');
    const scheduled = allDue.filter(id => progress.cards[id].last_result !== 'wrong');
    
    const reviewBannerContainer = document.getElementById('review-banner-container');
    const headerReviewBadge = document.getElementById('header-review-badge');

    if (allDue.length > 0) {
        reviewBannerContainer.innerHTML = `
            <div class="bg-yellow-100 border-l-4 border-yellow-500 text-yellow-800 p-4 rounded-lg cursor-pointer hover:bg-yellow-200" id="start-review-btn">
                <p class="font-bold">今日の復習があります！</p>
                <p>期限が来たカードが <span class="font-bold">${allDue.length}</span> 件あります。（要復習: ${urgent.length}件 / 再確認: ${scheduled.length}件）</p>
            </div>
        `;
        document.getElementById('start-review-btn').addEventListener('click', renderReview);
        headerReviewBadge.textContent = allDue.length;
        headerReviewBadge.classList.remove('hidden');
    } else {
        reviewBannerContainer.innerHTML = '';
        headerReviewBadge.classList.add('hidden');
    }


    quizData.forEach((data, index) => {
        // Check for items to review for this sentence
        const quizId = getItemId(data.id, 'quiz');
        const cardIds = data.flashcards.map(fc => getItemId(data.id, 'card', fc.en));
        
        const isQuizDue = progress.cards[quizId] && isDue(progress.cards[quizId]);
        const isVocabDue = cardIds.some(cardId => progress.cards[cardId] && isDue(progress.cards[cardId]));
        const hasDueItems = isQuizDue || isVocabDue;

        const item = document.createElement('button');
        item.className = 'toc-item p-3 border rounded-lg text-center hover:bg-gray-100 transition-colors flex flex-col justify-center items-center h-full';
        if (hasDueItems) item.classList.add('needs-review', 'border-red-400', 'bg-red-50');
        
        let reviewMarksHTML = '<div class="flex items-center space-x-1 mt-1">';
        if (isQuizDue) reviewMarksHTML += '<span class="review-mark" title="クイズ要復習">Q</span>';
        if (isVocabDue) reviewMarksHTML += '<span class="review-mark bg-sky-500" title="単語要復習">V</span>';
        reviewMarksHTML += '</div>';

        item.innerHTML = `
            <div class="toc-item-content">
                <span class="font-semibold">例文 ${data.id}</span>
                ${hasDueItems ? reviewMarksHTML : ''}
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
    if (indices.length === 0) {
        showMainMenu();
        return;
    }
    
    // This logic correctly handles both sequential (many indices) and individual (one index) modes.
    currentSentenceIndex = indices.shift();
    quizQueue = indices;
    
    currentStep = 1;
    // For individual mode, unlock all steps. For sequential, start from step 1.
    stepUnlocked = 3; 
    tocScreen.classList.add('hidden');
    resultScreen.classList.add('hidden');
    mainQuizArea.classList.remove('hidden');
    nextBtn.classList.add('hidden');
    renderCurrentStep();
}

function handleNext() {
    currentStep++;
    stepUnlocked = Math.max(stepUnlocked, currentStep);

    if (currentStep > 3) { // After flashcards
        if (quizQueue.length > 0) { // More questions in queue
            const session = quizQueue.shift();
            currentSentenceIndex = session;
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

    [...stepIndicatorsContainer.children].forEach(el => {
        const step = parseInt(el.dataset.step, 10);
        el.classList.remove('active', 'cursor-pointer', 'hover:bg-gray-200');
        el.disabled = step > stepUnlocked;

        if (step === currentStep) {
            el.classList.add('active');
        } else if (step < stepUnlocked) {
            el.classList.add('cursor-pointer', 'hover:bg-gray-200');
        }
    });
}

function updateSentenceNav() {
    const sentenceNav = document.getElementById('sentence-nav');
    if (isSequentialMode) {
        sentenceNav.classList.add('hidden');
    } else {
        sentenceNav.classList.remove('hidden');
        prevSentenceBtn.disabled = currentSentenceIndex === 0;
        nextSentenceBtn.disabled = currentSentenceIndex === quizData.length - 1;
    }
}

function renderCurrentStep() {
    updateProgress();
    Object.values(stepViews).forEach(view => view.classList.add('hidden'));
    stepViews[currentStep].classList.remove('hidden');
    nextBtn.classList.add('hidden');
    updateSentenceNav();

    const data = quizData[currentSentenceIndex];
    if (!data) return; 

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
    const itemId = getItemId(quizData[currentSentenceIndex].id, 'quiz');
    
    if (isCorrect) {
        markCorrect(itemId);
    } else {
        markWrong(itemId);
    }
    
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
    `;

    stepViews[2].querySelector('#show-translation-btn').addEventListener('click', e => {
        document.getElementById('translation-text').classList.toggle('hidden');
        e.target.textContent = document.getElementById('translation-text').classList.contains('hidden') ? '日本語訳を見る' : '日本語訳を隠す';
    });

    // Automatically show the next button
    nextBtn.textContent = '単語カードへ →';
    nextBtn.classList.remove('hidden');
    stepUnlocked = Math.max(stepUnlocked, 3); // Unlock next step
    updateProgress(); // To enable the step 3 button
}

function renderFlashcards(container, cards, isFinalCheck = false) {
    container.innerHTML = '';
    const progress = loadProgress();

    const savedCardMode = localStorage.getItem(CARD_MODE) || 'en-ja';
    const savedHintMode = localStorage.getItem(HINT_MODE) || 'on';
    const isHintVisible = (savedCardMode === 'ja-en' && savedHintMode === 'on');

    cards.forEach(card => {
        const sentenceId = isFinalCheck ? card.sentenceId : quizData[currentSentenceIndex].id;
        const cardId = getItemId(sentenceId, 'card', card.en);
        const cardProgress = progress.cards[cardId];
        
        const cardEl = document.createElement('div');
        // We wrap the card in an outer div to hold the buttons
        cardEl.className = 'flex flex-col gap-2';
        
        const flashcardDiv = document.createElement('div');
        flashcardDiv.className = 'flashcard h-48 relative'; // Add relative positioning for the badge

        let statusBadgeHTML = '';
        if (cardProgress && isDue(cardProgress) && cardProgress.last_result === 'wrong') {
            statusBadgeHTML = '<span class="absolute top-2 left-2 text-xs bg-red-100 text-red-700 px-2 py-0.5 rounded-full z-10">要復習</span>';
        } else if (cardProgress && isDue(cardProgress)) {
            statusBadgeHTML = '<span class="absolute top-2 left-2 text-xs bg-blue-100 text-blue-700 px-2 py-0.5 rounded-full z-10">再確認</span>';
        } else if (cardProgress && cardProgress.done) {
            statusBadgeHTML = '<span class="absolute top-2 left-2 text-xs bg-green-100 text-green-700 px-2 py-0.5 rounded-full z-10">できた</span>';
        }

        const frontContent = (savedCardMode === 'en-ja') 
            ? `<h3 class="text-2xl font-bold">${card.en}</h3>`
            : `<h3 class="text-xl font-bold">${card.ja}</h3>${isHintVisible ? `<p class="text-gray-500 mt-2">ヒント: ${card.hint}</p>` : ''}`;
        
        const backContent = (savedCardMode === 'en-ja')
            ? `<h3 class="text-xl font-bold">${card.ja}</h3>`
            : `<h3 class="text-2xl font-bold">${card.en}</h3>`;

        flashcardDiv.innerHTML = `
            ${statusBadgeHTML}
            <div class="flashcard-inner">
                <div class="flashcard-front">${frontContent}<button class="speaker-btn"><i class="fas fa-volume-up"></i></button></div>
                <div class="flashcard-back">
                    ${backContent}
                    <div class="text-center mt-2">
                        <p class="text-gray-500">${card.kana.replace(/\*\*(.*?)\*\*/g, '<b>$1</b>')}</p>
                        <p class="text-gray-400 text-sm">${card.phonetic}</p>
                    </div>
                    <button class="speaker-btn"><i class="fas fa-volume-up"></i></button>
                </div>
            </div>
        `;
        flashcardDiv.addEventListener('click', () => flashcardDiv.classList.toggle('flipped'));
        // Add speaker functionality to both buttons
        flashcardDiv.querySelectorAll('.speaker-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                e.stopPropagation(); // Prevent card from flipping when clicking button
                speak(card.en);
            });
        });
        
        const buttonContainer = document.createElement('div');
        buttonContainer.className = 'grid grid-cols-2 gap-2';
        
        const correctBtn = document.createElement('button');
        correctBtn.textContent = 'できた';
        correctBtn.className = 'w-full py-2 px-4 bg-green-500 text-white rounded-lg hover:bg-green-600 transition-colors';
        correctBtn.onclick = (e) => {
            e.stopPropagation();
            markCorrect(cardId);
            if (isFinalCheck) {
                renderFinalCheck(); // Re-render the whole final check screen
            } else {
                renderCurrentStep(); // Re-render to update badge
            }
        };

        const wrongBtn = document.createElement('button');
        wrongBtn.textContent = 'あとで復習';
        wrongBtn.className = 'w-full py-2 px-4 bg-white border border-gray-300 text-gray-700 rounded-lg hover:bg-gray-100 transition-colors';
        wrongBtn.onclick = (e) => {
            e.stopPropagation();
            markWrong(cardId);
            if (isFinalCheck) {
                renderFinalCheck(); // Re-render the whole final check screen
            } else {
                renderCurrentStep(); // Re-render to update badge
            }
        };

        buttonContainer.append(wrongBtn, correctBtn);
        cardEl.append(flashcardDiv, buttonContainer);
        container.appendChild(cardEl);
    });
}

function renderStep3(data) {
    const { flashcards } = data;
    
    stepViews[3].innerHTML = `
        <p class="text-sm text-gray-500 text-center mb-4">カードをタップして関連単語も覚えましょう。</p>
        <div id="flashcard-container-step3" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
    `;

    const container = document.getElementById('flashcard-container-step3');
    renderFlashcards(container, flashcards);

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
    syncAppSettings(); // Sync description text based on mode

    const container = document.getElementById('final-check-flashcards');
    renderFlashcards(container, finalCheckCards, true);
}

function showResult() {
    mainQuizArea.classList.add('hidden');
    resultScreen.classList.remove('hidden');
    
    const summaryEl = document.getElementById('self-assessment-summary');
    summaryEl.innerHTML = `
        <p class="text-xl">お疲れ様でした！</p>
        <p class="text-gray-600 mt-2">間違えた問題や「あとで復習」を選んだ問題は、目次画面から再度挑戦できます。</p>
    `;
}

// --- [FROM D3.0] Full Review Session Logic ---
function renderReview() {
    tocScreen.classList.add('hidden');
    mainQuizArea.classList.add('hidden');
    finalCheckArea.classList.add('hidden');
    reviewArea.classList.remove('hidden');

    const progress = loadProgress();
    const allDue = Object.keys(progress.cards).filter(id => isDue(progress.cards[id]));
    const urgent = allDue.filter(id => progress.cards[id].last_result === 'wrong').sort((a,b) => (progress.cards[a].next_due || 0) - (progress.cards[b].next_due || 0));
    const scheduled = allDue.filter(id => progress.cards[id].last_result !== 'wrong').sort((a,b) => (progress.cards[a].next_due || 0) - (progress.cards[b].next_due || 0));
    
    const reviewQueue = [...urgent, ...scheduled];
    let currentIndex = 0;

    function findDataById(itemId) {
        const parts = itemId.split('-');
        const sentenceId = parseInt(parts[0].substring(1), 10);
        const type = parts[1];
        const sentenceData = quizData.find(d => d.id === sentenceId);
        if (!sentenceData) return null;

        if (type === 'quiz') {
            return { type, sentence: sentenceData };
        }
        if (type === 'card') {
            const identifier = parts.slice(2).join('-');
            const cardData = sentenceData.flashcards.find(fc => fc.en.toLowerCase().replace(/[^a-z0-9]/g, '_').substring(0, 20) === identifier);
            return { type, sentence: sentenceData, card: cardData };
        }
        return null; // Should not happen
    }

    function renderNextItem() {
        if (currentIndex >= reviewQueue.length) {
            reviewArea.innerHTML = `
                <div class="text-center">
                    <h2 class="text-2xl font-bold text-gray-800 mb-4">復習完了！</h2>
                    <p class="text-gray-600 mb-6">お疲れ様でした。すべての復習項目が終わりました。</p>
                    <button id="review-back-to-toc" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-6 rounded-full">目次に戻る</button>
                </div>
            `;
            document.getElementById('review-back-to-toc').addEventListener('click', showMainMenu);
            return;
        }

        const itemId = reviewQueue[currentIndex];
        const itemData = findDataById(itemId);

        if (!itemData || (itemData.type === 'card' && !itemData.card)) {
            console.warn(`Could not find data for review item: ${itemId}. Skipping.`);
            currentIndex++;
            renderNextItem();
            return;
        }

        reviewArea.innerHTML = `
            <div class="w-full bg-gray-200 rounded-full h-2.5 mb-6">
                <div class="bg-blue-500 h-2.5 rounded-full" style="width: ${((currentIndex) / reviewQueue.length) * 100}%; transition: width 0.3s;"></div>
            </div>
            <div id="review-item-content"></div>
        `;
        const contentEl = document.getElementById('review-item-content');

        const advance = () => {
            currentIndex++;
            // Give a short delay to see the result before the next item renders
            setTimeout(renderNextItem, 600);
        };

        if (itemData.type === 'quiz') {
            const { sentence } = itemData;
            const { quiz } = sentence;
            const questionText = sentence.originalSentence.replace(new RegExp(quiz.target.replace(/([.*+?^=!:${}()|\[\]\/\\])/g, "\\$1"), 'i'), '______');
            const shuffledChoices = [...quiz.choices].sort(() => Math.random() - 0.5);
            let optionsHTML = shuffledChoices.map(choice => 
                `<button class="btn-option w-full p-3 bg-white border border-gray-300 rounded-lg text-left text-gray-700 font-medium">${choice}</button>`
            ).join('');

            contentEl.innerHTML = `
                <p class="text-sm text-gray-500 mb-2">例文 ${sentence.id} - 空所補充</p>
                <div class="mb-4 relative">
                    <p class="text-lg text-gray-800 text-center p-4 bg-gray-50 rounded-lg">${questionText}</p>
                    <button id="review-quiz-tts-btn" class="speaker-btn"><i class="fas fa-volume-up"></i></button>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3" id="review-options">
                    ${optionsHTML}
                </div>
            `;
            contentEl.querySelector('#review-quiz-tts-btn').addEventListener('click', () => speak(sentence.originalSentence));

            contentEl.querySelector('#review-options').addEventListener('click', (e) => {
                if (e.target.tagName === 'BUTTON') {
                    const isCorrect = e.target.innerText.toLowerCase() === quiz.target.toLowerCase();
                    if (isCorrect) markCorrect(itemId); else markWrong(itemId);
                    
                    Array.from(contentEl.querySelector('#review-options').children).forEach(button => {
                        button.disabled = true;
                        if (button.innerText.toLowerCase() === quiz.target.toLowerCase()) button.classList.add('!bg-green-200', '!border-green-500');
                        else if (button === e.target) button.classList.add('!bg-red-200', '!border-red-500');
                    });
                    advance();
                }
            });
        } else if (itemData.type === 'card') {
            const { card } = itemData;
            const cardContainer = document.createElement('div');
            // Temporarily set currentSentenceIndex for renderFlashcards to work
            const originalIndex = currentSentenceIndex;
            currentSentenceIndex = quizData.findIndex(d => d.id === itemData.sentence.id);
            renderFlashcards(cardContainer, [card]); 
            currentSentenceIndex = originalIndex; // Restore it

            contentEl.innerHTML = `<p class="text-sm text-gray-500 mb-2">例文 ${itemData.sentence.id} - 単語カード</p>`;
            contentEl.appendChild(cardContainer);

            // Override card buttons for review context
            const correctBtn = cardContainer.querySelector('.bg-green-500');
            const wrongBtn = cardContainer.querySelector('.border-gray-300');
            if (correctBtn) correctBtn.onclick = (e) => { e.stopPropagation(); markCorrect(itemId); correctBtn.style.opacity = '1'; wrongBtn.style.opacity = '0.5'; advance(); };
            if (wrongBtn) wrongBtn.onclick = (e) => { e.stopPropagation(); markWrong(itemId); wrongBtn.style.opacity = '1'; correctBtn.style.opacity = '0.5'; advance(); };
        }
    }

    if (reviewQueue.length > 0) {
        renderNextItem();
    } else {
        reviewArea.innerHTML = `
             <div class="text-center">
                <h2 class="text-2xl font-bold text-gray-800 mb-4">今日の復習はありません</h2>
                <p class="text-gray-600 mb-6">素晴らしいです！復習が必要な項目は今のところありません。</p>
                <button id="review-back-to-toc" class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-6 rounded-full">目次に戻る</button>
            </div>
        `;
        document.getElementById('review-back-to-toc').addEventListener('click', showMainMenu);
    }
}
</script>
