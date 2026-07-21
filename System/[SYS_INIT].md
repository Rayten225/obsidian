```dataviewjs
// 1. Укажи тут точную дату своего экзамена (Год-Месяц-День)
const target = moment("2027-06-01"); 
const daysLeft = target.diff(moment(), 'days');

// 2. Выгребаем все заметки для подсчета часов и тоннажа
const pages = dv.pages().where(p => p.ege_time || p.work_time || p.gym_weight);

let tEge = 0, tWork = 0, tGym = 0;

pages.forEach(p => {
    tEge += Number(p.ege_time) || 0;
    tWork += Number(p.work_time) || 0;
    tGym += Number(p.gym_weight) || 0;
});

// 3. Выгребаем баллы ЕГЭ (из папки 02_Log/Week), сортируем по хронологии
const weekPages = dv.pages('"02_Log/Week"').where(p => p.rus_score || p.math_score || p.inf_score).sort(p => p.file.name, 'asc');

const extractNums = (v) => {
    if (v === null || v === undefined || v === "") return [];
    if (Array.isArray(v)) return v.map(n => Number(n)).filter(n => !isNaN(n));
    if (typeof v === 'string' && v.includes(',')) return v.split(',').map(s => parseFloat(s.trim())).filter(n => !isNaN(n));
    const n = Number(v);
    return isNaN(n) ? [] : [n];
};

let allRus = [], allMath = [], allInf = [];
weekPages.forEach(p => {
    allRus.push(...extractNums(p.rus_score));
    allMath.push(...extractNums(p.math_score));
    allInf.push(...extractNums(p.inf_score));
});

// Функция расчета среднего балла строго по ПОСЛЕДНИМ 10 пробникам
const getAvgLast10 = (arr) => {
    if (arr.length === 0) return 0;
    const slice = arr.slice(-10); // Берем только последние 10 элементов
    return Math.round(slice.reduce((a, b) => a + b, 0) / slice.length);
};

const totalAvgScore = getAvgLast10(allRus) + getAvgLast10(allMath) + getAvgLast10(allInf);

// Функция очистки кривых дробей JS (чтобы вместо 14.30000001ч выводилось 14.3ч)
const clean = (num) => Number(num.toFixed(1));

dv.span(`
<div style="font-family: 'Courier New', monospace; font-size: 13px; line-height: 1.6; border-left: 3px solid #10B981; padding-left: 10px; background: rgba(16, 185, 129, 0.05); border-radius: 0 4px 4px 0; margin: 4px 0;">
    <b style="color:#10B981; font-size: 1.1em;">SYS.TELEMETRY // ОБЩИЙ НАЛЁТ</b><br>
    ├─ 📚 ЕГЭ:  <b>${clean(tEge)} ч.</b><br>
    ├─ 🎯 Средний балл: <b>${totalAvgScore}/240</b> <span style="font-size:10px; color:var(--text-muted);"></span><br>
    ├─ 💻 Код / СЕО:  <b>${clean(tWork)} ч.</b><br>
    ├─ 💪 Тоннаж (зал): <b>${tGym.toLocaleString('ru-RU')} кг</b><br>
    └─ ⏳ Прыжок (ЕГЭ): <b style="color:#EF4444;">${daysLeft+1} дн.</b>
</div>
`);
```
