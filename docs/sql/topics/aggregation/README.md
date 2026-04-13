# SQL Aggregation

## Цель
Корректно сворачивать строки в группы и считать метрики: знать разницу между `COUNT(*)` и `COUNT(col)`, понимать роль `HAVING` и отличие от `WHERE`, аккуратно работать с `NULL` и при необходимости использовать условные агрегаты и `COUNT(DISTINCT …)`.

## GROUP BY
- Каждая группа — один набор значений столбцов из списка `GROUP BY`. Если в списке несколько столбцов, группа задаётся **сочетанием** всех перечисленных значений (как составной ключ).
- В `SELECT` и `ORDER BY` на агрегированном запросе: либо столбец входит в `GROUP BY`, либо он **аргумент агрегатной функции** (с оговорками для функциональной зависимости в некоторых режимах СУБД — ориентируйтесь на свой диалект).

<div class="lenar-tip-grid" markdown="block">

!!! note "Пример: одна строка на пару «магазин + день»"

    Внутри группы считаются число заказов и суммарная выручка.

    ```sql
    SELECT store_id,
           order_date::date AS day,
           COUNT(*) AS orders_cnt,
           SUM(amount) AS revenue
    FROM orders
    WHERE status = 'paid'
    GROUP BY store_id, order_date::date
    ORDER BY day, store_id;
    ```

    Пример результата (условные данные, чтобы показать разные кейсы группировки):

    | store_id | day | orders_cnt | revenue |
    | --- | --- | ---: | ---: |
    | 10 | 2025-01-01 | 3 | 1250 |
    | 11 | 2025-01-01 | 1 | 300 |
    | 10 | 2025-01-02 | 2 | 900 |
    | 11 | 2025-01-02 | 4 | 2100 |
    | 12 | 2025-01-02 | 1 | 120 |
    | 12 | 2025-01-03 | 2 | 480 |

    Здесь видно: один магазин в разные дни (`store_id = 12`), несколько магазинов в один день (`2025-01-02`) и разное число строк внутри групп (`orders_cnt`).

!!! note "`GROUP BY` без агрегатов"

    `GROUP BY` можно использовать и без агрегатных функций — чтобы получить уникальные комбинации значений (по смыслу близко к `DISTINCT`):

    ```sql
    SELECT city
    FROM customers
    GROUP BY city;
    ```

    Пример результата (условные данные):

    | city |
    | --- |
    | Moscow |
    | Kazan |
    | Sochi |

    То же самое чаще записывают короче:

    ```sql
    SELECT DISTINCT city
    FROM customers;
    ```

</div>

Если в `SELECT` появится столбец, которого нет ни в `GROUP BY`, ни под агрегатом — в строгих режимах (например `ONLY_FULL_GROUP_BY` в MySQL) запрос будет отклонён; в других диалектах возможны неоднозначные или «произвольные» значения — такой запрос лучше не писать.

<div class="lenar-tip-grid" markdown="block">

!!! warning "Нельзя: столбец не в `GROUP BY` и не под агрегатом"

    ```sql
    SELECT city, customer_name, COUNT(*) AS n
    FROM customers
    GROUP BY city;
    ```

!!! warning "Нельзя: агрегат в `WHERE`"

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    WHERE SUM(amount) > 1000
    GROUP BY user_id;
    ```

!!! warning "Нельзя: алиас из `SELECT` в `WHERE` того же уровня"

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    WHERE revenue > 1000
    GROUP BY user_id;
    ```

!!! note "Корректный вариант через `HAVING`"

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    GROUP BY user_id
    HAVING SUM(amount) > 1000;
    ```

</div>

## WHERE и HAVING: что раньше
| Этап | Что фильтрует | Типичное условие |
| --- | --- | --- |
| `WHERE` | Исходные строки **до** группировки | `order_date >= DATE '2025-01-01'`, `status = 'paid'` |
| `HAVING` | Уже сформированные **группы** | `COUNT(*) >= 30`, `SUM(amount) > 10000` |

Правило: фильтры на исходные строки держите в `WHERE`, а условия на агрегаты — в `HAVING`, чтобы не тащить лишние строки в группировку. В `HAVING` можно ссылаться на агрегаты по имени из списка `SELECT` **только там**, где это поддерживает диалект; надёжнее повторить выражение агрегата или обернуть запрос во внешний уровень.

<div class="lenar-tip-grid" markdown="block">

!!! note "Только `WHERE`: фильтр до группировки"

    Берём только оплаченные заказы за период, а потом считаем метрики по пользователю.

    ```sql
    SELECT user_id,
           COUNT(*) AS paid_orders,
           SUM(amount) AS paid_revenue
    FROM orders
    WHERE status = 'paid'
      AND order_date >= DATE '2025-01-01'
    GROUP BY user_id;
    ```

!!! note "Только `HAVING`: фильтр уже готовых групп"

    Сначала строим группы по пользователям на всех строках, затем оставляем только крупные по сумме.

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    GROUP BY user_id
    HAVING SUM(amount) >= 10000;
    ```

