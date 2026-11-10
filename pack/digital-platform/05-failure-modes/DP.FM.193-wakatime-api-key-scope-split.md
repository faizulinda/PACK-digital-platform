---
id: DP.FM.193
name: WakaTime plugin API-ключ отклоняется на summaries endpoint — требуется OAuth Bearer
type: failure-mode
domain: digital-platform
pack_refs: []
status: active
valid_from: 2026-11-04
schema_version: 1
source: "inbox capture 2026-11-04 (git commit 09632ef FMT-exocortex-template)"
---

# DP.FM.193 — WakaTime plugin API-ключ отклоняется на summaries endpoint

## Описание

WakaTime разделил scope ключей доступа по endpoint'ам: plugin API key принимается только на `statusbar` endpoint (текущий день), а `summaries` endpoint (исторические данные) требует OAuth personal token. Скрипт, использующий plugin-ключ для обоих endpoint'ов, получает HTTP 403 на `summaries` и молча возвращает пустое значение — исторические данные теряются без явной ошибки.

## Симптом

Скрипт завершается успешно, но поля, заполняемые из исторических агрегатов (Day Close, мультипликатор за прошлые дни), пусты или нулевые. Причина не видна без явной обработки 403.

## Корень

Провайдер ввёл разделение: plugin API key — только для текущего дня (`statusbar`). Исторические агрегаты (`summaries`) требуют OAuth Bearer-токена. Скрипт не различал endpoint'ы и не обрабатывал 403 явно.

## Тест

«Возвращает ли скрипт читаемое сообщение при 403 от `summaries`, а не пустое значение?» и «Есть ли разная логика запроса для today vs. past days?»
- Нет → silent data loss при смене политики провайдера.

## Правило

При интеграции с внешними API с разделённым scope ключей:

1. **Маршрутизировать по endpoint-типу:** сегодня → `statusbar` (plugin-ключ); прошлые дни → `summaries` (OAuth Bearer из env `WAKATIME_OAUTH_TOKEN`).
2. **Обрабатывать 403 явно:** возвращать читаемую причину вместо пустого значения / голого HTTP-кода.
3. **Проверять наличие токена при старте:** если `WAKATIME_OAUTH_TOKEN` не задан — логировать предупреждение, а не падать молча.

## Пример

```python
# BEFORE: plugin key on summaries -> 403 -> empty result without an error
response = requests.get(SUMMARIES_URL, headers={"Authorization": f"Basic {PLUGIN_KEY}"})

# AFTER
if date == today:
    return fetch_statusbar(plugin_key)
else:
    if not oauth_token:
        return {"error": "WAKATIME_OAUTH_TOKEN not set — summaries endpoint requires OAuth"}
    return fetch_summaries(oauth_token)
```

## Связано

- DP.FM.036 — WakaTime Measurement Scope Bias (другая проблема: все репо без доменного фильтра)
- DP.FM.153 — Перемежающийся 401 со статическим ключом (другая причина: прокси/env, не key scope)
- WP-247 (FMT-exocortex-template, Day Close flow, issue #1414, issue #1398)
