# 📊 Пульс

### ⏱️ Работа и Учеба (Динамика)

```dataviewjs
const pages = dv.pages().where(p => p.file.path.startsWith("Log/Days/")).sort(p => p.file.name).slice(-7);
const dates = pages.map(p => p.file.name).values;
const work = pages.map(p => p.work_time || 0).values;
const ege = pages.map(p => p.ege_time || 0).values;

const chartData = {
    type: 'line',
    data: {
        labels: dates,
        datasets: [
            { label: '💻 Программирование (ч)', data: work, borderColor: '#ffffff', borderWidth: 2, tension: 0.2 },
            { label: '📚 ЕГЭ (ч)', data: ege, borderColor: '#888888', borderWidth: 2, tension: 0.2 }
        ]
    }
};

window.renderChart(chartData, this.container);
```

***

### 💪 Прогресс в зале (Тоннаж)

```dataviewjs
const pages = dv.pages().where(p => p.file.path.startsWith("Log/Days/") && p.gym_weight > 0).sort(p => p.file.name).slice(-10);
const dates = pages.map(p => p.file.name).values;
const weight = pages.map(p => p.gym_weight).values;

const chartData = {
    type: 'line',
    data: {
        labels: dates,
        datasets: [{ label: '🏋️‍♂️ Тоннаж (кг)', data: weight, borderColor: '#ffffff', borderWidth: 2, tension: 0.2 }]
    }
};

window.renderChart(chartData, this.container);
```

