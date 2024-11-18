# Структурные паттерны проектирования *

**Напоминание**: звёздочками выделено то, что будет на защите и экзамене.

Все примеры кода написаны на Java, но эти паттерны можно реализовывать на любом языке.

------

Структурные паттерны проектирования описывают, как классы и объекты могут быть составлены в более крупные структуры, обеспечивая гибкость и масштабируемость.

Сегодня мы рассмотрим три ключевых паттерна:

* Proxy (прокси)
* Adapter (адаптер)
* Bridge (мост)


## Proxy (прокси) *

Proxy — это структурный паттерн проектирования, который предоставляет суррогатный объект вместо реального.

Прокси-класс управляет доступом к реальному объекту, добавляя свою логику (например, кеширование, контроль доступа или отложенную инициализацию).

### Основные части паттерна "Прокси" *
Subject (Субъект): Интерфейс или абстрактный класс для реального объекта и прокси.

RealSubject (Реальный субъект): Класс, представляющий настоящий объект, доступ к которому мы контролируем.

Proxy (Прокси): Класс, который выступает в качестве заместителя реального объекта.

### Пример

Ситуация: Вы создаёте систему, где некоторым пользователям нужен доступ к ограниченным ресурсам.

Например, вы работаете с базой данных, но хотите ограничить доступ к её некоторым функциям.

Реализация:

Интерфейс работы с базой данных:

```java
public interface Database {
    void query(String sql);
}
```

Реальный класс базы данных:

```java
public class RealDatabase implements Database {
    @Override
    public void query(String sql) {
        System.out.println("Executing query: " + sql);
    }
}
```

Прокси с проверкой доступа:

```java
public class DatabaseProxy implements Database {
    private RealDatabase realDatabase;
    private boolean hasAccess;

    public DatabaseProxy(boolean hasAccess) {
        this.realDatabase = new RealDatabase();
        this.hasAccess = hasAccess;
    }

    @Override
    public void query(String sql) {
        if (hasAccess) {
            realDatabase.query(sql);
        } else {
            System.out.println("Access denied. Query cannot be executed.");
        }
    }
}
```

Использование:

```java
public class Main {
    public static void main(String[] args) {
        Database userDb = new DatabaseProxy(false);
        Database adminDb = new DatabaseProxy(true);

        userDb.query("SELECT * FROM users"); // Вывод: Access denied. Query cannot be executed.
        adminDb.query("SELECT * FROM users"); // Вывод: Executing query: SELECT * FROM users
    }
}
```

## Adapter (адаптер) *

Adapter — это структурный паттерн проектирования, который позволяет объектам с несовместимыми интерфейсами работать вместе.

### Основные части паттерна "Адаптер" *

Target (Цель): Интерфейс, ожидаемый клиентом.

Adaptee (Адаптируемый): Класс с несовместимым интерфейсом, который нужно адаптировать.

Adapter (Адаптер): Класс, который реализует интерфейс Target и преобразует запросы клиента в формат, понятный Adaptee.

### Пример

Интеграция сторонней библиотеки

Ситуация: Вы используете библиотеку для обработки данных, но её интерфейс не совпадает с вашими требованиями.

Реализация:

Сторонний класс библиотеки:

```java
public class ExternalLogger {
    public void logMessage(String msg) {
        System.out.println("External log: " + msg);
    }
}
```

Ваш целевой интерфейс:

```java
public interface Logger {
    void log(String message);
}
```

Адаптер для интеграции:

```java
public class LoggerAdapter implements Logger {
    private ExternalLogger externalLogger;

    public LoggerAdapter(ExternalLogger externalLogger) {
        this.externalLogger = externalLogger;
    }

    @Override
    public void log(String message) {
        externalLogger.logMessage(message); // Адаптируем метод
    }
}
```

Использование:

```java
public class Main {
    public static void main(String[] args) {
        ExternalLogger externalLogger = new ExternalLogger();
        Logger logger = new LoggerAdapter(externalLogger);

        logger.log("This is a test message."); // Вывод: External log: This is a test message.
    }
}
```

## Bridge (мост) *

Bridge — это структурный паттерн проектирования, который разделяет абстракцию (интерфейс) и реализацию (конкретные детали) на разные иерархии, чтобы их можно было изменять независимо.

### Основные части паттерна "Мост" *

Abstraction (Абстракция): Базовый интерфейс для управления объектами.

Implementor (Реализатор): Интерфейс для конкретной реализации.

ConcreteImplementor (Конкретный реализатор): Конкретная реализация интерфейса Implementor.

RefinedAbstraction (Расширенная абстракция): Конкретная реализация интерфейса Abstraction.

### Пример
Устройства вывода данных
Ситуация: Вы разрабатываете систему вывода данных, и ваша архитектура должна поддерживать как разные типы устройств (например, монитор и принтер), так и форматы данных (текст, изображения).

Реализация:

Интерфейс устройства:

```java
public interface Device {
    void print(String data);
}
```

Конкретные устройства:

```java
public class Monitor implements Device {
    @Override
    public void print(String data) {
        System.out.println("Displaying on monitor: " + data);
    }
}

public class Printer implements Device {
    @Override
    public void print(String data) {
        System.out.println("Printing to paper: " + data);
    }
}
```

Абстракция вывода:

```java
public abstract class Output {
    protected Device device;

    public Output(Device device) {
        this.device = device;
    }

    public abstract void render(String data);
}
```

Расширенная абстракция:

```java
public class TextOutput extends Output {
    public TextOutput(Device device) {
        super(device);
    }

    @Override
    public void render(String data) {
        device.print("Text: " + data);
    }
}

public class ImageOutput extends Output {
    public ImageOutput(Device device) {
        super(device);
    }

    @Override
    public void render(String data) {
        device.print("Image: [Binary data: " + data + "]");
    }
}
```

Использование:

```java
public class Main {
    public static void main(String[] args) {
        Device monitor = new Monitor();
        Device printer = new Printer();

        Output textOnMonitor = new TextOutput(monitor);
        Output textOnPrinter = new TextOutput(printer);

        textOnMonitor.render("Hello, world!"); // Вывод: Displaying on monitor: Text: Hello, world!
        textOnPrinter.render("Hello, world!"); // Вывод: Printing to paper: Text: Hello, world!

        Output imageOnMonitor = new ImageOutput(monitor);
        imageOnMonitor.render("101010101"); // Вывод: Displaying on monitor: Image: [Binary data: 101010101]
    }
}
```
