# 🇬🇧 English B1 // Главный Терминал

```dataviewjs
const bugs = dv.pages('"03_Knowledge/Englich/Error_Tracker"');
const open = bugs.where(b => b.status === "open").length;
const fixed = bugs.where(b => b.status === "fixed").length;
const rules = dv.pages('"03_Knowledge/Englich/Grammar_Engine"').length;
const chunks = dv.pages('"03_Knowledge/Englich/Chunks_Vocabulary"');
const ammo = chunks.where(c => c.status === "active").length;

dv.span(`<span style="font-family: monospace; font-size: 14px;">` +
`<b>ТЕЛЕМЕТРИЯ:</b> Движков: <b>${rules}</b> | ` +
`Активных багов: <b style="color:#EF4444;">${open}</b> | ` +
`Устранено: <b style="color:#10B981;">${fixed}</b> | ` +
`Патронов в обойме: <b style="color:#F59E0B;">${ammo}</b>` +
`</span>`);
```

***

### 🔫 Тактическая обойма (Чанки в активной прокачке)
```dataview
TABLE Триггер as "Мысль в голове", fired as "Выстрелов"
FROM "03_Knowledge/English/Chunks_Vocabulary"
WHERE status = "active"
SORT fired ASC
LIMIT 6
```

### 🛠️ Ремонтный цех (Открытые баги)
```dataview
TABLE Ляпнул, Надо, file.outlinks[0] as "Правило"
FROM "03_Knowledge/English/Error_Tracker"
WHERE status = "open"
```

### 📉 Топ нестабильных модулей (Где ты ошибаешься чаще всего)
```dataview
TABLE length(file.inlinks) as "Связанных багов"
FROM "03_Knowledge/English/Grammar_Engine"
SORT length(file.inlinks) DESC
LIMIT 5
```

***

### 🚀 ПЛАН ТЕКУЩЕГО СПРИНТА (Одиночный режим с ИИ)

#### Неделя 1: Базовые движки и «Снятие ржавчины»
- [ ] **День 1:** Past Simple vs Present Perfect `[[Past Simple vs Present Perfect]]`
- [ ] **День 2-3:** Past Continuous vs Past Simple
- [ ] **День 4-5:** Present Perfect Continuous `[[Present Perfect Continuous]]`
*Практика:* Голосовой диалог с ИИ (ChatGPT/Gemini) в роли Senior-раз работчика.

#### Неделя 2: Прыжки во времени и Пассив
- [ ] **День 1-2:** Past Perfect (Had done)
- [ ] **День 3-4:** Future in the Past (would)
- [ ] **День 5:** Passive Voice во всех временах
*Практика:* Проговаривание своих действий за компом вслух через связки *because* и *so*.

#### Неделя 3: Альтернативная реальность
- [ ] **День 1-2:** Conditionals 0 и 1 (If you do, it works)
- [ ] **День 3-4:** Conditionals 2 (If I had, I would)
- [ ] **День 5:** Косвенные вопросы (Could you tell me...)
*Практика:* Задать ИИ 5 вопросов через формулу *"Could you tell me..."*

#### Неделя 4: Интеграция и QA-тестирование
- [ ] Повторение топ-5 правил из таблицы «Топ нестабильных модулей» выше.
- [ ] Запись 2-минутного монолога на диктофон -> поиск багов -> занесение в `Error_Tracker`.