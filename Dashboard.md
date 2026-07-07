
### 📅 Стена дисциплины 
```dataviewjs
const today = window.moment().format("YYYY-MM-DD");

// 1. Сканируем еженедельные заметки и считаем чекбоксы под заголовками дней
const weekPages = dv.pages('"02_Log/Week"');
const dateStats = {}; 

for (let p of weekPages) {
    for (let t of p.file.tasks) {
        const subpath = (t.section ? t.section.subpath : "") || (t.header || "");
        const match = subpath.match(/\d{4}-\d{2}-\d{2}/);
        if (match) {
            const d = match[0];
            if (!dateStats[d]) dateStats[d] = { total: 0, completed: 0 };
            dateStats[d].total++;
            if (t.completed) dateStats[d].completed++;
        }
    }
}

const isDateDone = (dStr) => {
    const s = dateStats[dStr];
    return s && s.total > 0 && s.completed === s.total;
};

// 2. Расчет непрерывного стрика
let currentStreak = 0;
let checkDate = window.moment();
if (!isDateDone(checkDate.format("YYYY-MM-DD"))) {
    checkDate.subtract(1, 'days');
}
while (isDateDone(checkDate.format("YYYY-MM-DD"))) {
    currentStreak++;
    checkDate.subtract(1, 'days');
}

// 3. Генерация сплошной стены за последние 180 дней (полгода)
const daysCount = 180;
const mainContainer = document.createElement("div");
mainContainer.style.cssText = "background: var(--background-primary-alt); padding: 18px; border-radius: 8px; border: 1px solid var(--background-modifier-border); margin-bottom: 25px;";

let headerHtml = `
    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px;">
        <span style="font-weight: bold; font-size: 15px;">АКТИВНЫЙ СТРИК: <span style="color: #10B981; font-size: 18px;">🔥 ${currentStreak} ДН.</span></span>
        <span style="font-size: 12px; color: var(--text-muted);">Сплошная сетка | 180 дней</span>
    </div>
`;

// Сплошной flex-контейнер
let gridHtml = `<div style="display: flex; flex-wrap: wrap; gap: 5px; justify-content: flex-start;">`;

for (let i = daysCount - 1; i >= 0; i--) {
    const dStr = window.moment().subtract(i, 'days').format("YYYY-MM-DD");
    const isDone = isDateDone(dStr);
    const isToday = dStr === today;
    const stats = dateStats[dStr];
    
    let tooltip = 'Нет задач';
    if (stats && stats.total > 0) tooltip = `Задач: ${stats.completed}/${stats.total}`;

    let bg = isDone ? '#10B981' : 'var(--background-secondary)';
    let border = isToday ? '2px solid #3B82F6' : '1px solid var(--background-modifier-border)';

    gridHtml += `
        <div title="${dStr} | ${tooltip}" style="
            width: 18px; height: 18px; border-radius: 4px;
            background: ${bg}; border: ${border};
            display: flex; align-items: center; justify-content: center;
            font-size: 10px; color: #fff; font-weight: bold;
        ">${isDone ? '✓' : ''}</div>
    `;
}

gridHtml += `</div>`;
mainContainer.innerHTML = headerHtml + gridHtml;
dv.container.appendChild(mainContainer);
```

***

