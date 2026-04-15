# Поиск и сортировка с коллекциями

## Цель

Показать, как выполнять поиск и сортировку в Java-коллекциях с правильным выбором структуры и API.

Базовые правила:

- Линейный поиск (`contains`, `indexOf`) в `List` обычно `O(n)`.
- Бинарный поиск (`Collections.binarySearch`) работает только на отсортированных данных и имеет сложность `O(log n)`.
- Для сортировки `List` используй `list.sort(comparator)` или `Collections.sort` (обычно `O(n log n)`).

## Суть подхода

- Для разового поиска в небольшом наборе часто достаточно линейного прохода.
- Для повторяющихся поисков сначала сортируем данные, затем используем бинарный поиск.
- Критично держать единый критерий сравнения: один и тот же `Comparator` для сортировки и поиска.
- Если нужно постоянно хранить данные в порядке, чаще уместнее `TreeSet`/`TreeMap`, чем многократная сортировка списка.

## Стандартные `Comparator` в Java

- `Comparator.naturalOrder()` — естественный порядок (`Comparable`).
- `Comparator.reverseOrder()` — обратный естественный порядок.
- `Comparator.comparing(User::field)` — сравнение по полю-ключу.
- `Comparator.comparingInt(...)`, `comparingLong(...)`, `comparingDouble(...)` — сравнение по примитивам без лишнего boxing.
- `thenComparing(...)` — вторичный и последующие ключи сортировки.
- `Comparator.nullsFirst(...)` / `Comparator.nullsLast(...)` — явная политика для `null`.

## Нестандартные случаи: как действовать

- Сначала формализуй правило сравнения в терминах "меньше/равно/больше", а не в терминах набора `if` без инвариантов.
- Для многоуровневой сортировки строй компаратор цепочкой `comparing(...).thenComparing(...)`, чтобы правило было читаемым и предсказуемым.
- Если в данных есть `null`, явно укажи стратегию (`nullsFirst/nullsLast`), иначе поведение легко сломать.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Сортировка + бинарный поиск по тому же компаратору"

    Важно использовать один и тот же критерий сортировки и поиска, иначе результат `binarySearch` будет некорректным.
    Сложность: сортировка `O(n log n)`, один бинарный поиск `O(log n)`.

    ```java
    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.Comparator;
    import java.util.List;

    record User(String name, int score) {}

    List<User> users = new ArrayList<>(List.of(
        new User("Ann", 30),
        new User("Bob", 10),
        new User("Cara", 20)
    ));

    users.sort(Comparator.comparingInt(User::score)); // O(n log n)
    int pos = Collections.binarySearch(
        users,
        new User("", 20),
        Comparator.comparingInt(User::score)
    );
    ```

</div>

## Частые ошибки

- Делать бинарный поиск по неотсортированному списку.
- Использовать `TreeSet`/`TreeMap`, когда нужна только разовая сортировка списка.
- Писать несогласованный `Comparator` (нарушение транзитивности/антисимметрии).

## См. также

- [`Set`](../set/README.md)
- [`Map`](../map/README.md)
- [Равенство, хеширование и порядок](../equality-and-ordering/README.md)
