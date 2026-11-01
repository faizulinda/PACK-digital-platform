---
id: DP.M.342
name: Stable Login Identifier for Tracker Parsing
name_ru: Login исполнителя — стабильный идентификатор при парсинге трекера
name_en: Stable Login Identifier for Tracker Parsing
type: method
status: draft
summary: "При извлечении данных из трекеров (YouTrack, Jira, GitHub) исполнителя задачи идентифицировать по login/username, а не по display name. Display name меняется при переименовании; login остаётся стабильным. Паттерн: login как ID, display name из конфиг-маппинга."
created: 2026-10-29
trust:
  F: 3
  G: domain
  R: 0.80
epistemic_stage: emerging
related:
  uses: []
  references: []
  realized_by: []
tags: [tracker, login, identifier, parsing, analytics, stable-id]
wp: WP-40
---

# Login исполнителя — стабильный идентификатор при парсинге трекера

## Проблема

При построении аналитики по задачам трекера исполнитель идентифицируется по display name. Display name нестабилен: переименование, смена фамилии, опечатка — один человек выглядит как несколько в исторических агрегатах.

## Решение

Хранить исполнителя по login/username (стабильный ID), display name брать из конфиг-маппинга при выводе:

```
извлечь assignee.login → хранить login → при выводе: config["people"][login] → display_name
```

## Применимость

Скрипты аналитики, дашборды, прогнозные модели — любые инструменты, где нужна группировка по людям через данные трекеров (YouTrack, Jira, GitHub).

## Пример

`assignee_login` в `epic-forecast.py` (коммит 255e05b, DS-EvenBet): агрегирует данные по login, display name берёт из конфига `people`.
