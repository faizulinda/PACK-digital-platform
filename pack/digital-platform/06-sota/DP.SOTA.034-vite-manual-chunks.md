---
id: DP.SOTA.034
name: Vite manual chunks — изоляция helper динамических импортов и vendor-ui
type: sota
status: active
summary: "В Vite + React SPA с ленивой загрузкой helper динамических импортов и UI-kit по умолчанию «поднимаются» в крупный чанк тяжёлого модуля, и лендинг тянет весь прикладной код. Явные manual chunks (vendor-react с helper-ом и отдельный vendor-ui) вернули лендингу лёгкий старт: 788 → ~360 КиБ, Lighthouse mobile 71 → 86, FCP 3.6 → 1.8 с."
created: 2026-10-13
edition: "2026-10"
trust:
  F: 2
  G: domain
  R: 0.3
related:
  complements: [DP.M.332]
tags: [vite, react, bundle-splitting, manual-chunks, web-performance, lighthouse]
source: "DS-RollKeep, git fd536e7 (2026-10-08)"
---

# DP.SOTA.034 Vite manual chunks — изоляция helper динамических импортов и vendor-ui

## Проблема

В React SPA с ленивой загрузкой Vite по умолчанию «поднимает» (hoist) helper динамических импортов и vendor-зависимости в крупный чанк тяжёлого модуля. Результат: лендинг вытягивает весь прикладной код при первом открытии.

## Решение

В `vite.config.js` явно выделить отдельные чанки:

| Chunk | Содержимое |
|-------|-----------|
| `vendor-react` | React + helper динамических импортов |
| `vendor-ui` | UI-kit (Radix, Lucide и аналоги) |

Дополнительные практики прогрессивного улучшения:
- Аналитика (PostHog и аналоги) — загружать через `requestIdleCallback`
- Изображения → WebP с `srcset`
- Шрифты → `public/` с long-cache header

## Результат (DS-RollKeep, 2026-10-08)

| Метрика | До | После |
|---------|-----|-------|
| Bundle size | 788 КиБ | ~360 КиБ |
| Lighthouse mobile | 71 | 86 |
| FCP | 3.6 с | 1.8 с |

## Применимость

Любой Vite + React SPA с несколькими тяжёлыми vendor-зависимостями и ленивой загрузкой маршрутов.

## Связи

- Парный метод: DP.M.332 (SSG-гибрид, pre-render лендинга)
- DS-реализация: DS-RollKeep / vite.config.js (git fd536e7)
