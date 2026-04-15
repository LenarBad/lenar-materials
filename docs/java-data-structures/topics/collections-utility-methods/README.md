# Полезные методы `Collections` и `Arrays`

## Цель

Дать быстрый ориентир по часто используемым утилитам из `java.util.Collections` и `java.util.Arrays`, которые помогают писать короче и безопаснее при работе со структурами данных.

## Суть подхода

- Не писать вручную то, что уже есть в стандартной библиотеке (`sort`, `binarySearch`, `copyOf`, `frequency` и т.д.).
- Выбирать утилиту под задачу: порядок, поиск, копирование, read-only view, синхронизация.
- Всегда учитывать контракт метода: например, `binarySearch` требует отсортированного входа, `asList` даёт список фиксированного размера.
- Использовать Big-O из таблиц как быстрый фильтр, но проверять и семантику метода (view vs копия).

## Ключевые методы

### `Collections`

| Метод | Кратко | Big-O (типично) |
| --- | --- | --- |
| `sort(list)` | Сортирует `list` по естественному порядку элементов (`Comparable`). | `O(n log n)` |
| `sort(list, comparator)` | Сортирует `list` по правилу из `comparator`. | `O(n log n)` |
| `binarySearch(list, key[, comparator])` | Ищет `key` в отсортированном `list`; без сортировки результат некорректен. | `O(log n)`* |
| `reverse(list)` | Разворачивает порядок элементов в `list` на месте. | `O(n)` |
| `shuffle(list)` | Перемешивает `list` в случайном порядке. | `O(n)` |
| `min(coll)`, `max(coll)` | Возвращает минимум/максимум в `coll` (естественный порядок или компаратор). | `O(n)` |
| `frequency(coll, x)` | Считает, сколько раз элемент `x` встречается в `coll`. | `O(n)` |
| `disjoint(a, b)` | Возвращает `true`, если у коллекций `a` и `b` нет общих элементов. | до `O(n + m)` |
| `addAll(coll, elements...)` | Добавляет в `coll` все элементы из varargs `elements...`. | `O(k)`** |
| `replaceAll(list, old, new)` | Заменяет в `list` каждое вхождение `old` на `new`. | `O(n)` |
| `unmodifiableList/Set/Map(...)` | Возвращает read-only view поверх переданной коллекции/мапы. | `O(1)` (создание view) |
| `synchronizedList/Set/Map(...)` | Возвращает synchronized-обёртку над коллекцией/мапой. | `O(1)` (создание обёртки) |

\* Для `List` без эффективного random access фактическая стоимость может быть выше из-за доступа по индексу.
\** `k` — число добавляемых элементов; итог зависит от стоимости `add` у конкретной коллекции.

### `Arrays`

| Метод | Кратко | Big-O (типично) |
| --- | --- | --- |
| `sort(array)` | Сортирует массив `array` на месте. | `O(n log n)` |
| `binarySearch(array, key)` | Ищет `key` в отсортированном `array` бинарным поиском. | `O(log n)` |
| `fill(array, value)` | Заполняет все элементы `array` значением `value`. | `O(n)` |
| `copyOf(array, newLength)` | Возвращает копию `array` длиной `newLength`. | `O(newLength)` |
| `copyOfRange(array, from, to)` | Копирует полуинтервал `[from, to)` в новый массив. | `O(to - from)` |
| `asList(elements...)` | Возвращает `List` фиксированного размера поверх переданного массива/элементов. | `O(1)` (создание view) |
| `parallelSort(array)` | Сортирует массив параллельно (актуально на больших данных). | ~`O(n log n)` |
| `setAll(array, i -> value)` | Заполняет массив по генератору от индекса `i`. | `O(n)` |
| `parallelSetAll(array, i -> value)` | То же, но с параллельным заполнением. | `O(n)` (параллельно) |
| `compare(a, b)` | Лексикографически сравнивает два массива. | `O(min(n, m))` |
| `mismatch(a, b)` | Возвращает индекс первого различия массивов (или `-1`). | `O(min(n, m))` |

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Сортировка, разворот, перемешивание, бинарный поиск"

    Используй `binarySearch` только на уже отсортированном списке с тем же компаратором.
    Сложность: сортировка `O(n log n)`, разворот/перемешивание `O(n)`, бинарный поиск `O(log n)`.

    ```java
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.Comparator;
    import java.util.List;

    List<Integer> nums = new ArrayList<>(List.of(4, 1, 7, 2));

    nums.sort(Comparator.naturalOrder());
    Collections.reverse(nums);
    Collections.shuffle(nums);
    int pos = Collections.binarySearch(nums, 4); // корректно только для отсортированного списка
    ```

!!! note "Минимум/максимум, частота, пересечение"

    Быстрый набор для аналитики по коллекции без ручных циклов.
    Типично: `min/max/frequency` — `O(n)`, `disjoint` — до `O(n + m)` в зависимости от реализаций.

    ```java
    import java.util.Collections;
    import java.util.List;
    import java.util.Set;

    List<String> words = List.of("a", "b", "a", "c");
    int countA = Collections.frequency(words, "a");
    String minWord = Collections.min(words);
    String maxWord = Collections.max(words);

    boolean hasIntersection = !Collections.disjoint(
        Set.of("java", "sql"),
        Set.of("docker", "java")
    );
    ```

!!! note "Неизменяемые и синхронизированные обёртки"

    `unmodifiable*` защищает от случайных изменений через этот view.
    `synchronized*` добавляет базовую потокобезопасность на уровне отдельных операций.

    ```java
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;

    List<String> base = new ArrayList<>(List.of("Ann", "Bob"));
    List<String> readonly = Collections.unmodifiableList(base);
    List<String> sync = Collections.synchronizedList(base);
    ```

!!! note "`Arrays` для массивов: сортировка и бинарный поиск"

    Утилиты `Arrays` особенно полезны для примитивных массивов.
    Сложность: сортировка `O(n log n)`, бинарный поиск `O(log n)`.

    ```java
    import java.util.Arrays;

    int[] arr = {9, 1, 4, 7};
    Arrays.sort(arr);
    int idx = Arrays.binarySearch(arr, 4);
    ```

</div>

## Частые ошибки

- Вызывать `Collections.binarySearch` на неотсортированном списке.
- Путать `Collections.unmodifiableList(...)` с глубокой неизменяемостью: базовую коллекцию всё ещё можно менять через исходную ссылку.
- Использовать `synchronizedList` как полную замену concurrent-коллекциям без понимания сценария конкурентного доступа.

## См. также

- [Поиск и сортировка с коллекциями](../search-and-sorting-with-collections/README.md)
- [Обзор Collections Framework](../collections-framework/README.md)
