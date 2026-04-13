# SQL Fundamentals

Короткая точка входа: куда идти за **разбором** и шаблонами.

## С чего начать (основы по частям)
- [Основы SQL — обзор](../topics/basics/README.md) — карта: структура запроса, `NULL`, типы, имена и CTE.
- [Структура запроса](../topics/query-structure/README.md) — `SELECT`, `FROM`, `WHERE`, сортировка, лимит.
- [«NULL» и уникальность](../topics/null-and-uniqueness/README.md) — логика условий, `COALESCE`, `DISTINCT`.
- [Joins](../topics/joins/README.md) — соединения, `EXISTS`, условие в `ON` vs `WHERE`.
- [Aggregation](../topics/aggregation/README.md) — `GROUP BY`, `HAVING`, агрегаты и джойны.
- [Indexes](../topics/indexes/README.md) — индексы и типичные ограничения планировщика.

## Быстрые списки
- [Шаблоны запросов](../cheatsheets/query-patterns.md) — заготовки под задачу.
- [Повседневная аналитика](../recipes/daily-analytics-tasks.md) — сценарии отчётов.

## Дальше по ситуации
- Медленно или странный план — проверьте структуру запроса и индексы: [Структура запроса](../topics/query-structure/README.md), [Indexes](../topics/indexes/README.md).
- «Не сходятся цифры» или дубли — перепроверьте ключи соединения и агрегирование: [Joins](../topics/joins/README.md), [Aggregation](../topics/aggregation/README.md).

## Снаружи проекта
- [SQL Academy — курс и тренажёр](https://sql-academy.org/ru/guide) — дополнительный интерактивный трек; кратко о внешних источниках — в [разделе SQL](../README.md).
