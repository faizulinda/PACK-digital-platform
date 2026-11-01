---
id: DP.FM.190
name: YouTrack Feature Creation Traps
name_ru: Ловушки создания Feature-задачи в YouTrack через API
name_en: YouTrack Feature Creation Traps via API
type: fm
status: draft
summary: "Два неочевидных ограничения при создании Feature в YouTrack через API: (1) поле Marketing обязательно — без него API молча отказывает; (2) поле Stage игнорирует переданное значение — всегда устанавливается L2."
created: 2026-10-29
trust:
  F: 3
  G: domain
  R: 0.85
epistemic_stage: confirmed
related:
  uses: []
  references: []
  realized_by: []
tags: [youtrack, api, tracker, feature-creation, required-field, stage]
wp: WP-40
---

# Ловушки создания Feature-задачи в YouTrack через API

## Симптом 1: Feature не создаётся — нет ошибки, нет задачи

API-вызов завершается без явной HTTP-ошибки, но Feature-задача не появляется в трекере.

**Причина:** поле `Marketing` обязательно для типа Feature. Без явного значения YouTrack API молча отказывает в создании — без ошибки и без информативного сообщения.

**Обход:**

```json
{
  "Marketing": "Not required",
  "QA": "Not required",
  "Documentation": "Not required"
}
```

## Симптом 2: Stage ≠ переданному значению

Задача создана, но `Stage = L2`, хотя передавалось `"To Do"` или иное значение.

**Причина:** YouTrack принудительно устанавливает `Stage = L2` при создании Feature — независимо от переданного значения. Передавать Stage при создании бессмысленно.

**Обход:** устанавливать Stage отдельным вызовом после создания:

```
createIssue({..., "Marketing": "Not required"})
  → updateIssue(newId, {"Stage": "To Do"})
```

## Подтверждение

Подтверждено 26.10.2026 на задачах Original Games и дизайна (write-task skill, evenbet-agent-bridge коммит 8574185).
