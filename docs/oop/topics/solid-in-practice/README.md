# SOLID на практике

**SOLID** — это пять принципов проектирования, которые помогают держать код понятным, расширяемым и устойчивым к изменениям. Это не закон и не чеклист на каждый класс: принципы применяют там, где растёт сложность, появляются границы подсистем и важна стоимость сопровождения.

**Главная идея:** уменьшить связность между частями системы и сделать так, чтобы типичные изменения (новый сценарий, новая интеграция, новое правило) добавлялись **точечно**, без каскадных правок по всей кодовой базе.

Ниже — краткое имя принципа, что он проверяет в коде, как это обычно выглядит в Java и на что смотреть при ревью.

---

## S — Single Responsibility Principle (единственная ответственность)

**Смысл:** у класса должна быть **одна причина для изменения** — одна зона ответственности, которую можно назвать одной фразой без союза «и».

**Зачем:** когда в одном классе смешаны расчёт, форматирование, отправка по сети и логирование, любая мелкая правка в одной области тащит риски регрессий в другой. Разделение снижает стоимость тестов и ревью.

**Признаки нарушения**
- Файл на сотни строк с разными заголовками по смыслу («расчёт», «отправка», «валидация», «SQL»).
- Имя класса не помещает суть (`Manager`, `Helper`, `Util`).
- Один класс меняют разные команды по разным поводам.

**Пример (Java)**

```java
import java.util.List;

final class OrderReportData {
    private final List<String> lines;

    public OrderReportData(List<String> lines) {
        this.lines = List.copyOf(lines);
    }

    public List<String> lines() {
        return lines;
    }
}

final class OrderReportHtmlFormatter {
    public String toHtml(OrderReportData data) {
        StringBuilder sb = new StringBuilder("<html><body><ul>");
        for (String line : data.lines()) {
            sb.append("<li>").append(escapeHtml(line)).append("</li>");
        }
        sb.append("</ul></body></html>");
        return sb.toString();
    }

    private static String escapeHtml(String s) {
        return s.replace("&", "&amp;").replace("<", "&lt;");
    }
}

final class OrderReportMailer {
    public void sendHtml(String to, String html) {
        // здесь интеграция с почтовым API или шлюзом
    }
}
```

Здесь данные отчёта, разметка и доставка разведены: при смене шаблона HTML не трогают отправку писем.

**Связь с другими темами:** см. [Инкапсуляция](../encapsulation/README.md) — часто после выделения ответственности проще спрятать состояние за понятным API.

---

## O — Open/Closed Principle (открыт для расширения, закрыт для изменения)

**Смысл:** поведение системы можно **расширять** (новые варианты), не **ломая** уже стабильный код путём постоянных правок внутри одного большого метода.

**Зачем:** ветки `if (type == …)` по всему проекту при каждом новом типе требуют правок в нескольких местах. Контракт + новые реализации локализуют изменение.

**Признаки нарушения**
- Один метод с длинным `switch` / цепочкой `if-else` по типам или флагам.
- Добавление фичи = «найти все места, где уже ветвились».

**Пример (Java)**

```java
interface ShippingCostRule {
    int costFor(Order order);
}

final class DomesticShipping implements ShippingCostRule {
    public int costFor(Order order) {
        return 300;
    }
}

final class InternationalShipping implements ShippingCostRule {
    public int costFor(Order order) {
        return 900;
    }
}

final class Checkout {
    private final ShippingCostRule shipping;

    public Checkout(ShippingCostRule shipping) {
        this.shipping = shipping;
    }

    public int totalWithShipping(Order order, int goodsTotal) {
        return goodsTotal + shipping.costFor(order);
    }
}

final class Order {
    // поля заказа для примера
}
```

Новый тариф доставки — новый класс `implements ShippingCostRule`, без правки тела `totalWithShipping`, если контракт достаточен.

**Связь с другими темами:** [Полиморфизм](../polymorphism/README.md), [Композиция](../composition/README.md), [Interface vs Abstract Class](../interface-vs-abstract-class/README.md).

---

## L — Liskov Substitution Principle (принцип подстановки Барбары Лисков)

**Смысл:** объекты подкласса должны **корректно подставляться** вместо объектов базового типа: контракт базового класса (ожидания по предусловиям, постусловиям и инвариантам) не должен ломаться «сюрпризами» в наследнике.

**Зачем:** иначе полиморфизм становится ловушкой: код, написанный против базового типа, падает или ведёт себя нелогично при передаче наследника.

**Признаки нарушения**
- В подклассе метод «переопределён» так, что иногда бросает исключение там, где базовый класс этого не допускал без веской причины.
- Подкласс ослабляет гарантии (например, «счёт только на чтение» наследует «счёт со снятием» и ломает ожидания клиента).

**Пример (идея на Java)**