!!! note "`WHERE` + `HAVING` вместе"

    `WHERE` сужает входные строки (например, только `paid`), `HAVING` отбирает только нужные группы.

    ```sql
    SELECT user_id,
           SUM(amount) AS revenue
    FROM orders
    WHERE status = 'paid'
    GROUP BY user_id
    HAVING SUM(amount) >= 5000;
    ```

</div>

## Агрегатные функции (типичный набор)
| Функция | Смысл |
| --- | --- |
| `COUNT(*)` | Число строк в группе. Учитывает и строки, где отдельные столбцы — `NULL`: звёздочка считает **строки целиком**. |
| `COUNT(expr)` | Число строк, где `expr` не `NULL`. Пустая строка `''` обычно считается значением и входит в счёт. |
| `COUNT(1)` | В большинстве СУБД по смыслу близок к `COUNT(*)` — счёт строк группы; выбирайте один стиль в проекте. |
| `SUM`, `AVG` | Обычно **игнорируют** `NULL` в аргументе: в сумму и знаменатель среднего не попадают. |
| `MIN`, `MAX` | Работают и для дат и для строк (лексикографический порядок зависит от правил сравнения/кодировки). |

Если **все** значения в группе для `SUM(expr)` — `NULL`, результат часто **`NULL`**, а не ноль: при выводе и в дальнейших расчётах учитывайте `COALESCE(SUM(expr), 0)`. Для `AVG` при отсутствии непустых значений результат тоже обычно `NULL` — это «среднее по пустому множеству», а не ноль.

```sql
SELECT user_id,
       COALESCE(SUM(amount), 0) AS sum_amount,  -- нет строк или все NULL → 0
       AVG(amount) AS avg_amount                 -- нет непустых amount → NULL
FROM payments
GROUP BY user_id;
```

Возможный результат (условные данные):

| user_id | sum_amount | avg_amount |
| --- | ---: | ---: |
| 10 | 300 | 150 |
| 11 | 0 | NULL |
| 12 | 90 | 45 |

Для `user_id = 11` все `amount` в группе равны `NULL`: поэтому `SUM(amount)` был бы `NULL`, `COALESCE` превращает его в `0`, а `AVG(amount)` остаётся `NULL`.

## Условные агрегаты
Нужно посчитать сумму, среднее или число строк только по части группы — через `CASE` внутри агрегата или через `FILTER` в диалектах вроде PostgreSQL. Для **доли** (например, доля оплаченной суммы) удобно делить условную сумму на общую или использовать `AVG(CASE …)` по индикатору.

<div class="lenar-tip-grid" markdown="block">

!!! note "Условный `COUNT` по статусу"

    Считаем, сколько оплаченных заказов у каждого пользователя.

    ```sql
    SELECT user_id,
           SUM(CASE WHEN status = 'paid' THEN 1 ELSE 0 END) AS paid_orders
    FROM orders
    GROUP BY user_id;
    ```

    В PostgreSQL это же можно записать короче:

    ```sql
    SELECT user_id,
           COUNT(*) FILTER (WHERE status = 'paid') AS paid_orders
    FROM orders
    GROUP BY user_id;
    ```

!!! note "Условный `AVG`: средний чек только по `paid`"

    ```sql
    SELECT user_id,
           AVG(CASE WHEN status = 'paid' THEN amount END) AS avg_paid_amount
    FROM orders
    GROUP BY user_id;
    ```

    `CASE ... THEN amount END` без `ELSE` возвращает `NULL` для не-`paid`, а `AVG` их не учитывает.

</div>

## DISTINCT в агрегатах
`COUNT(DISTINCT col)` — число **различных** не-`NULL` значений в группе. Полезно для метрик вроде «уникальные пользователи за день» при группировке по дням. Несколько `COUNT(DISTINCT …)` в одном запросе может быть дорогим по плану — при росте данных проверяйте план выполнения и индексы.

## ROLLUP / CUBE / GROUPING SETS
Инструменты для отчётов с подитогами и несколькими уровнями группировки. `ROLLUP(a, b)` даёт итоги по `(a, b)`, по `a` и общий итог; в строках подитогов ключевые столбцы часто заполняются `NULL`, поэтому в отчётах полезны функции вида `GROUPING()` / `GROUPING_ID()` (названия зависят от СУБД), чтобы отличить «настоящий NULL» от «итоговой строки».

```sql
-- Пример для PostgreSQL: выручка по региону и дню + подитог по региону + общий
-- В строках подитогов ключевые столбцы будут NULL — их отличают от «настоящих» NULL
-- с помощью GROUPING() в вашей СУБД при построении отчёта
SELECT region,
       order_date::date AS day,
       SUM(amount) AS revenue
FROM orders
WHERE status = 'paid'
GROUP BY ROLLUP (region, order_date::date);
```

