<%*
let answer = await tp.system.prompt("Какой год создать? (0-текущий / 1-следующий)", "1");
if (answer === null) answer = "1";
answer = answer.toString().trim().toLowerCase();
let offset = 1;
if (["текущий","тек","current", "0"].includes(answer)) offset = 0;
else if (["следующий","след","next", "1"].includes(answer)) offset = 1;
else {
    const parsed = parseInt(answer, 10);
    if (!isNaN(parsed)) offset = parsed;
}

const today = new Date();
const targetYear = today.getFullYear() + offset;

try { await tp.file.rename(`Год ${targetYear}`); } catch (e) {}

const months = ['Январь','Февраль','Март','Апрель','Май','Июнь','Июль','Август','Сентябрь','Октябрь','Ноябрь','Декабрь'];
const quarters = [
    ['Q1', 'Январь–Март'],
    ['Q2', 'Апрель–Июнь'],
    ['Q3', 'Июль–Сентябрь'],
    ['Q4', 'Октябрь–Декабрь']
];

tR += `---
type: year
year: ${targetYear}
---

# 🚀 План на ${targetYear} год

### 🎯 Главные цели года
- [ ] Цель 1
- [ ] Цель 2
- [ ] Цель 3

---

### 🧭 План по кварталам
`;
for (const q of quarters) {
    tR += `**${q[0]} (${q[1]})**\n- Приоритет 1\n- Приоритет 2\n\n`;
}

tR += `---\n### 🗓 Детальный план по месяцам\n`;
for (let i = 0; i < months.length; i++) {
    tR += `**${months[i]}**\n- \n\n`;
}

tR += `---
### 📝 Итоги года
**Что удалось:**
* **Главные выводы:**
* `;
-%>