# Основы SQL (обзор)

## Цель
Раздел **«основы»** в этом проекте разбит на **несколько узких тем** ниже: так проще читать и дополнять материал, чем держать всё в одном длинном файле. Эта страница — **карта**: куда перейти за структурой запроса, за `NULL` и `DISTINCT`, за типами и датами, за именами и `WITH`.

| Тема | О чём |
| --- | --- |
| [Структура запроса](../query-structure/README.md) | `SELECT`, `FROM`, `WHERE`, `ORDER BY`, `LIMIT`; логический порядок шагов до группировки. |
| [«NULL», логика и уникальность](../null-and-uniqueness/README.md) | Трёхзначная логика, `IS NULL`, `COALESCE`, `DISTINCT`. |
| [Типы, даты и литералы](../types-and-literals/README.md) | Приведение типов, даты, `BETWEEN`, полуоткрытые интервалы, связь с индексами. |
| [Имена, алиасы и CTE](../identifiers-and-cte/README.md) | Кавычки, алиасы и `WHERE`, обобщённые табличные выражения (`WITH`). |

Дальше по сложности — [Joins](../joins/README.md), [Aggregation](../aggregation/README.md) и остальные темы в [обзоре тем](../README.md).

## Связанные материалы
- [Joins](../joins/README.md)
- [Aggregation](../aggregation/README.md)
- [Индексы](../indexes/README.md)

## Где использовать
- [SQL Fundamentals](../../learning/sql-fundamentals.md)
- [Шаблоны запросов](../../cheatsheets/query-patterns.md)
- [Рецепты: повседневная аналитика](../../recipes/daily-analytics-tasks.md)
