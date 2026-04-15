# Обзор Collections Framework

Большинство стандартных структур данных в Java сосредоточено в пакете **`java.util`**: общие **интерфейсы** и **реализации** под разные сценарии. На практике полезно объявлять поля и параметры как интерфейс (`List`, `Map`, …), а конкретный класс (`ArrayList`, `HashMap`, …) выбирать в одном месте создания.

## Интерфейсы и реализации

| Интерфейс | Назначение (кратко) | Частые реализации |
|-----------|---------------------|-------------------|
| `Collection` | Базовый контракт для коллекций элементов (`add`, `remove`, `contains`, `iterator`, `size`) | напрямую обычно не создают; используют наследников |
| `List` | Упорядоченная последовательность, допускает дубликаты, доступ по индексу | `ArrayList`, `LinkedList` |
| `Set` | Элементы без дубликатов по `equals` | `HashSet`, `LinkedHashSet`, `TreeSet` |
| `Map` | Ключ → значение, ключ уникален | `HashMap`, `LinkedHashMap`, `TreeMap` |
| `Queue` | FIFO и расширения | `ArrayDeque`, `LinkedList`, `PriorityQueue` |
| `Deque` | Добавление и извлечение с обоих концов | `ArrayDeque`, `LinkedList` |

## Иерархия вокруг `Collection`

- **`Collection`** — общий интерфейс для структур «набор элементов».
- **Наследуют `Collection`**: `List`, `Set`, `Queue` (а значит и `Deque` через `Queue`).
- **`Map` не наследует `Collection`**: у `Map` отдельная модель данных (пары ключ-значение) и отдельный API.

## Массив и коллекции

- **Массив `T[]`** — фиксированная длина после создания; примитивы (`int[]`) без обёрток.
- **Коллекции** — динамический размер и богатый API; в generic-коллекциях элементы и ключи обычно ссылочного типа (`Integer`, `String`, ваши классы). Плотное хранение примитивов без обёрток в стандартной библиотеке — массивы или внешние библиотеки.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Интерфейс слева, реализация справа"

    Показывает базовый принцип Collections Framework: работаем через контракты `List`/`Set`/`Map`.
    По операциям в примере: `ArrayList.add` — амортизированно `O(1)`, `new HashSet<>(names)` — `O(n)`, `HashMap.put` — в среднем `O(1)`.

    ```java
    import java.util.ArrayList;
    import java.util.HashMap;
    import java.util.HashSet;
    import java.util.List;
    import java.util.Map;
    import java.util.Set;

    List<String> names = new ArrayList<>();
    names.add("Ann");
    names.add("Bob");

    Set<String> unique = new HashSet<>(names);

    Map<String, Integer> scores = new HashMap<>();
    scores.put("Ann", 10);
    scores.put("Bob", 9);
    ```

</div>

Здесь типы слева — контракты (`List`, `Set`, `Map`), справа — конкретные реализации.

## Дальше по темам

- [`List`](../list/README.md)
- [`Set`](../set/README.md)
- [`Map`](../map/README.md)
- [`Queue`, `Deque`](../queue-and-deque/README.md)
- [Равенство, хеширование и порядок](../equality-and-ordering/README.md)
- [Обход и изменение](../iteration-and-modification/README.md)

## Частые ошибки

- Смешивать «контракт везде» и «конкретный класс везде»: либо объявляй интерфейс и создавай реализацию точечно, либо осознанно фиксируй реализацию, если это часть публичного API библиотеки.
- Полагаться на недокументированный порядок обхода у `HashMap`/`HashSet`.

## См. также

- [Абстракция](../../../oop/topics/abstraction/README.md) — программирование через интерфейсы коллекций.
