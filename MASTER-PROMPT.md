# Master Prompt — Personal Chinese Trainer App

> Reusable build spec. Give this prompt to an AI assistant (or future me) to (re)build or extend the app. Every requirement is testable; the QA checklist at the end is mandatory.

## Role

You are an expert in CEFR/HSK-based language pedagogy and front-end engineering. Build a **single-file HTML app** (`chinese-trainer.html`) for a learner at ~HSK 1 (100–300 words) who practices **45+ min daily** and wants all skills trained: listening, speaking (shadowing), reading, handwriting, and typing.

## Hard constraints

1. **One file**, no build step, double-click to open in Edge/Chrome on Windows. CSS+JS inline; the only external dependencies are CDN scripts: `hanzi-writer` (stroke order) and `opencc-js` (simplified→traditional conversion). Both must degrade gracefully offline.
2. **No accounts, no servers.** All progress in `localStorage` under a versioned key (`zhTrainerV2`). Migrate progress from any prior key on first run.
3. **Audio = Web Speech API** (`speechSynthesis`), `zh-CN` voice preferred (zh-TW/zh-HK when traditional script selected). Adjustable rate; a "slow replay" everywhere audio appears.
4. **Embedded data**, no fetching word lists: ≥250 HSK 1–2 words `[simplified, pinyin (tone marks), english, level]` plus ≥20 natural HSK1-level sentences. No duplicates. Traditional forms are derived at runtime via OpenCC (never hand-typed — too error-prone).

## Script & pinyin support (core requirement)

