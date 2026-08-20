<%*
let title = tp.file.title;
if (title.includes("Untitled") || title.includes("Без названия") || title.includes("Новая заметка")) {
    title = window.moment().format("YYYY-MM-DD");
    try { await tp.file.rename(title); } catch(e) {}
}

const noteDate = window.moment(title, "YYYY-MM-DD");
const wn = noteDate.isoWeek();
const year = noteDate.isoWeekYear();
const weekFileName = `${year}-W${wn < 10 ? '0' + wn : wn}`;

const dayNames = ['Вс', 'Пн', 'Вт', 'Ср', 'Чт', 'Пт', 'Сб'];
const dayName = dayNames[noteDate.day()]; 
const headingName = `${dayName} — ${title}`;
-%>
---
type: day
tags:
  - day
week: "[[<% weekFileName %>]]"
work_time: 0
ege_time: 0
gym_weight: 0
english_done: false
---
# 🗓 <% dayName %>, <% title %>

### 🎯 План на день
- [ ] Встать в 9:00
- [ ] Почистить зубы
- [ ] Позавтракать
- [ ] Сделать русский на главной странице 
- [ ] Выпить креатин и протеин
- [ ] пообедать
- [ ] Выпить протеин после тренировки (если есть)
- [ ] Почистить зубы перед сном 
- [ ] Лечь спать до 00:00
### Учебный план:
![[<% weekFileName %>#<% headingName %>]]

---
### 🧠 Заметки / Фокус дня
- 

### 🌙 Рефлексия (Итоги)
**Что прошло отлично:**
- 

**Что нужно улучшить (ошибки дня):**
-