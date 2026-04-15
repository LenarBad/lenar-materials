# Java Collections: краткая шпаргалка

Сжатый ориентир. Подробности — в [topics](../topics/README.md).

## Объявление vs создание

```java
List<String> a = new ArrayList<>();
Set<Integer> s = new HashSet<>();
Map<String, Integer> m = new HashMap<>();
Deque<String> d = new ArrayDeque<>();
```

## Что выбрать (очень грубо)

| Интерфейс / сценарий | Частый выбор |
|--------|----------------|
| `List`: индекс, в основном хвост | `ArrayList` |
| `List`: вставки у итератора в середину | `LinkedList` (или другая структура) |
| `Set`: без гарантии порядка | `HashSet` |
| `Set`: порядок вставки | `LinkedHashSet` |
| `Set`: сортировка элементов | `TreeSet` |
| `Map`: без гарантии порядка ключей | `HashMap` |
| `Map`: порядок вставки / LRU-подобное | `LinkedHashMap` |
| `Map`: сортировка по ключу | `TreeMap` |
| `Deque` FIFO / LIFO | `ArrayDeque` |
| Не FIFO: порядок по `Comparator` / `Comparable` | `PriorityQueue` |

## Контракты

- `HashMap`/`HashSet`: согласованные **`equals`/`hashCode`**.
- `TreeMap`/`TreeSet`/`PriorityQueue`: **`Comparable`** или **`Comparator`**.

## Тематические источники

- [Обзор фреймворка](../topics/collections-framework/README.md)
- [Равенство и порядок](../topics/equality-and-ordering/README.md)
- [Обход и изменение](../topics/iteration-and-modification/README.md)
