# `Queue`, `Deque`

**`Queue`** — обычно добавление с одного конца и извлечение с другого (типичный FIFO). **`Deque`** (double-ended queue) позволяет эффективно добавлять и снимать с **обоих** концов.

## Контракт `Queue`/`Deque` (главные методы)

- `Queue`: `offer(E)`, `poll()`, `peek()` — добавить, извлечь, посмотреть голову.
- `Deque`: `addFirst/addLast`, `pollFirst/pollLast`, `peekFirst/peekLast`.
- Для стека через `Deque`: `push`, `pop`, `peek`.
- В бизнес-коде безопаснее `offer/poll/peek` (без исключений при пустой структуре), чем `add/remove/element`.

## Основные реализации и стоимость операций

| Реализация | Когда уместна | Преимущества | Ограничения | добавление/удаление у концов | доступ к голове | Особенности |
|-----------|----------------|--------------|-------------|-------------------------------|-----------------|-------------|
| `ArrayDeque` | Выбор по умолчанию для FIFO/LIFO | Быстрый ring-buffer, без лишних узлов | Не хранит `null`, нет произвольного доступа по индексу | амортизированно `O(1)` | `O(1)` | И для `Queue`, и для стека |
| `LinkedList` (`Deque`) | Нужны операции на концах + совместимость с API `List`/`Deque` | Гибкий API, реализация `List` и `Deque` в одном классе | Накладные расходы памяти, хуже locality чем у `ArrayDeque` | `O(1)` | `O(1)` | `get(i)` остаётся `O(n)` |
| `PriorityQueue` | Нужна очередь по приоритету, а не FIFO | Минимум/максимум извлекается быстро | Итерация не в отсортированном порядке; не FIFO | `offer/poll` `O(log n)` | `peek` `O(1)` | Основана на куче |

## Обход

- Для обычной `Queue` чаще обрабатывают элементы в цикле `while ((x = q.poll()) != null)`.
- Для `Deque` можно обходить с головы или хвоста (`iterator()` / `descendingIterator()`).
- У `PriorityQueue` итерация не показывает элементы в порядке приоритета.
- Если нужен именно порядок приоритета, извлекай через `poll()` до опустошения.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "FIFO, стек через `Deque`, очередь с приоритетом"

    В одном примере видно три разных сценария использования очередей в Java.
    Сложность: для `ArrayDeque` операции в конце/начале обычно `O(1)` амортизированно; для `PriorityQueue` `add/poll` — `O(log n)`.

    ```java
    import java.util.ArrayDeque;
    import java.util.Deque;
    import java.util.PriorityQueue;
    import java.util.Queue;

    Queue<String> fifo = new ArrayDeque<>();
    fifo.add("a");
    fifo.add("b");
    fifo.poll();

    Deque<Integer> stack = new ArrayDeque<>();
    stack.push(1);
    stack.push(2);
    stack.pop();

    Queue<Integer> pq = new PriorityQueue<>();
    pq.add(5);
    pq.add(1);
    pq.poll();
    ```

</div>

## Частые ошибки

- Использовать `Stack` в новом коде без веской причины — лучше **`Deque`** и **`ArrayDeque`**.
- Путать **`PriorityQueue`** с FIFO-`Queue` — порядок извлечения задаётся приоритетом, а не порядком вставки.

## См. также

- [`List`](../list/README.md) — `LinkedList` как альтернативный `Deque`.
