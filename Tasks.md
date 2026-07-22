 📋 Пул задач

### 📌 Глобальный Бэклог (Без дедлайна)
- [ ] Обсудить с мамой обследование в Екб 
- [ ] проанализировать падение  

---
### ⏳ Забытое в ежедневниках (Долги)
```dataview
TASK
WHERE contains(file.name, "-W")
WHERE !completed
WHERE section != null
FLATTEN date(substring(meta(section).subpath, length(meta(section).subpath) - 10)) AS taskDate
WHERE taskDate <= date(today)
WHERE dateformat(taskDate, "kkkk-WW") = dateformat(date(today), "kkkk-WW")
GROUP BY file.link
```

### 🧠 Запылившиеся алгоритмы (Давно не вызывались)
```dataview
TABLE task_num as "Задания", (date(today) - last_check).days as "Дней простоя"
FROM "03_Knowledge/ЕГЭ/Русский/Номера"
WHERE type = "rus_schema"
SORT (date(today) - last_check).days DESC
LIMIT 1
```

```dataviewjs
const filePath = "03_Knowledge/ЕГЭ/Русский/00_Словник_Ударения.md";
const tFile = app.vault.getAbstractFileByPath(filePath);
if (!tFile) return dv.paragraph("❌ Файл не найден. Проверь путь.");

const today = window.moment().format("YYYY-MM-DD");
const page = dv.page(filePath);
if (!page) return;

const allTasks = page.file.tasks;
const openTasks = allTasks.filter(t => !t.completed);

// === 🏆 СЦЕНАРИЙ АБСОЛЮТНОЙ ПОБЕДЫ ===
if (allTasks.length > 0 && openTasks.length === 0) {
    const winDiv = document.createElement("div");
    winDiv.innerHTML = `
        <div style="padding: 24px; background: linear-gradient(135deg, #F59E0B 0%, #D97706 100%); color: white; border-radius: 12px; text-align: center; box-shadow: 0 10px 25px -5px rgba(245, 158, 11, 0.4); margin-bottom: 20px;">
            <h2 style="margin: 0 0 10px 0; color: white; font-size: 24px;">🏆 БАЗА ФИПИ ВЫУЧЕНА НА 100%!</h2>
            <p style="margin: 0 0 20px 0; font-size: 15px; opacity: 0.95;">Ты железобетонно закрыл все ${allTasks.length} слов. Твой процессор великолепен.</p>
            <button id="ng-plus-btn-4" style="background: white; color: #D97706; border: none; padding: 12px 24px; font-size: 16px; font-weight: bold; border-radius: 8px; cursor: pointer; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
                🔄 ИДТИ НА 2-Й КРУГ (Сбросить все галочки)
            </button>
        </div>
    `;
    dv.container.appendChild(winDiv);

    document.getElementById("ng-plus-btn-4").onclick = async () => {
        await app.vault.process(tFile, data => data.replace(/- \[x\]/g, "- [ ]"));
    };
    return;
}

const limit = 20;
const processedToday = allTasks.filter(t => t.text.includes(`🗓️${today}`)).length;
const remain = limit - processedToday;

// === ⏳ СЦЕНАРИЙ СУТОЧНОЙ НОРМЫ ===
if (remain <= 0) {
    dv.el("div", `<div style="padding:16px; background:#10B981; color:white; border-radius:8px; text-align:center; font-size: 15px; font-weight: 500; margin-bottom: 20px;">
    🎉 <b>НОРМА УДАРЕНИЙ ВЫПОЛНЕНА!</b><br>
    <span style="font-size: 12px; opacity: 0.9;">Ты разобрал ${limit} слов. Возвращайся завтра в 00:01.</span>
    </div>`);
    return;
}

dv.header(4, `🎯 Ударения (Осталось на сегодня: ${remain})`);

let batchPool = Array.from(openTasks.filter(t => !t.text.includes(`🗓️${today}`)));

// === 🎲 СТАБИЛЬНЫЙ РАНДОМ (Хэш-функция) ===
function getHash(text) {
    const clean = text.replace(/\[fails:: \d+\]/g, "").replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "").trim();
    const str = clean + today; 
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
        hash = ((hash << 5) - hash) + str.charCodeAt(i);
        hash |= 0;
    }
    return hash;
}

// Сортировка: сначала по ошибкам, при равных ошибках — по стабильному хэшу
batchPool.sort((a, b) => {
    const fA = a.text.match(/\[fails:: (\d+)\]/) ? parseInt(a.text.match(/\[fails:: (\d+)\]/)[1]) : 0;
    const fB = b.text.match(/\[fails:: (\d+)\]/) ? parseInt(b.text.match(/\[fails:: (\d+)\]/)[1]) : 0;
    
    if (fA !== fB) {
        return fA - fB;
    }
    return getHash(a.text) - getHash(b.text);
});

const batch = batchPool.slice(0, remain);

// Мобильно-ориентированный контейнер списков
const container = document.createElement("div");
container.style.cssText = "display: flex; flex-direction: column; gap: 12px; margin-top: 10px;";

batch.forEach(t => {
    const row = document.createElement("div");
    row.style.cssText = "display: flex; flex-direction: column; padding: 12px; background: var(--background-primary-alt); border-radius: 8px; border: 1px solid var(--background-modifier-border);";
    
    let wordHtml = t.text.replace(/\[fails:: \d+\]/g, "").replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "").trim();
    wordHtml = wordHtml.replace(/\*\*(.*?)\*\*/g, "<b>$1</b>");

    const textDiv = document.createElement("div");
    textDiv.innerHTML = wordHtml;
    textDiv.style.cssText = "font-size: 15px; margin-bottom: 10px; line-height: 1.4; color: var(--text-normal);";
    
    const actionsDiv = document.createElement("div");
    actionsDiv.style.cssText = "display: flex; gap: 8px; width: 100%;";

    async function processTask(isSuccess) {
        let currentFailsMatch = t.text.match(/\[fails:: (\d+)\]/);
        let fails = currentFailsMatch ? parseInt(currentFailsMatch[1]) : 0;
        
        if (!isSuccess) fails++;

        let baseText = t.text.replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "");
        if (currentFailsMatch) {
            baseText = baseText.replace(/\[fails:: \d+\]/, `[fails:: ${fails}]`);
        } else {
            baseText += ` [fails:: ${fails}]`;
        }
        
        const newText = baseText + ` 🗓️${today}`;
        const oldLine = (t.completed ? "- [x] " : "- [ ] ") + t.text;
        const newLine = (isSuccess ? "- [x] " : "- [ ] ") + newText;
        
        await app.vault.process(tFile, data => data.replace(oldLine, newLine));
    }

    const btnWin = document.createElement("button");
    btnWin.innerText = "✅ Знаю";
    btnWin.style.cssText = "flex: 1; background: #10B981; color: white; border: none; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 14px;";
    btnWin.onclick = async () => await processTask(true);

    const btnFail = document.createElement("button");
    btnFail.innerText = "❌ Ошибся";
    btnFail.style.cssText = "flex: 1; background: #EF4444; color: white; border: none; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 14px;";
    btnFail.onclick = async () => await processTask(false);

    actionsDiv.appendChild(btnWin);
    actionsDiv.appendChild(btnFail);
    
    row.appendChild(textDiv);
    row.appendChild(actionsDiv);
    container.appendChild(row);
});

dv.container.appendChild(container);
```

