# Лаба 6 - Тестирование ПО

На лекции мы обсудили, что такое ORM и посмотрели примеры кода

## Что нужно сделать

Написать простой код используя ORM.

### Сущности

**Category (Категория):**

id (PK) — уникальный идентификатор категории.
name — название категории.

**Product (Продукт):**

id (PK) — уникальный идентификатор продукта.
name — название продукта.
price — цена продукта.
category_id (FK) — ссылка на категорию продукта.

Одна категория может содержать множество продуктов.

Реализовать CRUD-операции:

Создание категорий и продуктов.
Чтение продуктов по категориям.
Обновление категории у продукта.
Удаление категории и всех связанных продуктов.
