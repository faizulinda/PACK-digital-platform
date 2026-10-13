---
id: DP.M.332
name: "SSG Hybrid Pre-render of a React SPA Landing"
name_ru: "SSG-гибрид: pre-render React SPA лендинга при сборке без фреймворка"
name_en: "SSG-hybrid: pre-render a React SPA landing at build time without a framework"
summary: "Скрипт после vite build рендерит лендинг через renderToString и вшивает HTML в dist/index.html; React при загрузке находит готовый DOM и вызывает hydrateRoot. Краулеры без JavaScript видят текст, первый экран не ждёт React. Без перехода на Next.js или Astro."
type: method
domain: digital-platform
pack: PACK-digital-platform
trust: observed
epistemic_stage: 2
category: web-frontend
valid_from: 2026-10-13
status: draft
related:
  see_also: [DP.SOTA.034]
tags: [react, vite, ssg, prerender, hydration, seo, landing]
source: "DS-RollKeep, git 7ce6729 (2026-10-08)"
schema_version: 1
---

# DP.M.332 SSG-гибрид: pre-render React SPA лендинга при сборке без фреймворка

## 1. Определение

Метод гибридного SSG без перехода на Next.js/Astro: скрипт после `vite build` рендерит лендинг через `renderToString` в Node.js и вшивает HTML в `dist/index.html`; React при загрузке находит готовый DOM и вызывает `hydrateRoot` — не перерисовывает.

## 2. Проблема

Стандартный React SPA отдаёт краулерам пустой `<div id="root">`, первый экран появляется только после выполнения бандла — задержка FCP и потеря SEO.

## 3. IPO-паттерн

| Элемент | Описание |
|---------|----------|
| Входы | `dist/index.html` после `vite build`; серверный entrypoint лендинга (без `window`/`document`) |
| Обработка | Node.js: `renderToString(лендинг)` → вставить HTML в `<div id="root">` в dist-файле |
| Выходы | `dist/index.html` с готовым HTML; React делает `hydrateRoot` при загрузке страницы |

## 4. Шаги метода

1. Создать серверный entrypoint лендинга — без обращений к `window`/`document`
2. После `vite build` запустить post-build скрипт: загрузить `dist/index.html`, рендернуть через `renderToString`, вставить HTML
3. Убедиться, что клиентский код вызывает `hydrateRoot`, а не `createRoot` — иначе перерисовка уничтожит преимущество
4. Тест: открыть страницу с отключённым JS — текст лендинга должен быть виден

## 5. Применимость

Vite + React проект, где лендинг статичен, а приложение за ним динамично. Не требует перехода на Next.js или Astro.

## 6. Связанные документы

- DP.SOTA.034 (Vite manual chunks) — парный паттерн оптимизации
- DS-реализация: DS-RollKeep / scripts/prerender.js (git 7ce6729)
