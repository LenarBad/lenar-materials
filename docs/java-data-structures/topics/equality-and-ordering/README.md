# Равенство, хеширование и порядок

Коллекции **`HashMap`**, **`HashSet`**, **`LinkedHashMap`**, **`LinkedHashSet`** опираются на контракты **`equals`** и **`hashCode`** для ключей и элементов. **`TreeMap`** и **`TreeSet`** дополнительно требуют **сравнимости** ключей/элементов.

## `equals` и `hashCode`

- Если два объекта **равны** по `equals`, их **`hashCode` обязан совпадать**.
- Совпадение `hashCode` **не гарантирует** равенство по `equals` — возможны коллизии.
- Для ключей в `HashMap` нарушение контракта ведёт к дубликатам, «потерянным» ключам и нестабильному поведению.

## Изменяемые объекты как ключи

Если поля, участвующие в `equals`/`hashCode`, **меняются после вставки** в хеш-таблицу, запись может стать **недостижимой** по исходному ключу. Практично использовать **неизменяемые** ключи (`String`, обёртки, immutable record).

## Порядок: `Comparable` и `Comparator`

- **`TreeSet` / `TreeMap`** упорядочивают элементы/ключи по **естественному порядку** (`Comparable`) или по переданному **`Comparator`**.
- **`PriorityQueue`** использует тот же механизм сравнения для приоритета.

## Примеры

<div class="lenar-tip-grid" markdown="block">

!!! note "Ключ для `HashMap` с корректными `equals/hashCode`"

    Идея примера: логическая идентичность ключа определяется полем `id`, и `hashCode` согласован с `equals`.
    При корректном контракте ключа операции `put/get` в `HashMap` обычно работают за среднее `O(1)`.

    ```java
    import java.util.HashMap;
    import java.util.Map;
    import java.util.Objects;

    final class Key {
        private final String id;

        Key(String id) {
            this.id = id;
        }

        @Override
        public boolean equals(Object o) {
            return o instanceof Key k && Objects.equals(id, k.id);
        }

        @Override
        public int hashCode() {
            return Objects.hash(id);
        }
    }

    Map<Key, String> m = new HashMap<>();
    m.put(new Key("x"), "value");
    ```

</div>

## Частые ошибки

- Генерировать `equals`/`hashCode` только по **части полей**, а потом удивляться дубликатам или пропускам.
- Переопределить `equals` и **забыть** `hashCode` (или наоборот).
- Полагаться на **нестабильный** порядок итерации `HashMap`/`HashSet` в бизнес-логике.

## См. также

- [`Map`](../map/README.md)
- [`Set`](../set/README.md)