Точный синтаксис и семантика `NULL` в строках итогов — в документации вашей СУБД; проверяйте план и результат на учебных данных.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "`WHERE` + `HAVING` в одном запросе"

    Сначала отбрасываем неподходящие заказы по дате и статусу, затем группируем по пользователю и оставляем только «крупных» по выручке.

    ```sql
    SELECT user_id,
           SUM(amount) AS revenue,
           COUNT(*) AS paid_orders
    FROM orders
    WHERE order_date >= DATE '2025-01-01'
      AND status = 'paid'
    GROUP BY user_id
    HAVING SUM(amount) >= 5000
    ORDER BY revenue DESC;
    ```

!!! note "Условная сумма: `CASE` и `FILTER`"

    Один и тот же смысл «сумма только по оплаченным»; второй вариант короче там, где есть `FILTER`.

    ```sql
    SELECT user_id,
           SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS paid_amount
    FROM payments
    GROUP BY user_id;
    ```

    ```sql
    SELECT user_id, SUM(amount) FILTER (WHERE status = 'paid') AS paid_amount
    FROM payments
    GROUP BY user_id;
    ```

!!! note "Доля оплаченного: `SUM(CASE …)` и `AVG` по индикатору"

    ```sql
    SELECT user_id,
           SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END)
             / NULLIF(SUM(amount), 0) AS paid_share_of_amount,
           AVG(CASE WHEN status = 'paid' THEN 1.0 ELSE 0 END) AS paid_rows_share
    FROM payments
    GROUP BY user_id;
    ```

    `NULLIF` в знаменателе защищает от деления на ноль, если у пользователя нет строк или сумма нулевая.

!!! note "`HAVING`: отсев групп по агрегату"

    Оставляем только категории с достаточным числом наблюдений.

    ```sql
    SELECT category_id, AVG(amount) AS avg_amount, COUNT(*) AS n
    FROM order_items
    GROUP BY category_id
    HAVING COUNT(*) >= 30
    ORDER BY avg_amount DESC;
    ```

!!! note "`COUNT(*)` и `COUNT(col)` в одной группе"

    Иллюстрация: звёздочка — строки, выражение — непустые значения, `DISTINCT` — уникальные значения в группе.

    ```sql
    SELECT user_id,
           COUNT(*) AS rows_in_group,
           COUNT(email) AS non_null_emails,
           COUNT(DISTINCT tag) AS distinct_tags
    FROM user_tags
    GROUP BY user_id;
    ```

!!! note "Сумма по заказам до джойна с широкой таблицей"

    Типичный приём, если дальше возможен джойн «один-ко-многим» к строкам заказа: сначала одна строка на заказ/пользователя с суммой.

    ```sql
    SELECT u.id, COALESCE(s.total_spent, 0) AS total_spent
    FROM users u
    LEFT JOIN (
      SELECT user_id, SUM(amount) AS total_spent
      FROM orders
      WHERE status = 'paid'
      GROUP BY user_id
    ) s ON s.user_id = u.id;
    ```

</div>

## Частые ошибки
- Ожидание, что `AVG` учитывает нули там, где в данных `NULL` — по умолчанию `NULL` не входит в среднее; явные нули и «нет значения» — разные случаи.
- Путаница `WHERE` и `HAVING`: условие на поле исходной строки в `HAVING` без агрегата часто означает логическую ошибку или лишнюю работу — перенесите в `WHERE`.
- Забытый столбец в `GROUP BY` при несовместимом `SELECT` — ошибка или неочевидная интерпретация в зависимости от режима СУБД.

<div class="lenar-tip-grid" markdown="block">

!!! warning "`WHERE` и `HAVING`: анти-пример и исправление"

    Неправильно (фильтр по строке уехал в `HAVING`):

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    GROUP BY user_id
    HAVING status = 'paid';
    ```

    Правильно:

    ```sql
    SELECT user_id, SUM(amount) AS revenue
    FROM orders
    WHERE status = 'paid'
    GROUP BY user_id;
    ```

!!! warning "Столбец вне `GROUP BY`: анти-пример и исправление"

    Неправильно (`customer_name` не в `GROUP BY` и не агрегирован):

    ```sql
    SELECT city, customer_name, COUNT(*) AS n
    FROM customers
    GROUP BY city;
    ```

    Правильно (агрегируем только по нужному ключу):

    ```sql
    SELECT city, COUNT(*) AS n
    FROM customers
    GROUP BY city;
    ```

</div>

## Связанные материалы
- [Структура запроса](../query-structure/README.md)
- [«NULL» и уникальность](../null-and-uniqueness/README.md)
- [Joins](../joins/README.md)
- [Индексы](../indexes/README.md)

## Где использовать
- [SQL Fundamentals](../../learning/sql-fundamentals.md) — короткий учебный вход
- [Шаблоны запросов](../../cheatsheets/query-patterns.md)
- [Рецепты: повседневная аналитика](../../recipes/daily-analytics-tasks.md)
