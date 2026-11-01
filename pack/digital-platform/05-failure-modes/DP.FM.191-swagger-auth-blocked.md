---
id: DP.FM.191
name: "Swagger/OpenAPI UI закрыт авторизацией на тестовом стенде — агент читает 401 вместо схемы"
status: draft
type: failure-mode
pack: PACK-digital-platform
domain: digital-platform / api-integration
epistemic_stage: confirmed
valid_from: 2026-10-30
source: "inbox-check 2026-10-30 (DS-strategy, git commit b5ad184)"
related:
  see_also: [DP.SOTA.003]
---

# DP.FM.191 — Swagger/OpenAPI UI закрыт авторизацией на тестовом стенде — агент читает 401 вместо схемы

## Описание

Интерактивная документация API (Swagger UI, Redoc) на тестовых стендах часто закрыта теми же middleware-правилами, что и сам API: basic auth, OAuth, IP-whitelist. Агент или скрипт получает 401 или HTML-страницу входа вместо JSON-схемы и не может автоматически прочитать спецификацию.

## Пример

EvenBet Original Games API: запрос к Swagger UI на тестовом стенде возвращал 401. Агент не мог получить схему автоматически. Паттерн обхода: пилот вручную экспортировал `openapi.json` и положил его в `released-api.md` в репозитории рядом с кодом потребителя (DS-EvenBet, git commit a55896a).

## Тест обнаружения

«Возвращает ли GET /swagger.json или /openapi.json запрос 401 / HTML-страницу входа?» Да → агент не получит схему без ручного экспорта.

## Инвариант

Swagger UI — потребительский интерфейс, не public endpoint. Его закрытие авторизацией — штатное решение безопасности. Автоматизация, рассчитывающая на открытый доступ к схеме, ломается тихо (агент не сигнализирует, что получил не JSON).

## Митигация

1. Пилот вручную экспортирует `openapi.json` / `swagger.yaml` и кладёт в репозиторий рядом с кодом потребителя (`released-api.md`, `api-spec.yaml`).
2. При обновлении API — повторить экспорт.
3. Агент потребляет статический файл, не пытается читать live UI.

## Связи

- DP.SOTA.003 (Open API Specifications) — контекст: OpenAPI как стандарт спецификации; данный FM — о доступности runtime-экземпляра на стенде