```dataviewjs
const filePath = "03_Knowledge/ЕГЭ/Русский/01_Формы_Слов.md";
const tFile = app.vault.getAbstractFileByPath(filePath);
if (!tFile) return dv.paragraph("❌ Файл не найден. Проверь путь.");

const today = window.moment().format("YYYY-MM-DD");
const page = dv.page(filePath);
if (!page) return;

const allTasks = page.file.tasks;
const openTasks = allTasks.filter(t => !t.completed);

// === 🏆 СЦЕНАРИЙ АБСОЛЮТНОЙ ПОБЕДЫ ===
if (allTasks.length > 0 && openTasks.length === 0) {
    const winDiv = document.createElement("div");
    winDiv.innerHTML = `
        <div style="padding: 24px; background: linear-gradient(135deg, #2563EB 0%, #1D4ED8 100%); color: white; border-radius: 12px; text-align: center; box-shadow: 0 10px 25px -5px rgba(37, 99, 235, 0.4); margin-bottom: 20px;">
            <h2 style="margin: 0 0 10px 0; color: white; font-size: 22px;">🏆 ВСЯ МОРФОЛОГИЯ ОСВОЕНА!</h2>
            <p style="margin: 0 0 20px 0; font-size: 14px; opacity: 0.95;">Все ${allTasks.length} форм слов разложены по ячейкам памяти. Превосходный результат.</p>
            <button id="ng-plus-btn-7" style="background: white; color: #2563EB; border: none; padding: 12px 24px; font-size: 16px; font-weight: bold; border-radius: 8px; cursor: pointer; width: 100%; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);">
                🔄 ИДТИ НА 2-Й КРУГ (Сбросить прогресс)
            </button>
        </div>
    `;
    dv.container.appendChild(winDiv);

    document.getElementById("ng-plus-btn-7").onclick = async () => {
        await app.vault.process(tFile, data => data.replace(/- \[x\]/g, "- [ ]"));
    };
    return;
}

const limit = 20;
const processedToday = allTasks.filter(t => t.text.includes(`🗓️${today}`)).length;
const remain = limit - processedToday;

// === ⏳ СЦЕНАРИЙ СУТОЧНОЙ НОРМЫ ===
if (remain <= 0) {
    dv.el("div", `<div style="padding:16px; background:#2563EB; color:white; border-radius:8px; text-align:center; font-size: 15px; font-weight: 500; margin-bottom: 20px;">
    🧬 <b>НОРМА ФОРМ СЛОВ ВЫПОЛНЕНА!</b><br>
    <span style="font-size: 12px; opacity: 0.9;">10 модулей отстреляно. Барабан заблокирован до завтра.</span>
    </div>`);
    return;
}

dv.header(4, `🧬 Формы слов (Осталось на сегодня: ${remain})`);

let batchPool = Array.from(openTasks.filter(t => !t.text.includes(`🗓️${today}`)));

// === 🎲 СТАБИЛЬНЫЙ РАНДОМ (Хэш-функция) ===
// Генерируем уникальный, но постоянный на сегодня "вес" для каждого слова
function getHash(text) {
    const clean = text.replace(/\[fails:: \d+\]/g, "").replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "").trim();
    const str = clean + today; 
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
        hash = ((hash << 5) - hash) + str.charCodeAt(i);
        hash |= 0;
    }
    return hash;
}

// Сортировка: сначала по ошибкам (0 идут первыми), при равных ошибках — по стабильному хэшу
batchPool.sort((a, b) => {
    const fA = a.text.match(/\[fails:: (\d+)\]/) ? parseInt(a.text.match(/\[fails:: (\d+)\]/)[1]) : 0;
    const fB = b.text.match(/\[fails:: (\d+)\]/) ? parseInt(b.text.match(/\[fails:: (\d+)\]/)[1]) : 0;
    
    if (fA !== fB) {
        return fA - fB;
    }
    
    // Если ошибок поровну — сортируем по нашему замороженному рандому
    return getHash(a.text) - getHash(b.text);
});

const batch = batchPool.slice(0, remain);

// Мобильно-ориентированный контейнер списков
const container = document.createElement("div");
container.style.cssText = "display: flex; flex-direction: column; gap: 12px; margin-top: 10px;";

batch.forEach(t => {
    const row = document.createElement("div");
    row.style.cssText = "display: flex; flex-direction: column; padding: 12px; background: var(--background-primary-alt); border-radius: 8px; border: 1px solid var(--background-modifier-border);";
    
    let wordHtml = t.text.replace(/\[fails:: \d+\]/g, "").replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "").trim();
    wordHtml = wordHtml.replace(/\*\*(.*?)\*\*/g, "<b>$1</b>");

    const textDiv = document.createElement("div");
    textDiv.innerHTML = wordHtml;
    textDiv.style.cssText = "font-size: 15px; margin-bottom: 10px; line-height: 1.4; color: var(--text-normal);";
    
    const actionsDiv = document.createElement("div");
    actionsDiv.style.cssText = "display: flex; gap: 8px; width: 100%;";

    async function processTask(isSuccess) {
        let currentFailsMatch = t.text.match(/\[fails:: (\d+)\]/);
        let fails = currentFailsMatch ? parseInt(currentFailsMatch[1]) : 0;
        
        if (!isSuccess) fails++;

        let baseText = t.text.replace(/ 🗓️\d{4}-\d{2}-\d{2}/g, "");
        if (currentFailsMatch) {
            baseText = baseText.replace(/\[fails:: \d+\]/, `[fails:: ${fails}]`);
        } else {
            baseText += ` [fails:: ${fails}]`;
        }
        
        const newText = baseText + ` 🗓️${today}`;
        const oldLine = (t.completed ? "- [x] " : "- [ ] ") + t.text;
        const newLine = (isSuccess ? "- [x] " : "- [ ] ") + newText;
        
        await app.vault.process(tFile, data => data.replace(oldLine, newLine));
    }

    const btnWin = document.createElement("button");
    btnWin.innerText = "✅ Знаю";
    btnWin.style.cssText = "flex: 1; background: #10B981; color: white; border: none; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 14px;";
    btnWin.onclick = async () => await processTask(true);

    const btnFail = document.createElement("button");
    btnFail.innerText = "❌ Ошибся";
    btnFail.style.cssText = "flex: 1; background: #EF4444; color: white; border: none; padding: 10px; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 14px;";
    btnFail.onclick = async () => await processTask(false);

    actionsDiv.appendChild(btnWin);
    actionsDiv.appendChild(btnFail);
    
    row.appendChild(textDiv);
    row.appendChild(actionsDiv);
    container.appendChild(row);
});

dv.container.appendChild(container);
```
