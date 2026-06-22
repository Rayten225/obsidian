<%*
const a = await tp.system.prompt("0 — эта неделя, 1 — следующая", "0");
const offset = parseInt(a) || 0;

// Используем window.moment для 100% совместимости
const startOfWeek = window.moment().add(offset, 'weeks').startOf('isoWeek');

const days = [];
const dayNames = ['Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб', 'Вс'];

for (let i = 0; i < 7; i++) {
    days.push({
        name: dayNames[i],
        date: startOfWeek.clone().add(i, 'days').format("YYYY-MM-DD")
    });
}

const wn = startOfWeek.isoWeek();
const year = startOfWeek.isoWeekYear();
const fileName = `${year}-W${wn < 10 ? '0' + wn : wn}`;

try { 
    // Проверяем, есть ли уже такой файл, чтобы не выбивать ошибку переименования
    if (tp.file.title !== fileName) {
        await tp.file.rename(fileName); 
    }
} catch (e) {
    new Notice("Файл недели уже существует или переименован.");
}
-%>
---
type: week
---
# 📅 Неделя <% wn %> (<% days[0].date %> — <% days[6].date %>)

### 📊 Авто-сводка за неделю
<%* tR += "```dataviewjs\n"; %>
const weekDates = ["<% days[0].date %>", "<% days[1].date %>", "<% days[2].date %>", "<% days[3].date %>", "<% days[4].date %>", "<% days[5].date %>", "<% days[6].date %>"];

const pages = dv.pages().where(p => weekDates.includes(p.file.name));

dv.table(["День", "💻 Код (ч)", "📚 ЕГЭ (ч)", "💪 Тоннаж (кг)", "🇬🇧 Англ"], 
    pages.sort(p => p.file.name).map(p => [p.file.link, p.work_time || 0, p.ege_time || 0, p.gym_weight || 0, p.english_done ? "✅" : "❌"])
);
let tWork = 0, tEge = 0, tGym = 0, tEng = 0;
pages.forEach(p => { tWork += p.work_time || 0; tEge += p.ege_time || 0; tGym += p.gym_weight || 0; if(p.english_done) tEng++; });
dv.paragraph("> **🔥 ИТОГИ НЕДЕЛИ:** Код: " + tWork + " ч | ЕГЭ: " + tEge + " ч | Тоннаж: " + tGym + " кг | Англ: " + tEng + " дн");
<%* tR += "```\n"; %>

### 📝 Планы по дням

<%* for (let d of days) { %>
#### <% d.name %> — <% d.date %>
- [ ] 

<%* } %>