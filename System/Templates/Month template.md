<%*
function pad(n){ return n < 10 ? '0' + n : '' + n; }
function fmt(d){ return d.getFullYear() + '-' + pad(d.getMonth()+1) + '-' + pad(d.getDate()); }

let answer = await tp.system.prompt("Какой месяц генерировать? (0=текущий, 1=следующий)", "1");
if (answer === null) answer = "1";
answer = answer.toString().trim().toLowerCase();
let offset = 1;
if (["текущий", "тек", "current", "0"].includes(answer)) offset = 0;
else if (["следующий", "след", "next", "1"].includes(answer)) offset = 1;
else { const parsed = parseInt(answer, 10); if (!isNaN(parsed)) offset = parsed; else offset = 1; }

const today = new Date();
let curYear = today.getFullYear();
let curMonth = today.getMonth();
const targetMonthIndex = (curMonth + offset) % 12;
const targetYear = curYear + Math.floor((curMonth + offset) / 12);
const firstDay = new Date(targetYear, targetMonthIndex, 1);
const lastDay = new Date(targetYear, targetMonthIndex + 1, 0);

const monthNamesNom = ['Январь','Февраль','Март','Апрель','Май','Июнь','Июль','Август','Сентябрь','Октябрь','Ноябрь','Декабрь'];
const displayMonthName = monthNamesNom[firstDay.getMonth()];
const newFileName = `${targetYear}-${pad(firstDay.getMonth()+1)} ${displayMonthName}`;

try { await tp.file.rename(newFileName); } catch(e){}

function getWeeksByMonthStart(first, last) {
    const weeks = [];
    const fdow = first.getDay() || 7;
    const daysToSunday = 7 - fdow;
    let firstWeekEnd = new Date(first);
    firstWeekEnd.setDate(first.getDate() + daysToSunday);
    if (firstWeekEnd > last) firstWeekEnd = new Date(last);
    weeks.push({ start: new Date(first), end: new Date(firstWeekEnd) });
    
    let curStart = new Date(firstWeekEnd);
    curStart.setDate(curStart.getDate() + 1);
    while (curStart <= last) {
        let curEnd = new Date(curStart);
        curEnd.setDate(curStart.getDate() + 6);
        if (curEnd > last) curEnd = new Date(last);
        weeks.push({ start: new Date(curStart), end: new Date(curEnd) });
        curStart = new Date(curEnd);
        curStart.setDate(curStart.getDate() + 1);
    }
    return weeks;
}
const weeks = getWeeksByMonthStart(firstDay, lastDay);

tR += `---
type: month
month_start: ${fmt(firstDay)}
month_end: ${fmt(lastDay)}
---

# 🎯 План на месяц — ${displayMonthName} ${firstDay.getFullYear()}

### 🏆 Главные цели месяца
- [ ] Цель 1
- [ ] Цель 2
- [ ] Цель 3

---

### 🗓 Детальный план по неделям
`;

for (let i = 0; i < weeks.length; i++) {
    const w = weeks[i];
    tR += `**Неделя ${i+1}** (${fmt(w.start)} — ${fmt(w.end)})\n- \n\n`;
}

tR += `---
### 🧠 Итоги и рефлексия (в конце месяца)
**Что удалось:**
* **Что не удалось:**
* **Что изменить в следующем месяце:**
* `;
-%>