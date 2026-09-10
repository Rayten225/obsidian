
```dataviewjs
const tasks = dv.pages().file.tasks.where(t => !t.completed);

const q1 = tasks.where(t => t.text.includes("#q1"));
const q2 = tasks.where(t => t.text.includes("#q2"));
const q3 = tasks.where(t => t.text.includes("#q3"));
const q4 = tasks.where(t => t.text.includes("#q4"));

dv.span(`<b style="color: #EF4444; font-size: 1.2em;">🔥 Q1: Срочно и Важно (#q1)</b><br><span style="font-size: 0.85em; color: var(--text-muted);">Сделать немедленно</span>`);
if (q1.length > 0) dv.taskList(q1, false);
else dv.paragraph("*Чисто*");

dv.span(`<br><b style="color: #10B981; font-size: 1.2em;">🎯 Q2: Важно, не срочно (#q2)</b><br><span style="font-size: 0.85em; color: var(--text-muted);">Запланировать в календарь</span>`);
if (q2.length > 0) dv.taskList(q2, false);
else dv.paragraph("*Чисто*");

dv.span(`<br><b style="color: #F59E0B; font-size: 1.2em;">⚡ Q3: Срочно, не важно (#q3)</b><br><span style="font-size: 0.85em; color: var(--text-muted);">Делегировать или автоматизировать</span>`);
if (q3.length > 0) dv.taskList(q3, false);
else dv.paragraph("*Чисто*");

dv.span(`<br><b style="color: var(--text-muted); font-size: 1.2em;">🗑️ Q4: Не срочно и не важно (#q4)</b><br><span style="font-size: 0.85em; color: var(--text-muted);">Игнорировать</span>`);
if (q4.length > 0) dv.taskList(q4, false);
else dv.paragraph("*Чисто*");
```

---
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
```dataviewjs
// Настройки карантина
const CAPACITY = 5; 
const REVIEW_DAY = 7; // Воскресенье
const REVIEW_HOUR = 18; // 18:00

// Вычисление дедлайна относительно текущего времени
let nextReview = window.moment().isoWeekday(REVIEW_DAY).hour(REVIEW_HOUR).minute(0).second(0);
if (window.moment().isAfter(nextReview)) {
    nextReview.add(1, 'weeks');
}

const inboxTasks = dv.pages().file.tasks.where(t => !t.completed && t.text.includes("#inbox"));
const count = inboxTasks.length;

// Цветовая индикация перегруза
let color = "#10B981"; // Зеленый (Норма)
if (count >= CAPACITY) color = "#EF4444"; // Красный (Перегруз)
else if (count >= CAPACITY * 0.7) color = "#F59E0B"; // Желтый (Внимание)

dv.span(`
<div style="background: var(--background-primary-alt); padding: 15px; border-radius: 8px; border: 1px solid var(--background-modifier-border); margin-bottom: 20px;">
    <div style="display: flex; justify-content: space-between; align-items: center;">
        <b style="color: ${color}; font-size: 1.1em;">📦 Буфер задач: ${count} / ${CAPACITY}</b>
        <span style="font-size: 0.9em; color: var(--text-muted);">Разбор: <b>${nextReview.format("DD.MM в HH:mm")}</b> (${nextReview.fromNow()})</span>
    </div>
</div>
`);

if (count > 0) {
    dv.taskList(inboxTasks, false);
} else {
    dv.paragraph("🔻 *Буфер пуст. Оперативная память свободна.*");
}
```
