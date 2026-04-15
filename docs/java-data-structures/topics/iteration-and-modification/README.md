# Обход и изменение коллекций

Стандартный способ обхода — **`Iterator`**, расширенный цикл **`for (T x : collection)`**, методы **`forEach`**, **`removeIf`**. При одновременном структурном изменении коллекции во время итерации «чужим» способом возможен **`ConcurrentModificationException`** (fail-fast поведение у большинства непотокобезопасных коллекций в одном потоке).

## Безопасное удаление во время обхода

- Через **`Iterator.remove()`** после `next()` — согласованно с итератором.
- Или **`removeIf`** / **`Collection.removeIf`** — удаление по предикату без ручного итератора.
- Или собрать элементы для удаления во **временный `List`**, затем вызвать `removeAll` — простой и читаемый приём для небольших коллекций.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Безопасное удаление при итерации"

    Первый способ — через `Iterator.remove()`, второй — через `removeIf`.
    Оба прохода линейные: обычно `O(n)` по числу элементов коллекции.

    ```java
    import java.util.ArrayList;
    import java.util.Iterator;
    import java.util.List;

    List<String> words = new ArrayList<>(List.of("a", "ab", "abc"));

    Iterator<String> it = words.iterator();
    while (it.hasNext()) {
        String w = it.next();
        if (w.length() < 3) {
            it.remove();
        }
    }

    words.removeIf(w -> w.length() < 3);
    ```

</div>

## Частые ошибки

- Удалять из `ArrayList` в цикле по **индексу** с ростом индекса без учёта сдвига — пропуск элементов; проще **`Iterator.remove`** или **`removeIf`**.
- Модифицировать коллекцию из **двух мест** одновременно без синхронизации или concurrent-реализации — гонки и исключения.

## См. также

- [Recipes: итерация и изменение](../../recipes/concurrent-modification-when-iterating.md)
