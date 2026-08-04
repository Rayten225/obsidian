```dataviewjs
const btn = document.createElement("button");
btn.textContent = "Обновить свойства в старых днях и неделях";
btn.style.cssText = "background: #3B82F6; color: white; padding: 12px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; border: none;";

btn.onclick = async () => {
    btn.textContent = "Идет обновление, жди...";
    btn.style.background = "#F59E0B";

    // 1. Обновляем ДНИ
    const days = dv.pages('"02_Log/Days"');
    for (let p of days) {
        const file = app.vault.getAbstractFileByPath(p.file.path);
        if (file) {
            await app.fileManager.processFrontMatter(file, (fm) => {
                fm.type = "day";
                if (!fm.tags) fm.tags = [];
                if (!fm.tags.includes("day")) fm.tags.push("day");
                
                // Вычисляем, к какой неделе относится этот старый день
                const d = window.moment(p.file.name.replace(".md", ""), "YYYY-MM-DD");
                if (d.isValid()) {
                    const wn = d.isoWeek();
                    const yr = d.isoWeekYear();
                    const weekStr = `${yr}-W${wn < 10 ? '0'+wn : wn}`;
                    fm.week = `[[${weekStr}]]`;
                }
            });
        }
    }

    // 2. Обновляем НЕДЕЛИ
    const weeks = dv.pages('"02_Log/Week"');
    for (let p of weeks) {
        const file = app.vault.getAbstractFileByPath(p.file.path);
        if (file) {
            await app.fileManager.processFrontMatter(file, (fm) => {
                fm.type = "week";
                if (!fm.tags) fm.tags = [];
                if (!fm.tags.includes("week")) fm.tags.push("week");
            });
        }
    }
    
    btn.textContent = "✅ Готово! Все старые файлы обновлены";
    btn.style.background = "#10B981";
};

dv.container.appendChild(btn);
```