---
id: DP.M.345
name: "Decisions Drift Check"
status: draft
type: method
pack: PACK-digital-platform
domain: digital-platform
epistemic_stage: hypothesis
trust: medium
valid_from: 2026-10-31
source: "git commit 46d02ef DS-EvenBet, inbox-capture 2026-10-31"
---

# DP.M.345 — Decisions Drift Check

## Описание

Паттерн предотвращения тихого дрейфа решений: решение записывается один раз в реестр с регулярными выражениями «что должно быть в задаче», скрипт периодически проверяет соответствие задач трекера принятым решениям.

## Структура

**Реестр в `decisions.md`** — таблица «Проверки для сверки»:

| Решение | Задача | expected_regex | stale_regex (опц.) |
|---------|--------|---------------|-------------------|
| Описание решения | TICKET-123 | `ожидаемое значение` | `устаревшее значение` |

- `expected_regex` — Python-regex, который **должен** встречаться в summary + description задачи.
- `stale_regex` — regex, который **не должен** встречаться (маркер устаревшего состояния).

**Скрипт `decisions-drift.py`:**
1. Читает таблицу «Проверки для сверки» из `decisions.md`.
2. Для каждой строки: GET к API трекера (read-only).
3. Проверяет: `expected_regex` присутствует AND `stale_regex` отсутствует (если задан).
4. Записывает отчёт `drift-YYYY-MM-DD.md`.
5. Exit 1 при расхождении, exit 2 при недоступности трекера.

## Когда запускать

- Пятничная сверка (Weekly Review).
- После крупного изменения формулировок задач.
- Опционально: в CI как pre-merge гейт.

## Ключевые свойства

- Скрипт **read-only**: не изменяет трекер.
- Одна строка = одна проверяемая договорённость.
- `stale_regex` явно обнаруживает устаревшее состояние (не только «нового нет», но и «старое ещё есть»).
- Exit-коды разделяют «нарушение» (1) и «недоступность» (2).

## Отличие от DP.M.269 (Bidirectional registry drift guard)

| Аспект | DP.M.269 | DP.M.345 |
|--------|----------|----------|
| Объект | Файлы репозитория | Задачи трекера |
| Механизм | commit-msg hook | Scheduled скрипт |
| Направление | Bidirectional | Unidirectional (decisions → tracker) |
| Проверка | Факт изменения файла | Regex в тексте задачи |

## Применимость

Любой проект с реестром решений и задачами в трекере (YouTrack, Jira, GitHub Issues, Linear). Особенно важен в длинных эпиках (> 2 месяцев), где формулировки задач правятся по ходу работы.

## Связи

- DP.M.339 (FM Epic Agent Pipeline) — этот метод вызывается на этапе 5 конвейера
- DP.M.269 (Bidirectional registry drift guard) — смежный, file-level
- DS-EvenBet/tools/decisions-drift.py — реализация, коммит 46d02ef
- DS-EvenBet/process/methods/fm-epic-pipeline.md — контекст применения
- WP-40 (Original Games), WP-30 (Game Server)
