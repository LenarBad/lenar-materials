# SQL Query Patterns

Один файл — **набор коротких заготовок** для копирования в рабочий запрос. Последовательные объяснения, типовые ошибки и связь с индексами — в [темах](../topics/README.md); сценарии «под отчёт» — в [рецептах](../recipes/README.md).

## См. тематические источники
- [Основы SQL — обзор](../topics/basics/README.md)
- [Структура запроса](../topics/query-structure/README.md)
- [NULL и уникальность](../topics/null-and-uniqueness/README.md)
- [Joins](../topics/joins/README.md)
- [Aggregation](../topics/aggregation/README.md)
- [Indexes](../topics/indexes/README.md)

## Выборка и предикаты

<div class="lenar-tip-grid" markdown="block">

!!! note "Базовый каркас"

    ```sql
    SELECT id, name, created_at
    FROM users
    WHERE deleted_at IS NULL
      AND country = 'RU'
    ORDER BY created_at DESC
    LIMIT 100;
    ```

!!! note "Полуоткрытый интервал по времени"

    ```sql
    SELECT *
    FROM events
    WHERE event_ts >= :start_ts
      AND event_ts <  :end_ts;
    ```

!!! note "`IN`"

    ```sql
    SELECT * FROM orders
    WHERE status IN ('new', 'paid', 'shipped');
    ```

!!! note "`BETWEEN` (границы включены)"

    ```sql
    SELECT * FROM events
    WHERE event_ts::date BETWEEN DATE '2025-04-01' AND DATE '2025-04-30';
    ```

!!! note "`UNION ALL`"

    ```sql
    SELECT id, amt FROM ledger_archive WHERE yr = 2024
    UNION ALL
    SELECT id, amt FROM ledger_current;
    ```

</div>

## Джойны

<div class="lenar-tip-grid" markdown="block">

!!! note "INNER JOIN"

    ```sql
    SELECT u.id, o.id AS order_id
    FROM users u
    INNER JOIN orders o ON o.user_id = u.id
    WHERE u.region = 'EU';
    ```

!!! note "LEFT JOIN + агрегат во вложенном SELECT"

    ```sql
    SELECT u.id, lo.last_order_at
    FROM users u
    LEFT JOIN (
      SELECT user_id, MAX(created_at) AS last_order_at
      FROM orders
      GROUP BY user_id
    ) lo ON lo.user_id = u.id;
    ```

!!! note "Самосоединение (иерархия)"

    ```sql
    SELECT e.name AS emp, m.name AS mgr
    FROM employees e
    LEFT JOIN employees m ON m.id = e.manager_id;
    ```

</div>

## EXISTS и фильтр по агрегату

<div class="lenar-tip-grid" markdown="block">

!!! note "Есть хотя бы одна строка во второй таблице"

    ```sql
    SELECT u.id
    FROM users u
    WHERE EXISTS (
      SELECT 1 FROM orders o WHERE o.user_id = u.id
    );
    ```

!!! note "Нет ни одной"

    ```sql
    SELECT u.id
    FROM users u
    WHERE NOT EXISTS (
      SELECT 1 FROM orders o WHERE o.user_id = u.id
    );
    ```

!!! note "Есть группа с суммой выше порога"

    Коррелированный подзапрос с `HAVING` — альтернатива `IN (подзапрос с GROUP BY)`.

    ```sql
    SELECT u.id, u.email
    FROM users u
    WHERE EXISTS (
      SELECT 1
      FROM orders o
      WHERE o.user_id = u.id
      GROUP BY o.user_id
      HAVING SUM(o.total) > 10000
    );
    ```

</div>

## Агрегация

<div class="lenar-tip-grid" markdown="block">

!!! note "Группы и `HAVING`"

    ```sql
    SELECT status, COUNT(*) AS n, SUM(amount) AS total
    FROM payments
    WHERE paid_at >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY status
    HAVING COUNT(*) >= 10
    ORDER BY total DESC;
    ```

!!! note "Условная сумма"

    ```sql
    SELECT user_id,
           SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS paid_total
    FROM payments
    GROUP BY user_id;
    ```

</div>

## Уникальные значения в группе

<div class="lenar-tip-grid" markdown="block">

!!! note "DAU по дням (PostgreSQL)"

    ```sql
    SELECT date_trunc('day', created_at) AS day,
           COUNT(DISTINCT user_id) AS dau
    FROM events
    GROUP BY 1
    ORDER BY 1;
    ```

</div>

## Оконные функции

<div class="lenar-tip-grid" markdown="block">

!!! note "Первая строка в группе"

    ```sql
    SELECT *
    FROM (
      SELECT id, user_id, amount,
             ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn
      FROM orders
    ) t
    WHERE rn = 1;
    ```

!!! note "Скользящая сумма по строкам окна"

    Разбор с дневной агрегацией — в [рецепте «Скользящая сумма за 7 дней»](../recipes/daily-analytics-tasks.md).

    ```sql
    SELECT day, daily_rev,
           SUM(daily_rev) OVER (
             ORDER BY day
             ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
           ) AS rev_7d
    FROM daily_revenue;
    ```

</div>

## CTE

<div class="lenar-tip-grid" markdown="block">

!!! note "Именованный подзапрос"

    ```sql
    WITH recent AS (
      SELECT * FROM orders WHERE created_at >= CURRENT_DATE - 7
    )
    SELECT user_id, COUNT(*) AS n
    FROM recent
    GROUP BY user_id;
    ```

</div>

## Диагностика плана

<div class="lenar-tip-grid" markdown="block">

!!! note "PostgreSQL: `EXPLAIN` с анализом"

    ```sql
    EXPLAIN (ANALYZE, BUFFERS)
    SELECT ...
    ;
    ```

    Подробнее — в темах [Структура запроса](../topics/query-structure/README.md) и [Indexes](../topics/indexes/README.md).

</div>
