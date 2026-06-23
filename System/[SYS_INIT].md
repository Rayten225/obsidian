```dataviewjs
// 1. Укажи тут точную дату своего экзамена (Год-Месяц-День)
const target = moment("2027-05-26"); 
const daysLeft = target.diff(moment(), 'days');

// 2. Выгребаем все заметки, в которых заполнено хотя бы одно из наших свойств
const pages = dv.pages().where(p => p.ege_time || p.work_time || p.gym_weight);

let tEge = 0, tWork = 0, tGym = 0;

pages.forEach(p => {
    tEge += Number(p.ege_time) || 0;
    tWork += Number(p.work_time) || 0;
    tGym += Number(p.gym_weight) || 0;
});

// Функция очистки кривых дробей JS (чтобы вместо 14.30000001ч выводилось 14.3ч)
const clean = (num) => Number(num.toFixed(1));

dv.span(`
<div style="font-family: 'Courier New', monospace; font-size: 13px; line-height: 1.6; border-left: 3px solid #10B981; padding-left: 10px; background: rgba(16, 185, 129, 0.05); border-radius: 0 4px 4px 0; margin: 4px 0;">
    <b style="color:#10B981; font-size: 1.1em;">SYS.TELEMETRY // ОБЩИЙ НАЛЁТ</b><br>
    ├─ 📚 ЕГЭ:  <b>${clean(tEge)} ч.</b><br>
    ├─ 💻 Код / СЕО:  <b>${clean(tWork)} ч.</b><br>
    ├─ 💪 Тоннаж (зал): <b>${tGym.toLocaleString('ru-RU')} кг</b><br>
    └─ ⏳ Прыжок (ЕГЭ): <b style="color:#EF4444;">${daysLeft} дн.</b>
</div>
`);
```
