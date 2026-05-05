<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Анализатор уровня английских слов (CEFR A1–C2)</title>
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
            background: #f0f4f8;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 24px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.1);
            padding: 24px;
        }
        h1 {
            font-size: 1.8rem;
            margin-top: 0;
            color: #1e3c72;
        }
        .sub {
            color: #2c5282;
            margin-bottom: 20px;
            border-left: 4px solid #3182ce;
            padding-left: 12px;
        }
        textarea {
            width: 100%;
            padding: 16px;
            font-size: 16px;
            font-family: monospace;
            border: 2px solid #cbd5e0;
            border-radius: 16px;
            resize: vertical;
            background: #fefefe;
        }
        button {
            background: #2b6cb0;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 18px;
            font-weight: bold;
            border-radius: 40px;
            margin: 16px 0;
            cursor: pointer;
            transition: 0.2s;
        }
        button:hover {
            background: #2c5282;
            transform: scale(1.02);
        }
        .stats {
            display: flex;
            flex-wrap: wrap;
            gap: 12px;
            margin: 20px 0;
            background: #edf2f7;
            padding: 16px;
            border-radius: 20px;
        }
        .stat-card {
            background: white;
            border-radius: 16px;
            padding: 8px 16px;
            min-width: 80px;
            text-align: center;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
        .stat-level {
            font-weight: bold;
            font-size: 1.2rem;
        }
        .stat-count {
            font-size: 1.8rem;
            font-weight: 800;
        }
        .result-area {
            background: #f9fafb;
            border-radius: 20px;
            padding: 20px;
            border: 1px solid #e2e8f0;
            margin-top: 16px;
            line-height: 1.7;
        }
        .word-span {
            display: inline-block;
            margin: 2px 1px;
            padding: 2px 5px;
            border-radius: 12px;
            font-size: 1rem;
            transition: 0.1s;
        }
        .A1 { background-color: #c6f7d0; border-bottom: 2px solid #2b8c4a; }
        .A2 { background-color: #b8e4f0; border-bottom: 2px solid #1e6f8c; }
        .B1 { background-color: #ffe0a3; border-bottom: 2px solid #b87c00; }
        .B2 { background-color: #ffc9b3; border-bottom: 2px solid #c23b0e; }
        .C1 { background-color: #f2c1ff; border-bottom: 2px solid #8b2fc9; }
        .C2 { background-color: #fbc4c4; border-bottom: 2px solid #b91c1c; }
        .unknown { background-color: #e2e8f0; border-bottom: 2px solid #718096; }
        footer {
            margin-top: 24px;
            text-align: center;
            font-size: 0.8rem;
            color: #4a5568;
        }
        @media (max-width: 640px) {
            .stat-card { min-width: 60px; }
            .stat-count { font-size: 1.2rem; }
        }
    </style>
</head>
<body>
<div class="container">
    <h1>📊 Анализатор уровня английских слов</h1>
    <div class="sub">Определяет уровень CEFR (A1 — начальный, C2 — профессиональный) для каждого слова в тексте.<br>
    📚 Словарь загружается из JSON (~15 тыс. слов). Первый анализ может занять долю секунды.</div>
    
    <textarea id="inputText" rows="8" placeholder="Вставьте текст на английском языке...&#10;Например: I can run fast, but she abandoned the idea."></textarea>
    <div><button id="analyzeBtn">🔍 Анализировать текст</button></div>
    
    <div id="statsPanel" class="stats" style="display: none;"></div>
    
    <h3>📝 Результат с подсветкой уровней:</h3>
    <div id="resultArea" class="result-area">Загрузка словаря... Пожалуйста, подождите.</div>
    <footer>⚡ Словарь CEFR-J (около 15000 слов). Уровни по шкале CEFR. Неизвестные слова — серым цветом.</footer>
</div>

<script>
    // Глобальный словарь уровней (будет заполнен из JSON)
    let wordLevels = {};
    
    // Карта неправильных глаголов (оставляем для базовой лемматизации)
    const irregularMap = {
        "went": "go", "gone": "go", "done": "do", "did": "do", "made": "make",
        "came": "come", "took": "take", "taken": "take", "given": "give", "seen": "see",
        "ate": "eat", "eaten": "eat", "drank": "drink", "drunk": "drink", "spoke": "speak",
        "spoken": "speak", "wrote": "write", "written": "write", "knew": "know", "known": "know",
        "thought": "think", "bought": "buy", "brought": "bring", "caught": "catch",
        "taught": "teach", "fought": "fight", "found": "find", "heard": "hear",
        "held": "hold", "kept": "keep", "left": "leave", "lost": "lose", "met": "meet",
        "paid": "pay", "ran": "run", "said": "say", "sold": "sell", "sat": "sit",
        "slept": "sleep", "stood": "stand", "told": "tell", "understood": "understand",
        "woke": "wake", "worn": "wear", "won": "win"
    };
    
    // Функция загрузки JSON-словаря
    async function loadDictionary() {
        try {
            const response = await fetch('cefrj-vocabulary-profile-1.5.json');
            if (!response.ok) throw new Error(`HTTP ${response.status}`);
            const data = await response.json();
            
            // Очищаем старый словарь
            wordLevels = {};
            let added = 0;
            for (const entry of data) {
                const word = entry.headword.toLowerCase();
                const level = entry.CEFR;
                if (level && ['A1','A2','B1','B2','C1','C2'].includes(level)) {
                    // Не перезаписываем, если слово уже есть (оставляем первый уровень)
                    if (!wordLevels[word]) {
                        wordLevels[word] = level;
                        added++;
                    }
                }
            }
            console.log(`Словарь загружен: ${added} уникальных слов с уровнями.`);
            document.getElementById('resultArea').innerHTML = 'Словарь загружен. Введите текст и нажмите "Анализировать".';
            // Если в текстовом поле уже есть текст, можно автоматически проанализировать
            const textarea = document.getElementById('inputText');
            if (textarea.value.trim() !== "") {
                document.getElementById('analyzeBtn').click();
            }
        } catch (error) {
            console.error('Ошибка загрузки словаря:', error);
            document.getElementById('resultArea').innerHTML = 'Не удалось загрузить словарь. Убедитесь, что файл cefrj-vocabulary-profile-1.5.json находится в той же папке, и повторите попытку.';
        }
    }
    
    // Базовая лемматизация (модифицирована для работы с загруженным словарём)
    function simpleLemmatize(word) {
        let lower = word.toLowerCase();
        // 1. Проверка неправильных глаголов
        if (irregularMap[lower]) return irregularMap[lower];
        
        // 2. Множественное число существительных (простые правила)
        if (lower.endsWith("ies") && !lower.endsWith("eies")) {
            let cand = lower.slice(0, -3) + "y";
            if (wordLevels[cand]) return cand;
        }
        if (lower.endsWith("es") && !(lower.endsWith("sses") || lower.endsWith("ches") || lower.endsWith("shes"))) {
            let cand = lower.slice(0, -2);
            if (wordLevels[cand]) return cand;
        }
        if (lower.endsWith("s") && !lower.endsWith("ss") && !lower.endsWith("us") && !lower.endsWith("is")) {
            let cand = lower.slice(0, -1);
            if (wordLevels[cand]) return cand;
        }
        
        // 3. -ing форма
        if (lower.endsWith("ing")) {
            let stem = lower.slice(0, -3);
            if (wordLevels[stem]) return stem;
            if (stem.endsWith("e")) {
                let noE = stem.slice(0, -1);
                if (wordLevels[noE]) return noE;
            }
            // удвоение согласной: running -> run
            if (stem.length > 1 && stem[stem.length-1] === stem[stem.length-2]) {
                let cand = stem.slice(0, -1);
                if (wordLevels[cand]) return cand;
            }
        }
        
        // 4. -ed окончание
        if (lower.endsWith("ed")) {
            let stem = lower.slice(0, -2);
            if (wordLevels[stem]) return stem;
            if (stem.endsWith("e")) {
                let noE = stem.slice(0, -1);
                if (wordLevels[noE]) return noE;
            }
        }
        
        // Возвращаем исходное слово, если ни одно правило не сработало
        return lower;
    }
    
    // Функция анализа текста
    function analyzeText(text) {
        const tokens = text.match(/\b[\w']+(?:-\w+)*\b|\S/g) || [];
        const resultTokens = [];
        let stats = { "A1":0, "A2":0, "B1":0, "B2":0, "C1":0, "C2":0, "unknown":0 };
        
        for (let token of tokens) {
            if (/^[a-zA-Z']+$/.test(token)) {
                let lemma = simpleLemmatize(token);
                let level = wordLevels[lemma] || "unknown";
                if (level !== "unknown") stats[level]++;
                else stats.unknown++;
                resultTokens.push({ text: token, level: level, lemma: lemma });
            } else {
                resultTokens.push({ text: token, level: null });
            }
        }
        return { tokens: resultTokens, stats: stats };
    }
    
    // Рендер раскрашенного текста
    function renderResult(tokens) {
        let html = '';
        for (let t of tokens) {
            if (t.level === null) {
                html += t.text;
            } else {
                let levelClass = t.level;
                let title = `Уровень: ${t.level} (лемма: ${t.lemma || t.text.toLowerCase()})`;
                html += `<span class="word-span ${levelClass}" title="${title}">${escapeHtml(t.text)}</span>`;
            }
        }
        return html;
    }
    
    function escapeHtml(str) {
        return str.replace(/[&<>]/g, function(m) {
            if (m === '&') return '&amp;';
            if (m === '<') return '&lt;';
            if (m === '>') return '&gt;';
            return m;
        });
    }
    
    function displayStats(stats) {
        const levels = ['A1','A2','B1','B2','C1','C2','unknown'];
        const levelNames = {'A1':'Начальный','A2':'Элементарный','B1':'Средний','B2':'Выше среднего','C1':'Продвинутый','C2':'Профессиональный','unknown':'Неизвестный'};
        let html = '';
        for (let lvl of levels) {
            let count = stats[lvl] || 0;
            html += `
                <div class="stat-card">
                    <div class="stat-level" style="color:${lvl==='unknown'?'#718096':'#2d3748'}">${lvl} ${levelNames[lvl]}</div>
                    <div class="stat-count">${count}</div>
                </div>
            `;
        }
        return html;
    }
    
    // Обработчик кнопки
    document.getElementById('analyzeBtn').addEventListener('click', () => {
        const textarea = document.getElementById('inputText');
        const text = textarea.value;
        if (!text.trim()) {
            alert('Пожалуйста, введите текст для анализа.');
            return;
        }
        if (Object.keys(wordLevels).length === 0) {
            alert('Словарь ещё не загружен. Пожалуйста, подождите или обновите страницу.');
            return;
        }
        const { tokens, stats } = analyzeText(text);
        const coloredHtml = renderResult(tokens);
        document.getElementById('resultArea').innerHTML = coloredHtml;
        const statsHtml = displayStats(stats);
        const statsPanel = document.getElementById('statsPanel');
        statsPanel.innerHTML = statsHtml;
        statsPanel.style.display = 'flex';
    });
    
    // Загрузка словаря при старте
    loadDictionary();
    
    // Демо текст для примера (можно оставить, но не автоанализировать до загрузки словаря)
    window.addEventListener('load', () => {
        const demoText = "I can run fast and swim well. However, she abandoned the project because it was too difficult. The weather is beautiful today.";
        document.getElementById('inputText').value = demoText;
        // Не вызываем анализ автоматически, дождёмся загрузки словаря
        // loadDictionary сам вызовет анализ, если текст не пуст.
    });
</script>
</body>
</html>
