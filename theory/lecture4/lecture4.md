# Поведенческие паттерны проектирования *

**Напоминание**: звёздочками выделено то, что будет на защите и экзамене.

Все примеры кода написаны на Java, но эти паттерны можно реализовывать на любом языке.

------

Как мы обсудили в третьей лекции, поведенческие паттерны определяют механизмы взаимодействия между объектами и распределения обязанностей между ними.

Их тоже существует много, но в этой лекции рассмотрим только:

* Strategy (стратегия)
* Responsibility chain (цепочка обязанностей)
* Iterator (итератор)

## Strategy (стратегия) *

Паттерн "Стратегия" — это поведенческий паттерн проектирования, который позволяет определить семейство алгоритмов, инкапсулировать каждый из них и сделать их взаимозаменяемыми.

Это позволяет изменять алгоритмы независимо от клиентов, которые ими пользуются.

### Основные части паттерна "Стратегия" *

* Strategy (Стратегия): Интерфейс или абстрактный класс, который определяет метод для выполнения алгоритма.
* ConcreteStrategy (Конкретная стратегия): Конкретные реализации интерфейса или абстрактного класса стратегии. Каждая реализация представляет собой конкретный алгоритм.
* Context (Контекст): Класс, который использует стратегию. Он содержит ссылку на объект стратегии и вызывает метод стратегии для выполнения алгоритма.

В качестве использования паттерна "стратегия" рассмотрим реализации различных алгоритмов сортировки.

Интерфейс стратегии:
```java
public interface SortingStrategy {
    void sort(int[] array);
}
```

Реализация конкретных стратегий:
```java
public class BubbleSortStrategy implements SortingStrategy {
    @Override
    public void sort(int[] array) {
        // Реализация сортировки пузырьком
        System.out.println("Sorting using Bubble Sort");
    }
}

public class QuickSortStrategy implements SortingStrategy {
    @Override
    public void sort(int[] array) {
        // Реализация быстрой сортировки
        System.out.println("Sorting using Quick Sort");
    }
}
```

Создание контекта:
```java
public class Sorter {
    private SortingStrategy strategy;

    public void setStrategy(SortingStrategy strategy) {
        this.strategy = strategy;
    }

    public void sortArray(int[] array) {
        strategy.sort(array);
    }
}
```

Пример использования:
```java
public class Main {
    public static void main(String[] args) {
        Sorter sorter = new Sorter();

        // Использование стратегии сортировки пузырьком
        sorter.setStrategy(new BubbleSortStrategy());
        int[] array1 = {5, 3, 8, 4, 2};
        sorter.sortArray(array1);

        // Использование стратегии быстрой сортировки
        sorter.setStrategy(new QuickSortStrategy());
        int[] array2 = {5, 3, 8, 4, 2};
        sorter.sortArray(array2);
    }
}
```

## Responsibility chain (цепочка обязанностей) *

Поведенческий паттерн проектирования, который позволяет передавать запросы по цепочке обработчиков.

Каждый обработчик решает, может ли он обработать запрос сам или нужно передать его следующему обработчику в цепочке, запросы передаются по цепочке обработчиков до тех пор, пока один из них не обработает запрос

```Java
interface Handler {
    void handleRequest(Request request);
    void setNextHandler(Handler nextHandler);
}

// Конкретный обработчик A
class ConcreteHandlerA implements Handler {
    private Handler nextHandler;

    @Override
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE_A) {
            System.out.println("ConcreteHandlerA handled the request.");
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }

    @Override
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
}

// Конкретный обработчик B
class ConcreteHandlerB implements Handler {
    private Handler nextHandler;

    @Override
    public void handleRequest(Request request) {
        if (request.getType() == RequestType.TYPE_B) {
            System.out.println("ConcreteHandlerB handled the request.");
        } else if (nextHandler != null) {
            nextHandler.handleRequest(request);
        }
    }

    @Override
    public void setNextHandler(Handler nextHandler) {
        this.nextHandler = nextHandler;
    }
}

// Запрос
class Request {
    private RequestType type;

    public Request(RequestType type) {
        this.type = type;
    }

    public RequestType getType() {
        return type;
    }
}

// Типы запросов. В реальной жизни может быть чем-то типа POST или GET
enum RequestType {
    TYPE_A, TYPE_B
}

// Пример использования
public class Main {
    public static void main(String[] args) {
        Handler handlerA = new ConcreteHandlerA();
        Handler handlerB = new ConcreteHandlerB();

        handlerA.setNextHandler(handlerB);

        Request requestA = new Request(RequestType.TYPE_A);
        Request requestB = new Request(RequestType.TYPE_B);

        handlerA.handleRequest(requestA); // Вывод: ConcreteHandlerA handled the request.
        handlerA.handleRequest(requestB); // Вывод: ConcreteHandlerB handled the request.
    }
}
```

## Iterator (итератор) *

Поведенческий паттерн проектирования, который предоставляет способ последовательного доступа к элементам составного объекта (например, коллекции или массиву) независимо от его реализации.

```Java
// Интерфейс итератора
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Конкретный итератор
class ArrayIterator<T> implements Iterator<T> {
    private T[] items;
    private int position;

    public ConcreteIterator(T[] items) {
        this.items = items;
        this.position = 0;
    }

    @Override
    public boolean hasNext() {
        return position < items.length;
    }

    @Override
    public T next() {
        if (this.hasNext()) {
            return items[position++];
        }
        throw new ArrayIndexOutOfBoundsException(); // либо как-то по-другому обрабатывать этот случай, но лучше бросать ошибку
    }
}
```