- **Script toggle (简/繁)** in the header, instantly switching ALL hanzi everywhere: flashcards, listening feedback, writing practice, typing answers, word list. Memoize OpenCC conversions; fall back to simplified if the CDN is unreachable.
- **Pinyin display modes**: always show / only after reveal. Setting applies to flashcards.
- **Tone colors** (toggleable, Pleco scheme: 1 red, 2 green, 3 blue, 4 purple, neutral gray). Implement a pinyin syllable splitter (regex: optional initial + vowel cluster + optional n/ng/r coda) and a tone detector from diacritics. Minor splitter edge cases are acceptable.
- **Pinyin Lab tab**: the four tones demonstrated with 妈/麻/马/骂/吗 + audio, a tone-ear trainer (hear a word, identify the first syllable's tone, colored-pinyin feedback), and short practical tips (ü, tone-3 sandhi, 不/一 tone changes).
- **Typing accepts**: toneless pinyin (`xuexi`), tone numbers (`xue2xi2`, neutral as 5 or omitted), `v` or `ü`; spaces/apostrophes ignored.

## Modules (tabs)

1. **Today** — stats (due / new / learned / total), 45-min plan checklist with per-task done counters, streak (🔥, consecutive days with ≥1 exercise), settings (new words/day, audio speed, script, card front, pinyin display, tone colors).
2. **Flashcards (SRS)** — simplified SM-2: Again (5 min, lapse), Hard (×1.2), Good (×2.3, first interval 1 d), Easy (×3.2, first 3 d). New cards capped per day; new cards show full answer immediately. **Card front modes**: hanzi → meaning, audio → meaning, English → hanzi (recall). Keyboard: Space reveal, 1–4 grade. Audio autoplays on reveal.
3. **Listening** — word and sentence modes; play/slow buttons; 4-option multiple choice (distractors from learned pool); reveal shows hanzi + colored pinyin; running score.
4. **Writing** — Hanzi Writer per character of the chosen word (random-from-learned or typed in); watch strokes / trace with outline / from memory (hints after 2 misses). Respects script toggle.
5. **Typing** — prompt = English + audio; pinyin mode (rules above) or hanzi mode (user's IME, exact match after stripping punctuation); feedback with colored pinyin; running score.
6. **Word list** — searchable (hanzi / pinyin with or without tones / english), shows learned ✅, HSK level, per-word audio.

## Pedagogy rules

- Recognition before recall; default 8 new words/day.
- Distractors only from words the learner has seen (fallback: HSK1 pool).
- Audio everywhere — every word/sentence displayed should be one click from being heard.
- Encourage shadowing: after listening answers, the learner should replay and repeat aloud.

## UI

Dark theme, CJK font stack (`Microsoft YaHei`, `Noto Sans SC`), large hanzi (≥60 px on cards), responsive ≤600 px, no scroll traps. Chinese-red accent. Cheerful but minimal; no gamification beyond streak + scores.

## Mandatory QA before delivery

1. `node --check` on the extracted script.
2. Headless smoke test (stub `localStorage`, `speechSynthesis`, `document`, `HanziWriter`, `OpenCC`): data integrity (counts, no duplicate hanzi+meaning), `plainPinyin`, syllable splitter + tone detection on tricky cases (`xuéxí`, `nǚ'ér`, `Běijīng`, `xiànzài`), numbered-pinyin matching (`xue2xi2`), SRS grade scheduling, streak increment, v1→v2 migration, T() fallback when OpenCC is absent.
3. Confirm every tab renders without throwing when state is empty (fresh user) and when populated.

## v3 — "Coach" tier (current build: `chinese-coach.html`)

The learner: gaming-industry product marketer & writer (ex-Acer Predator), job-hunting; wants a structured **60-minute daily session**. v3 adds on top of everything above:

1. **Guided session engine** — one button runs 8 timed blocks (warm-up reviews 9' → new words 7' → listening 7' → dialogue 9' → games 7' → speaking 8' → handwriting 6' → boss quiz 7') with a sticky countdown bar, block skip, and an end-of-session report (minutes, XP, per-activity counts). Every block is also a standalone tab.
2. **Work vocab pack (~65 words)** — marketing, gaming/esports, hardware, job-interview terms; tagged `"P"`, shown with a WORK badge, interleaved ~1-in-4 into new flashcards (toggleable).
3. **Dialogues (6 scenarios)** — launch meeting, game-expo booth, **job interview**, milk-tea order, ad-copy review, new-PC chat. Chat-bubble UI, per-line audio, play-all (chained TTS via `onend`), step-through for role-play, pinyin/english toggles, comprehension MCQs (index-based answers).
4. **Speaking studio** — Web Speech API `SpeechRecognition` (zh-CN/zh-TW); learner hears a sentence, speaks it; score = LCS(target, transcript)/len, per-character green/red feedback, blur-to-hide text mode. Degrade gracefully if unsupported.
5. **Sentence builder** — segmented word tiles to arrange (bank of 20, incl. work sentences); index-based tap/remove, audio hint, auto-speak on success.
6. **Number dojo** — programmatic 数字 generator (`numHanzi` for 0–9999 incl. 零-insertion and 十 edge cases), three modes: plain numbers, prices (块), clock times (点/分); type-the-digits answers.
7. **Boss quiz** — 10 mixed questions (audio-word→EN, EN→hanzi, tone-ID, audio-sentence→EN) with XP bonus.
8. **Progression** — XP per exercise (map per type), levels (`floor(sqrt(xp/50))+1`), header HUD, XP bar, **84-day GitHub-style heatmap**.
9. State key `zhTrainerV3`, migrates V2/V1 (card indices stay valid because the core word array order never changes — only append).

### Additional QA for v3
`numHanzi` edge cases (0, 10, 12, 110, 105, 1001, 9999) · boss-quiz generation never throws on fresh state · session engine block transitions and report · LCS scorer · all 11 tabs render empty & populated · dialogue MCQ index answers correct.

### Hard-won build lessons
- **Never put a raw backtick inside a template literal** (broke v2's tone button). v3 uses string concatenation for all HTML-building to eliminate the class of bug.
- MCQ answers must be **index-based**, never compared by `textContent` (apostrophes/escaping).
- When overwriting a file, the sandbox mount may serve a stale copy — QA from a fresh-named copy if needed.

## v4 — Taiwan Edition (current build: `chinese-coach.html`)

Learner profile sharpened by resume: **Clark, Taipei-based, 10 yrs at Acer (Head Gaming Copywriter, Predator/Nitro, Predator Atlas 8 handheld @ Computex), job-hunting in Taiwan.** v4 reorients everything to Taiwan:

1. **Taiwan-first defaults** — traditional script (繁體) + zh-TW voice by default; 台灣用語 toggle.
2. **Lexical TW variants** — word entries gain `{tw:[hanzi,pinyin], n:note}`: 計程車/出租車, 筆電/笔记本, 滑鼠/鼠标, 螢幕/屏幕, 影片/视频, 行銷/营销, 履歷/简历, 薪水/工资, 不會/不客气, 哪裡/哪儿, 禮拜/星期, 中文/汉语, 社群媒體/社交媒体, 發表會/发布会, 網路上/网上, 目標族群/受众, 成長/增长. Variants shown by default; mainland form in a note. Typing accepts both.
3. **Taiwan life pack (26 words, level "T")** — 捷運, 機車, 夜市, 珍珠奶茶, 悠遊卡, 滷肉飯, 颱風, 老闆, 加班, 名片, 紅包, 過年, 中秋節, 拜拜, 不好意思, 掌機, 電玩展, 萬…
4. **Cultural notes** (~60 words) — lucky/unlucky numbers, gift taboos, red-ink taboo, KTV, 帥哥 vendor talk, two-handed name cards, NT$ pricing, 注音 Bopomofo awareness, Mid-Autumn BBQ marketing story.
5. **Radical system** — 30-radical glossary (glyph, meaning, image-story, deck examples) + 145 per-character mnemonic stories (traditional-aware, e.g. 聽 ear+heart, 愛 keeps the heart, 電 rain+lightning, 筆 bamboo+hair). Shown on flashcard reveals, word-list detail, writing tab. New **Radicals tab**: learn mode + which-radical-hides-inside quiz (distractor guard: exclude radicals visually present in the char in either script).
6. **Dialogues TW-ized + resume-tailored** — launch meeting in 台北 (NT$ prices), 台北電玩展 booth (掌機), **job interview using Clark's real story** (10 yrs Taipei, Acer, 產品行銷+文案, 20+ launches/yr), night-market run (悠遊卡, 半糖少冰), copy review, new rig. No erhua anywhere.
7. Session = 9 blocks (radical recognition added, 5 min). State key `zhTrainerV4`, migrates V1–V3 (core word order unchanged; script forced to 't' once).

### Additional QA for v4
TW accessor toggling (wHanzi/wPin) · radical quiz x200 distractor-containment guard · every quiz example char has a CHARRAD story · typing matches both TW and mainland forms · extras schema validation · all 12 tabs render.

### More hard-won lessons
- The sandbox mount can serve **inconsistent partial states** after overwrites/edits (grep sees new content while size/tail are old). Never `cp` from the mount right after an edit — verify size + tail + extraction all agree first, or repair via the file tools, which always see the real file.

## v5 — Forgebound layer + logs (current build, patched in place — never rebuilt)

1. **Daily Lesson Log** — every session saves {date, mode, minutes, XP, topic/dialogue, new words, characters practiced, weak points, review-tomorrow}; last entry's review items surface on Home.
2. **Mistake Log** — `recordMiss(kind, wordIdx, label)` hooked into cards/listening/tones/typing/numbers/boss/dialogues/radicals/handwriting (HanziWriter `onMistake`, >2 stroke errors = miss). **Review Weak Points** = top-20 missed words as a flashcard queue.
3. **Personal Sentence Bank (Core ᛟ)** — 46 built-ins across 9 personal categories (daily, work/job hunt, marketing, gaming/AI PC, fitness, Taiwan, social, growth, VARGR/Forgebound) + add/delete your own; every sentence: trad (via T), pinyin, English, category, ★difficulty, audio, mic practice.
4. **Career Mandarin (Hunt ᚨ)** — 8 sections, 41 lines: self-intro, interview answers, recruiter/LinkedIn (訊息/線上), freelance pitch (接案), Predator-style taglines, laptop specs (記憶體/硬碟/解析度 TW forms), AI PC terms (人工智慧/晶片), explaining the Acer decade. Each line: 🔊/🐢/🎤 (mic hands off to Voice studio via `practiceItem`+`spKeep`).
5. **Session modes** — Full 60 (9 blocks) / Minimum 30 (6) / Survival 10 (3); chosen on Home, block lists derived from one SBLOCKS source.
6. **Export/Import** — JSON full-state backup + restore (validated), CSV for vocab (incl. TW forms + miss counts) and sentence bank. BOM-prefixed CSV for Excel.
7. **VARGR/Forgebound UI** — rune-glyph nav only (ᚹ Voice·ᛖ Echo·ᛊ Script·ᚺ Hand·ᚠ Forge·ᛟ Core·ᚨ Hunt·ᚷ Gate·ᚱ Runes·ᛞ Archive; Circuit/Pulse as categories inside Hunt/Core), legend in Archive. Hanzi rendering untouched.
8. Same storage key (`zhTrainerV4`) — defaults-merge adds new fields; no migration risk.

### v5 QA learnings
- When the sandbox mount won't refresh an overwritten file, **mirror the patch**: apply the identical old→new replacements to the last verified script copy in /tmp (assert each anchor count==1 — this simultaneously proves the real file's edits hit unique anchors) and run node --check + logic tests on the mirror.
- Pass replacement strings to patch scripts as **raw text files**, never escaped heredocs.

## Extension roadmap (future sessions)

HSK 3 vocab pack · example sentence per word · more dialogue packs (salary negotiation, 104 phone screen, Computex booth pitch) · 注音 Bopomofo mode · weekly review report generated from lessonLog · streak freeze item · SRS for sentence bank.
