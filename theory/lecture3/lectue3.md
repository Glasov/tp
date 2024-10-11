# Порождающие паттерны проектирования *

**Напоминание**: звёздочками выделено то, что будет на защите и экзамене.

Все примеры кода написаны на Java, но эти паттерны можно реализовывать на любом языке.

------

Паттерны проектирования - это проверенные временем решения для типичных задач, которые часто встречаются в разработке программного обеспечения.
Они помогают разработчикам писать код более эффективно и поддерживать его легче.

По своему предназначению они разделяются на три вида:

Порождающие паттерны: определяют механизмы создания объектов, которые повышают гибкость и повторное использование существующего кода

Структурные паттерны: определяют способы составления объектов и классов в более крупные структуры

Поведенческие паттерны: определяют механизмы взаимодействия между объектами и распределения обязанностей между ними

В этой лекции поговорим о порождающих паттернах проектирования.
Их много, но рассмотрим пока что только:

* Singleton (одиночка)
* Factory method (Фабричный метод)
* Abstract factory (Абстрактная фабрика)
* Builder (Строитель)

## Singleton (одиночка) *
Паттерн, который гарантирует, что у класса есть только один экземпляр, и предоставляет глобальную точку доступа к этому экземпляру.

Этот паттерн полезен в ситуациях, когда необходимо контролировать доступ к ресурсам, таким как файлы, базы данных или конфигурационные параметры,
и обеспечить, что эти ресурсы используются только одним экземпляром класса.

Пример реализации на Java:

```Java
public class Singleton {
    // Приватное статическое поле для хранения единственного экземпляра
    private static Singleton instance;

    // Приватный конструктор, чтобы предотвратить создание экземпляров извне
    private Singleton() {
        // Инициализация, если необходимо
    }

    // Статический метод для получения единственного экземпляра
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    // Другие методы класса
    public void someMethod() {
        // Реализация метода
    }
}
```

Можно реализовывать его по-разному:
инициализировать объект сразу или по запросу, делать класс потокобезопасным или нет

## Factory method (Фабричный метод) *
Паттерн, который определяет интерфейс для создания объекта, но позволяет подклассам изменять тип создаваемого объекта.

Этот паттерн позволяет делегировать создание объектов подклассам, что делает код более гибким и расширяемым.

Пример использования:

Предположим, что у нас есть приложение для создания различных типов логгеров (например, для файлового логирования и консольного логирования).
Мы хотим, чтобы наше приложение могло легко добавлять новые типы логгеров без изменения существующего кода.

```java
// Интерфейс для продукта (логгера)
interface Logger {
    void log(String message);
}

// Конкретный продукт: Файловый логгер
class FileLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Logging to file: " + message);
    }
}

// Конкретный продукт: Консольный логгер
class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("Logging to console: " + message);
    }
}

// Создатель (фабрика)
abstract class LoggerFactory {
    // Фабричный метод
    public abstract Logger createLogger();

    // Метод, использующий фабричный метод
    public void logMessage(String message) {
        Logger logger = createLogger();
        logger.log(message);
    }
}

// Конкретный создатель: Фабрика для файлового логгера
class FileLoggerFactory extends LoggerFactory {
    @Override
    public Logger createLogger() {
        return new FileLogger();
    }
}

// Конкретный создатель: Фабрика для консольного логгера
class ConsoleLoggerFactory extends LoggerFactory {
    @Override
    public Logger createLogger() {
        return new ConsoleLogger();
    }
}
```

## Abstract factory (Абстрактная фабрика) *

Паттерн, который предоставляет интерфейс для создания множества взаимосвязанных или взаимозависимых объектов, независимо от их конкретных реализаций.

Этот паттерн позволяет создавать объекты, которые могут работать вместе, обеспечивая при этом гибкость и расширяемость.

Пример реализации на Java:

```java
// Интерфейс для продукта A
interface Button {
    void paint();
}

// Конкретный продукт A1
class WindowsButton implements Button {
    @Override
    public void paint() {
        System.out.println("You have created a Windows button.");
    }
}

// Конкретный продукт A2
class MacButton implements Button {
    @Override
    public void paint() {
        System.out.println("You have created a Mac button.");
    }
}

// Интерфейс для продукта B
interface Checkbox {
    void paint();
}

// Конкретный продукт B1
class WindowsCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("You have created a Windows checkbox.");
    }
}

// Конкретный продукт B2
class MacCheckbox implements Checkbox {
    @Override
    public void paint() {
        System.out.println("You have created a Mac checkbox.");
    }
}

// Интерфейс абстрактной фабрики
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Конкретная фабрика 1
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

// Конкретная фабрика 2
class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }

    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Клиентский код
public class Application {
    private Button button;
    private Checkbox checkbox;

    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }

    public void paint() {
        button.paint();
        checkbox.paint();
    }
}

// Пример использования
public class Main {
    public static void main(String[] args) {
        Application app1 = new Application(new WindowsFactory());
        app1.paint();

        Application app2 = new Application(new MacFactory());
        app2.paint();
    }
}
```

В этом примере мы обстрагировались от конкретных реализаций Checkbox и Button, работаем с интерфейсами, и если захотим поддерживать, например, линукс,
не придётся переписывать логику всего приложения, а просто создать новую фабрику - и всё

## Builder (Строитель) *

Паттерн, который предоставляет способ создания сложных объектов поэтапно.

Он позволяет разделить процесс создания объекта на несколько шагов, что делает его более гибким и управляемым.

Это особенно полезно, когда объект имеет много параметров или его создание требует выполнения нескольких действий.

Пример на языке Java:

```java
// Продукт
class Pizza {
    private String dough;
    private String sauce;
    private String topping;

    public void setDough(String dough) {
        this.dough = dough;
    }

    public void setSauce(String sauce) {
        this.sauce = sauce;
    }

    public void setTopping(String topping) {
        this.topping = topping;
    }

    @Override
    public String toString() {
        return "Pizza{" +
                "dough='" + dough + '\'' +
                ", sauce='" + sauce + '\'' +
                ", topping='" + topping + '\'' +
                '}';
    }
}

// Интерфейс строителя
interface PizzaBuilder {
    void buildDough();
    void buildSauce();
    void buildTopping();
    Pizza getResult();
}

// Конкретный строитель
class HawaiianPizzaBuilder implements PizzaBuilder {
    private Pizza pizza;

    public HawaiianPizzaBuilder() {
        this.pizza = new Pizza();
    }

    @Override
    public void buildDough() {
        pizza.setDough("cross");
    }

    @Override
    public void buildSauce() {
        pizza.setSauce("mild");
    }

    @Override
    public void buildTopping() {
        pizza.setTopping("ham+pineapple");
    }

    @Override
    public Pizza getResult() {
        return pizza;
    }
}

// Директор
class PizzaDirector {
    private PizzaBuilder builder;

    public PizzaDirector(PizzaBuilder builder) {
        this.builder = builder;
    }

    public void constructPizza() {
        builder.buildDough();
        builder.buildSauce();
        builder.buildTopping();
    }
}

// Пример использования
public class Main {
    public static void main(String[] args) {
        PizzaBuilder builder = new HawaiianPizzaBuilder();
        PizzaDirector director = new PizzaDirector(builder);

        director.constructPizza();
        Pizza pizza = builder.getResult();

        System.out.println(pizza);
    }
}
```

Здесь получаем, что пиццу мы готовим поэтапно: сначала занимаемся тестом, потом соусом, а потом - начинкой.
Тем самым мы разбили создание сложного объекта на несколько более простых этапов