### 📈 Линейный график результатов
```dataviewjs
const weekPages = dv.pages('"02_Log/Week"').where(p => p.rus_score || p.math_score || p.inf_score).sort(p => p.file.name, 'asc');

if (weekPages.length < 1) {
    dv.paragraph("🔻 *График пуст. Заполни свойства rus_score, math_score или inf_score вверху файлов 02_Log/Week*");
} else {
    const records = weekPages.values;
    const width = 620;
    const height = 240;
    const padding = 45;
    
    const stepX = records.length > 1 ? (width - padding * 2) / (records.length - 1) : 0;
    const getValuesY = (val) => height - padding - (val / 100) * (height - padding * 2);

    const extractNums = (v) => {
        if (v === null || v === undefined || v === "") return [];
        if (Array.isArray(v)) return v.map(n => Number(n)).filter(n => !isNaN(n));
        if (typeof v === 'string' && v.includes(',')) return v.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n));
        const n = Number(v);
        return isNaN(n) ? [] : [n];
    };

    const cleanVal = (v) => {
        const nums = extractNums(v);
        return nums.length ? Math.min(100, Math.max(0, Math.round(nums.reduce((a,b)=>a+b,0)/nums.length))) : null;
    };

    let xLabels = "";
    let rusPoints = [], mathPoints = [], infPoints = [];
    let allPointsByX = {};
    let allRusNums = [], allMathNums = [], allInfNums = [];

    records.forEach((p, idx) => {
        const x = padding + idx * stepX;
        const label = p.file.name.replace(".md", "").split("-W")[1] || p.file.name;
        
        allRusNums.push(...extractNums(p.rus_score));
        allMathNums.push(...extractNums(p.math_score));
        allInfNums.push(...extractNums(p.inf_score));

        const r = cleanVal(p.rus_score);
        const m = cleanVal(p.math_score);
        const inf = cleanVal(p.inf_score);

        allPointsByX[x] = [];
        if (r !== null) { rusPoints.push({x, y: getValuesY(r), val: r}); allPointsByX[x].push({y: getValuesY(r), val: r, color: "#EF4444"}); }
        if (m !== null) { mathPoints.push({x, y: getValuesY(m), val: m}); allPointsByX[x].push({y: getValuesY(m), val: m, color: "#3B82F6"}); }
        if (inf !== null) { infPoints.push({x, y: getValuesY(inf), val: inf}); allPointsByX[x].push({y: getValuesY(inf), val: inf, color: "#10B981"}); }
        
        xLabels += `<text x="${x}" y="${height - 12}" font-size="11" font-weight="bold" fill="var(--text-muted)" text-anchor="middle">W${label}</text>`;
    });

    const getAvgLast10 = (arr) => {
        if (arr.length === 0) return "-";
        const slice = arr.slice(-10);
        return Math.round(slice.reduce((a, b) => a + b, 0) / slice.length);
    };

    const rusAvg = getAvgLast10(allRusNums);
    const mathAvg = getAvgLast10(allMathNums);
    const infAvg = getAvgLast10(allInfNums);

    const makePath = (pts) => pts.length > 0 ? pts.map((p, i) => `${i===0?'M':'L'}${p.x},${p.y}`).join(" ") : "";
    
    let circlesAndLabels = "";
    Object.keys(allPointsByX).forEach(x => {
        let pts = allPointsByX[x];
        pts.sort((a, b) => a.y - b.y);
        
        pts.forEach((p, i) => {
            circlesAndLabels += `<circle cx="${x}" cy="${p.y}" r="4.5" fill="${p.color}" stroke="var(--background-primary-alt)" stroke-width="1.5"></circle>`;
            let labelY = p.y - 8;
            if (i > 0 && Math.abs(pts[i-1].y - p.y) < 14) {
                labelY = p.y + 16;
            }
            circlesAndLabels += `<text x="${x}" y="${labelY}" font-size="10" font-weight="bold" fill="var(--text-normal)" text-anchor="middle">${p.val}</text>`;
        });
    });

    const chartContainer = document.createElement("div");
    chartContainer.style.cssText = "background: var(--background-primary-alt); padding: 15px; border-radius: 8px; border: 1px solid var(--background-modifier-border);";

    let yGrid = "";
    [0, 25, 50, 75, 100].forEach(val => {
        const y = getValuesY(val);
        yGrid += `
            <line x1="${padding}" y1="${y}" x2="${width - padding}" y2="${y}" stroke="var(--background-modifier-border)" stroke-width="1" stroke-dasharray="4,4"></line>
            <text x="${padding - 10}" y="${y + 4}" font-size="10" fill="var(--text-muted)" text-anchor="end">${val}</text>
        `;
    });

    chartContainer.innerHTML = `
        <div style="display: flex; gap: 16px; margin-bottom: 15px; font-size: 12px; font-weight: bold; justify-content: center; flex-wrap: wrap;">
            <span style="color: #EF4444;">● Русский (${rusAvg})</span>
            <span style="color: #3B82F6;">● Математика (${mathAvg})</span>
            <span style="color: #10B981;">● Информатика (${infAvg})</span>
        </div>
        <svg viewBox="0 0 ${width} ${height}" style="width: 100%; height: auto; overflow: visible;">
            ${yGrid}
            ${xLabels}
            <path d="${makePath(rusPoints)}" fill="none" stroke="#EF4444" stroke-width="2.5"></path>
            <path d="${makePath(mathPoints)}" fill="none" stroke="#3B82F6" stroke-width="2.5"></path>
            <path d="${makePath(infPoints)}" fill="none" stroke="#10B981" stroke-width="2.5"></path>
            ${circlesAndLabels}
        </svg>
    `;
    dv.container.appendChild(chartContainer);
}
```

