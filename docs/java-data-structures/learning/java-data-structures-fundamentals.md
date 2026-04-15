# Основы: структуры данных в Java

Краткий маршрут по материалам домена. Детали — в [темах](../topics/README.md).

## Когда использовать этот раздел

- Нужно быстро вспомнить, **какой интерфейс** объявить и **какую реализацию** создать.
- Хотите избежать типовых ошибок с **`equals`/`hashCode`**, порядком обхода и **одновременным изменением** коллекции.

## Пошаговый подход

1. Прочитайте [обзор Collections Framework](../topics/collections-framework/README.md) и таблицу интерфейсов.
2. Выберите семейство: [`List`](../topics/list/README.md), [`Set`](../topics/set/README.md), [`Map`](../topics/map/README.md) или [`Queue`/`Deque`](../topics/queue-and-deque/README.md).
3. Если используете `HashMap`/`HashSet` или деревья — раздел [равенство и порядок](../topics/equality-and-ordering/README.md).
4. Перед сложным обходом с удалением — [обход и изменение](../topics/iteration-and-modification/README.md) и [рецепт](../recipes/concurrent-modification-when-iterating.md).

## Проверка результата

- Публичные поля и параметры объявлены как **`List`/`Map`/…**, а не как конкретный класс, если нет осознанного исключения.
- Для ключей и элементов `Set`/`Map` контракт **`equals`/`hashCode`** соблюдён и задокументирован для команды.
- Нет удаления «в лоб» из коллекции во время обхода без **`Iterator.remove`**, **`removeIf`** или копии.

## Тематические источники

- [Topics](../topics/README.md)
- [Cheatsheet](../cheatsheets/java-collections-quick-reference.md)
