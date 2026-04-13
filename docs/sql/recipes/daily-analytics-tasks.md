# Повседневная аналитика в SQL

Сценарии для типичных запросов к продуктовым и операционным данным. Примеры ориентированы на **PostgreSQL-подобный** синтаксис (`INTERVAL`, `date_trunc`, оконные функции); при переносе на другую СУБД замените функции даты и литералы по документации.

См. тематические источники:
- [Aggregation](../topics/aggregation/README.md)
- [Joins](../topics/joins/README.md)
- [Indexes](../topics/indexes/README.md)

## 1. DAU за календарный период

**Задача:** ежедневное число уникальных пользователей по событиям.

**Идея:** усечь время до дня, посчитать `COUNT(DISTINCT user_id)` по группе.

<div class="lenar-tip-grid" markdown="block">

!!! note "DAU по дням"

    Границы периода — полуоткрытый интервал по `event_ts`.

    ```sql
    SELECT date_trunc('day', event_ts)::date AS day,
           COUNT(DISTINCT user_id) AS dau
    FROM events
    WHERE event_ts >= DATE '2025-04-01'
      AND event_ts <  DATE '2025-05-01'
    GROUP BY 1
    ORDER BY 1;
    ```

</div>

**Проверка:** сумма `dau` по дням не должна сравниваться с «числом событий» — это разные метрики. На больших объёмах индекс по `(event_ts)` или `(event_ts, user_id)` помогает плану ([Indexes](../topics/indexes/README.md)).

## 2. Конверсия шага воронки за период

**Задача:** доля пользователей, дошедших от события `A` до события `B` в одном окне (например, в течение суток после `A`).

**Идея:** множество пользователей с `A` и множество с `B` после `A`; отношение размеров. Ниже — упрощённый каркас (детали зависят от схемы).

<div class="lenar-tip-grid" markdown="block">

!!! note "Когорта A и конверсия в B за 24 часа"

    Параметры `:start` и `:end` задают окно наблюдения за событиями `A`.

    ```sql
    WITH first_a AS (
      SELECT user_id, MIN(event_ts) AS t_a
      FROM events
      WHERE event_type = 'signup'
        AND event_ts >= :start
        AND event_ts < :end
      GROUP BY user_id
    ),
    reached_b AS (
      SELECT DISTINCT e.user_id
      FROM events e
      JOIN first_a fa ON fa.user_id = e.user_id
      WHERE e.event_type = 'first_purchase'
        AND e.event_ts >= fa.t_a
        AND e.event_ts < fa.t_a + INTERVAL '24 hours'
    )
    SELECT
      (SELECT COUNT(*) FROM first_a) AS users_with_a,
      (SELECT COUNT(*) FROM reached_b) AS users_with_b_after_a,
      (SELECT COUNT(*) FROM reached_b)::float
        / NULLIF((SELECT COUNT(*) FROM first_a), 0) AS conversion;
    ```

</div>

**Частая ошибка:** считать конверсию по **событиям**, а не по **уникальным пользователям** — раздувает знаменатель/числитель ([Aggregation](../topics/aggregation/README.md)).

## 3. Первое платёжное событие на пользователя

**Задача:** для каждого пользователя взять первый платёж (по времени).

**Идея:** `ROW_NUMBER()` с партицией по `user_id`, затем фильтр `rn = 1`.

<div class="lenar-tip-grid" markdown="block">

!!! note "Одна строка на пользователя после ранжирования"

    ```sql
    SELECT user_id, amount, paid_at
    FROM (
      SELECT user_id, amount, paid_at,
             ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY paid_at ASC) AS rn
      FROM payments
      WHERE status = 'paid'
    ) t
    WHERE rn = 1;
    ```

</div>

**Связь с джойнами:** если сразу джойнить платежи к заказам один-ко-многим без такого шага, суммы по пользователю могут задвоиться ([Joins](../topics/joins/README.md)).

## 4. Retention 7/28 (идея)

**Задача:** пользователи, вернувшиеся через N дней после «когорты» первого визита.

Упрощённый подход: таблица первых визитов и флаг повторного визита в окне; затем агрегация по когорте. Реализация сильно зависит от определения «возврата» и таймзоны — зафиксируйте бизнес-правило до написания SQL.

**Проверка:** на малых выборках сверьте вручную несколько `user_id`.

## 5. Топ-N внутри каждой категории

**Задача:** три самых дорогих товара по каждой категории.

<div class="lenar-tip-grid" markdown="block">

!!! note "ROW_NUMBER с отсечением по rn"

    ```sql
    SELECT *
    FROM (
      SELECT category_id, product_id, price,
             ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn
      FROM products
      WHERE active = TRUE
    ) t
    WHERE rn <= 3;
    ```

</div>

Альтернатива в PostgreSQL — `DISTINCT ON` с осторожностью к детерминированности сортировки.

## 6. Скользящая сумма за 7 дней по дням

**Задача:** для каждого календарного дня показать сумму выручки не только за этот день, но и **накопленно за последние 7 дней** (или чисто «окно» 7 дней — зависит от определения; ниже — окно из 7 строк по упорядоченным дням).

**Идея:** сначала агрегировать выручку по дням, затем оконная функция `SUM … ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`.

<div class="lenar-tip-grid" markdown="block">

!!! note "Дневная выручка и сумма за 7 дней подряд"

    Предполагается, что в `orders` есть `paid_at` и `amount`. При пропусках дней окно всё равно по **строкам**, не по календарю — при необходимости заполните нули в подзапросе.

    ```sql
    WITH daily AS (
      SELECT paid_at::date AS day, SUM(amount) AS daily_rev
      FROM orders
      WHERE status = 'paid'
        AND paid_at >= DATE '2025-01-01'
        AND paid_at <  DATE '2025-04-01'
      GROUP BY 1
    )
    SELECT day, daily_rev,
           SUM(daily_rev) OVER (
             ORDER BY day
             ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
           ) AS rolling_7d_rev
    FROM daily
    ORDER BY day;
    ```

</div>

**Проверка:** на первых днях окна меньше семи строк — сумма всё равно по доступным; уточните в отчёте, нужна ли неполная «прогревочная» неделя.

## 7. Контроль дубликатов перед отчётом

**Задача:** убедиться, что по заявленному уникальному ключу (email, внешний id) нет повторов — иначе метрики по пользователям разъедутся.

Запросы «найти ключи с `COUNT(*) > 1`» и «вытащить все строки нарушителей» опираются на те же приёмы `GROUP BY` + `HAVING` и аккуратное соединение обратно к исходной таблице.

## Общие рекомендации
- Согласуйте **границы периодов** и таймзону до публикации цифр.
- При «тормозах» снимайте план и смотрите оценку строк.
- Не смешивайте без нужны уровни детализации: сначала ключ, по которому уникальна строка, потом джойны.
