---
id: DP.FM.194
name: Windows cp1251-консоль роняет скрипт после успешной записи при выводе многобайтовых символов
type: failure-mode
domain: digital-platform
pack_refs: []
status: active
valid_from: 2026-11-04
schema_version: 1
source: "inbox capture 2026-11-04 (git commit e21185f DS-strategy)"
---

# DP.FM.194 — Windows cp1251-консоль роняет скрипт после успешной записи при выводе «→»

## Описание

Скрипт-генератор успешно записывает файл, а затем падает с `UnicodeEncodeError` при выводе итоговой строки, содержащей многобайтовый символ («→», «✓», «•»). Windows-консоль в кодировке cp1251 не может закодировать эти символы. Вызывающий скрипт интерпретирует ненулевой exit code как «файл не сгенерирован» и пропускает следующий шаг (коммит, деплой).

## Симптом

Файл существует на диске и корректен, но шаг пайплайна после генерации не выполнился — артефакт «не обновился». Ошибка выглядит как «не сгенерировано», хотя генерация прошла успешно.

## Корень

`sys.stdout` в Python-скрипте на Windows с cp1251-консолью вызывает `UnicodeEncodeError` при выводе символов вне кодировки cp1251. Скрипт не устанавливал кодировку stdout явно перед выводом.

## Тест

«Запускается ли скрипт на Windows/CI без гарантии UTF-8 консоли и содержит ли stdout символы вне ASCII?»
- Да → потенциальный crash-after-success, который pipeline воспринимает как ошибку генерации.

## Правило

В Python-скриптах, запускаемых на Windows или в CI без явной UTF-8-консоли, добавить в начало **до** любого вывода:

```python
import sys
sys.stdout.reconfigure(encoding="utf-8", errors="replace")
sys.stderr.reconfigure(encoding="utf-8", errors="replace")
```

`errors="replace"` предотвращает crash: несовместимые символы заменяются `?`, скрипт завершается с кодом 0. Если потеря символов критична — использовать `errors="xmlcharrefreplace"`.

## Пример

```python
# BEFORE: crashes on a Windows cp1251 console with non-ASCII characters
print(f"Generated {output_file} → {len(entries)} entries")
# UnicodeEncodeError: 'charmap' codec can't encode character '→' in position 42

# AFTER
import sys
sys.stdout.reconfigure(encoding="utf-8", errors="replace")
print(f"Generated {output_file} → {len(entries)} entries")  # the arrow becomes "?" on cp1251, no crash
```

## Связано

- DP.FM.029 — Cross-platform path leak (другая тема: пути, не кодировки)
- DP.FM.031 — Hardcoded OS Path (другая тема)
- WP-247 (DS-strategy, scripts/generate-executor-catalog.py, issue #1411)
