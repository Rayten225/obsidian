# 📋 Пул задач

### 📌 Глобальный Бэклог (Без дедлайна)
- [ ] Почистить картошки

---
### ⏳ Забытое в ежедневниках (Долги)
```dataview
TASK
WHERE contains(file.name, "-W")
WHERE !completed
WHERE section != null
WHERE date(substring(meta(section).subpath, length(meta(section).subpath) - 10)) <= date(today)
GROUP BY file.link
```