Базовый тип обещает: «снятие возможно, если хватает средств». Подкласс, который **всегда** бросает `UnsupportedOperationException` в `withdraw`, для клиента, ожидающего обычный счёт, нарушает подстановку. Лучше вынести «только чтение» в отдельный контракт (интерфейс вида `BalanceView`) или не наследовать таким образом.

**Связь с другими темами:** [Наследование](../inheritance/README.md).

---

## I — Interface Segregation Principle (разделение интерфейсов)

**Смысл:** клиенты не должны зависеть от методов, которые им **не нужны**. Лучше несколько узких интерфейсов, чем один «комбайн».

**Зачем:** реализация вынуждена реализовывать пустые или искусственные методы (`throw new UnsupportedOperationException()`), растёт связность, тесты усложняются.

**Признаки нарушения**
- Интерфейс на 20+ методов; половина классов реализует только два.
- Часто встречаются заглушки `UnsupportedOperationException` в реализациях.

**Пример (Java)**

```java
interface OrderReader {
    Order findById(String id);
}

interface OrderWriter {
    void save(Order order);
}

// Кэширующий слой читает, но не пишет — ему не нужен OrderWriter.
final class CachedOrderReader implements OrderReader {
    private final OrderReader delegate;

    public CachedOrderReader(OrderReader delegate) {
        this.delegate = delegate;
    }

    public Order findById(String id) {
        return delegate.findById(id);
    }
}
```

**Связь с другими темами:** [Абстракция](../abstraction/README.md), [Interface vs Abstract Class](../interface-vs-abstract-class/README.md).

---

## D — Dependency Inversion Principle (инверсия зависимостей)

**Смысл:** модули верхнего уровня (политика, use-case) должны зависеть от **абстракций**, а не от конкретных реализаций инфраструктуры. Детали (БД, HTTP, файлы) подключаются снизу.

**Зачем:** бизнес-логику можно тестировать и развивать, подменяя зависимости; смена хранилища не тянет переписывание сервиса целиком.

**Признаки нарушения**
- В доменном сервисе прямой `new PostgresOrderRepository()` или статические вызовы к БД.
- Невозможно подменить время, случайность, сеть в тесте без хаков.

**Пример (Java)**

```java
interface OrderRepository {
    Order findById(String id);
}

interface Clock {
    java.time.Instant now();
}

final class SystemClock implements Clock {
    public java.time.Instant now() {
        return java.time.Instant.now();
    }
}

final class CancelStaleOrdersUseCase {
    private final OrderRepository orders;
    private final Clock clock;

    public CancelStaleOrdersUseCase(OrderRepository orders, Clock clock) {
        this.orders = orders;
        this.clock = clock;
    }

    public void run(String id, java.time.Duration maxAge) {
        Order order = orders.findById(id);
        java.time.Instant created = order.createdAt();
        if (clock.now().isAfter(created.plus(maxAge))) {
            order.cancel();
        }
    }
}

final class Order {
    private final java.time.Instant createdAt;

    public Order(java.time.Instant createdAt) {
        this.createdAt = createdAt;
    }

    public java.time.Instant createdAt() {
        return createdAt;
    }

    public void cancel() {
        // доменное действие
    }
}
```

В тесте подставляют фиктивный `Clock` и in-memory `OrderRepository`, не трогая текст `CancelStaleOrdersUseCase`.

**Связь с другими темами:** [Абстракция](../abstraction/README.md), [Композиция](../composition/README.md).

---

## Как применять SOLID без перегиба

- Начинай с **SRP** и **DIP**, когда появляются боли в тестах и смене интеграций.
- **OCP** и **ISP** усиливают контракты, когда растёт число вариантов поведения и клиентов.
- **LSP** проверяй при **наследовании**: если подкласс «ломает» ожидания базового типа, чаще проблема в модели наследования, а не «плохой SOLID».

Не дроби код на десятки слоёв «ради букв»: если класс маленький, живёт в одном контексте и редко меняется, лишняя абстракция только ухудшит чтение.

---

## См. также

- [Полиморфизм](../polymorphism/README.md)
- [Наследование](../inheritance/README.md)
- [Композиция](../composition/README.md)
- [Инкапсуляция](../encapsulation/README.md)
- [Абстракция](../abstraction/README.md)
- [Interface vs Abstract Class](../interface-vs-abstract-class/README.md)

---

## Частые ошибки

- Требовать «идеальный SOLID» в прототипе и одноразовых скриптах.
- Вводить интерфейсы на каждый класс без сценария подмены и без второй реализации.
- Путать **DIP** с обязательным фреймворком внедрения зависимостей: суть в направлении зависимостей, а не в конкретной библиотеке.
- Игнорировать **LSP** при глубоком наследовании и затем «лечить» это проверками типов в клиентском коде.
