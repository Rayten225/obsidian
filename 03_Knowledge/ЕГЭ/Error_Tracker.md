# 🛠️ Ремонтный цех ЕГЭ (Баг-репорты)

```dataviewjs
const bugs = dv.current().file.tasks.where(t => t.text.includes("BUG::"));
const open = bugs.where(t => !t.completed).length;
const fixed = bugs.where(t => t.completed).length;

dv.el("div", 
    `<div style="padding: 14px; background: #1E293B; border-left: 4px solid #EF4444; border-radius: 6px; margin-bottom: 15px;">` +
    `<span style="font-family: monospace; font-size: 14px; color: #F8FAFC;">` +
    `<b>ТЕХНИЧЕСКИЙ ДОЛГ:</b> Активных багов в очереди: <b style="color:#EF4444;">${open}</b> | ` +
    `Устранено навсегда: <b style="color:#10B981;">${fixed}</b>` +
    `</span></div>`
);
```

### 🚨 Открытые баги (Требуют повторного решения через 3 дня)
```dataview
TASK
FROM "03_Knowledge/ЕГЭ/Error_Tracker"
WHERE !completed AND contains(text, "BUG::")
SORT file.mtime ASC
```

***

### 📝 Склад баг-репортов (Формат записи нового косяка)
> [!warning]- Как правильно заносить ошибку (Разверни инструкцию)
> **Синтаксис:** `- [ ] [BUG:: Предмет | № задания] Краткая суть косяка -> Правильный алгоритм 🗓️YYYY-MM-DD`
> *Дата в конце (`🗓️`) — это день, когда ты ОБЯЗАН заново решить аналогичную задачу, чтобы закрыть баг.*

- [ ] [BUG:: Информатика | №24] Ошибка в регулярном выражении `([6789][67890]+)([-+](0|[6789][67890]+))+` вместо + на до * (`[67890]+`)  -> так как может быть 123+2+34 если сделать + то будут только от двухзначных чисел а если * то цифры тоже будут работать   🗓️2026-07-06

- [ ] [BUG:: Математика | №6] Повторить все свойства логарифмов   🗓️2026-07-07