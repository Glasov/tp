# Тестирование ПО *

**Напоминание**: звёздочками выделено то, что будет на защите и экзамене.

Все примеры кода написаны на Java, но эти паттерны можно реализовывать на любом языке.

------

Автоматизированные тесты — это программы, которые проверяют правильность работы другого программного кода.

Они позволяют исключить человеческий фактор и ускоряют процесс разработки.

Зачем нужны:

* Экономия времени: после написания автотестов их можно запускать многократно, экономя время на ручное тестирование
* Повышение качества: автотесты помогают находить баги до того, как продукт попадёт к пользователям
* Поддержка старого кода: они проверяют, что изменения не сломали функциональность

## Виды тестирования ПО *

### Юнит-тесты

Необходимы, чтобы проверить отдельный, изолированный компонент кода (функцию, метод, класс). При этом зависимости (сервисы, базы данных и т.д.) замоканы.
Они самые простые в написании и использовании.

### Интеграционные тесты

Необходимы для проверки взаимодействия между несколькими компонентами - сервисами, системами и т.д.

В отличие от юнит-тестов, зависимости НЕ мокаются - поднимаются БД, API и т.д.

### Системные тесты

Проверка всей системы в целом на работоспособность. Выполняются в окружении, максимально приближенном к реальному и охватывают полный цикл работы системы.

### Регрессионные тесты

Регрессионные тесты используются для проверки, что новые изменения не сломали существующий функционал. Могут включать в себя юнит-тесты, интеграционные тесты и системные тесты.

Суть в их запуске после внесения изменений в проект.

### Тестирование производительности

Проверяют, как система справляется с нагрузкой и остаётся ли производительной.

Проверяет время отклика, количество обрабатываемых запросов, устойчивость к высокой нагрузке.

Может включать тесты на нагрузку (Load Testing), стресс-тесты (Stress Testing) и тесты на масштабируемость (Scalability Testing).

### Smoke-тестирование

Быстрая проверка, что основные функции приложения работают и система не сломана. Как правило это просто запуск приложения для проверки, что оно хотя бы запускается.

Проверяются самые простые сценарии, не вникая в подробности.

### End-To-End (E2E) тесты

Проверить весь пользовательский путь от начала до конца. Как правило используется несколько компонентов - фронт, бекенд, база данныых - и, соответственно, из-за этого они могут долго выполняться.

## Принципы автотестов, TDD *

Из принципов тестирования можно выделить:
* Изолированность. Тест проверяет только одну функцию.
* Повторяемость. Тест должен выдавать одинаковые результаты в любых условиях.
* Понятность. Код теста должен быть понятен даже новичку.
* Быстрота. Чем быстрее выполняются тесты, тем чаще их можно запускать (как правило, тесты выполняются при каждом коммите).

TDD (Test-Driven Development):
Это методология, где тесты пишутся перед кодом. Примерный цикл:

Написать тест, который заведомо падает.

Написать минимально возможный код для прохождения теста.

Рефакторить код, сохраняя тесты рабочими.

В итоге получаем рабочую фичу, полную покрытую тестами, и заранее описанную функциональность - ещё до самой разработки.

## Основы написания тестов *

Для каждого языка программирования как правило используются свои библиотеки для написания тестов, мы будем рассматривать только юнит-тесты.

Все юнит-тесты разделяются на три блока:
* Подготовка (arrange) - задаются начальные параметры и моки. Например, задача seed для рандома, мокирование репозитория
* Действие (act) - тестируемый код выполняется
* Проверка (assert) - проверка результата

В юнит-тестах зависимости мокаются. Это означает, что создаются объекты, имеющие тот же интерфейс, что и настоящие, но которые полностью контролируем мы: мы определяем, при каких вводных данных что каждый метод должен вернуть.

Получается такой "фейковый" объект, с которым можно работать как с настоящим.

## Примеры

Во всех примерах проверяется работа сервиса с замоканным репозиторием.

В коде производятся две проверки:

1. получение пользователя по id
2. проверка выбрасывания ошибки при ненайденном пользователе

### Python

```python
import unittest
from unittest.mock import MagicMock

class TestUserService(unittest.TestCase):
    def setUp(self):
        self.mock_repository = MagicMock(UserRepository)
        self.service = UserService(self.mock_repository)

    def test_get_user_name_success(self):
        # Arrange (Подготовка)
        self.mock_repository.get_user_by_id.return_value = {"id": 1, "name": "Alice"}

        # Act (Действие)
        user_name = self.service.get_user_name(1)

        # Assert (Проверка)
        self.assertEqual(user_name, "Alice")
        self.mock_repository.get_user_by_id.assert_called_once_with(1)

    def test_get_user_name_not_found(self):
        self.mock_repository.get_user_by_id.return_value = None

        with self.assertRaises(ValueError) as context:
            self.service.get_user_name(1)
        self.assertEqual("User not found", str(context.exception))
        self.mock_repository.get_user_by_id.assert_called_once_with(1)

class UserRepository:
    def get_user_by_id(self, user_id):
        raise NotImplementedError()

class UserService:
    def __init__(self, repository):
        self.repository = repository

    def get_user_name(self, user_id):
        user = self.repository.get_user_by_id(user_id)
        if user:
            return user["name"]
        else:
            raise ValueError("User not found")
```

### Java

В коде используется библиотека

`org.mockito:mockito-core`

```Java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

class UserRepository {
    public User getUserById(int userId) {
        throw new UnsupportedOperationException("Not implemented");
    }
}

class UserService {
    private final UserRepository repository;

    public UserService(UserRepository repository) {
        this.repository = repository;
    }

    public String getUserName(int userId) {
        User user = repository.getUserById(userId);
        if (user != null) {
            return user.getName();
        } else {
            throw new IllegalArgumentException("User not found");
        }
    }
}

class User {
    private final int id;
    private final String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class UserServiceTest {
    private UserRepository mockRepository;
    private UserService service;

    @BeforeEach
    public void setUp() {
        mockRepository = Mockito.mock(UserRepository.class);
        service = new UserService(mockRepository);
    }

    @Test
    public void testGetUserNameSuccess() {
        // Arrange
        Mockito.when(mockRepository.getUserById(1)).thenReturn(new User(1, "Alice"));

        // Act
        String userName = service.getUserName(1);

        // Assert
        assertEquals("Alice", userName);
        Mockito.verify(mockRepository, Mockito.times(1)).getUserById(1);
    }

    @Test
    public void testGetUserNameNotFound() {
        // Arrange
        Mockito.when(mockRepository.getUserById(1)).thenReturn(null);

        // Act & Assert
        Exception exception = assertThrows(IllegalArgumentException.class, () -> service.getUserName(1));
        assertEquals("User not found", exception.getMessage());
        Mockito.verify(mockRepository, Mockito.times(1)).getUserById(1);
    }
}
```

### Go

Для запуска необходимо перенести этот код в файл с постфиксом `_test.go` (например `user_test.go`) и вызвать `go test`

```go
package main

import (
	"errors"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/mock"
)

type User struct {
	ID   int
	Name string
}

type UserRepository interface {
	GetUserById(userId int) (*User, error)
}

type UserService struct {
	repository UserRepository
}

func (s *UserService) GetUserName(userId int) (string, error) {
	user, err := s.repository.GetUserById(userId)
	if err != nil {
		return "", err
	}
	if user != nil {
		return user.Name, nil
	}
	return "", errors.New("user not found")
}

type MockUserRepository struct {
	mock.Mock
}

func (m *MockUserRepository) GetUserById(userId int) (*User, error) {
	args := m.Called(userId)
	if args.Get(0) != nil {
		return args.Get(0).(*User), args.Error(1)
	}
	return nil, args.Error(1)
}

func TestGetUserNameSuccess(t *testing.T) {
	mockRepo := new(MockUserRepository)
	service := UserService{repository: mockRepo}

	mockRepo.On("GetUserById", 1).Return(&User{ID: 1, Name: "Alice"}, nil)

	userName, err := service.GetUserName(1)
	assert.NoError(t, err)
	assert.Equal(t, "Alice", userName)
	mockRepo.AssertCalled(t, "GetUserById", 1)
}

func TestGetUserNameNotFound(t *testing.T) {
	mockRepo := new(MockUserRepository)
	service := UserService{repository: mockRepo}

	mockRepo.On("GetUserById", 1).Return(nil, nil)

	_, err := service.GetUserName(1)
	assert.Error(t, err)
	assert.EqualError(t, err, "user not found")
	mockRepo.AssertCalled(t, "GetUserById", 1)
}
```

### C#

```cs
using System;
using Moq;
using NUnit.Framework;

public class User {
    public int Id { get; set; }
    public string Name { get; set; }
}

public interface IUserRepository {
    User? GetUserById(int userId);
}

public class UserService {
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository) {
        _repository = repository;
    }

    public string GetUserName(int userId) {
        var user = _repository.GetUserById(userId);
        if (user != null) {
            return user.Name;
        }
        throw new Exception("User not found");
    }
}

[TestFixture]
public class UserServiceTest {
    private Mock<IUserRepository> _mockRepository;
    private UserService _service;

    [SetUp]
    public void SetUp() {
        _mockRepository = new Mock<IUserRepository>();
        _service = new UserService(_mockRepository.Object);
    }

    [Test]
    public void GetUserName_Success() {
        // Arrange
        _mockRepository.Setup(repo => repo.GetUserById(1)).Returns(new User { Id = 1, Name = "Alice" });

        // Act
        var userName = _service.GetUserName(1);

        // Assert
        Assert.That(userName, Is.EqualTo("Alice"));
        _mockRepository.Verify(repo => repo.GetUserById(1), Times.Once);
    }

    [Test]
    public void GetUserName_NotFound() {
        // Arrange
        _mockRepository.Setup(repo => repo.GetUserById(1)).Returns((User) null);

        // Act & Assert
        var ex = Assert.Throws<Exception>(() => _service.GetUserName(1));
        Assert.That(ex.Message, Is.EqualTo("User not found"));
        _mockRepository.Verify(repo => repo.GetUserById(1), Times.Once);
    }
}
```

### C++

```cpp
#include <gtest/gtest.h>
#include <gmock/gmock.h>
#include <stdexcept>
#include <string>
#include <map>
#include <optional>
#include <memory>

// Базовый класс репозитория
class UserRepository {
public:
    virtual ~UserRepository() = default;
    virtual std::optional<std::map<std::string, std::string>> getUserById(int userId) = 0;
};

// Mock-класс для репозитория
class MockUserRepository : public UserRepository {
public:
    MOCK_METHOD((std::optional<std::map<std::string, std::string>>), getUserById, (int userId), (override));
};

// Сервисный класс
class UserService {
public:
    explicit UserService(std::shared_ptr<UserRepository> repository) : repository_(std::move(repository)) {}

    std::string getUserName(int userId) {
        auto user = repository_->getUserById(userId);
        if (user.has_value()) {
            return user.value().at("name");
        } else {
            throw std::runtime_error("User not found");
        }
    }

private:
    std::shared_ptr<UserRepository> repository_;
};

// Тесты
TEST(UserServiceTest, GetUserNameSuccess) {
    // Arrange
    auto mockRepository = std::make_shared<MockUserRepository>();
    UserService service(mockRepository);

    // Act
    EXPECT_CALL(*mockRepository, getUserById(1))
        .WillOnce(testing::Return(std::map<std::string, std::string>{{"id", "1"}, {"name", "Alice"}}));

    // Assert
    std::string userName = service.getUserName(1);
    EXPECT_EQ(userName, "Alice");
}

TEST(UserServiceTest, GetUserNameNotFound) {
    // Arrange
    auto mockRepository = std::make_shared<MockUserRepository>();
    UserService service(mockRepository);

    // Act + Assert
    EXPECT_CALL(*mockRepository, getUserById(1))
        .WillOnce(testing::Return(std::nullopt));

    EXPECT_THROW({
        try {
            service.getUserName(1);
        } catch (const std::runtime_error &e) {
            EXPECT_STREQ("User not found", e.what());
            throw;
        }
    }, std::runtime_error);
}
```

На С++ как всегда больно писать, вот примерные шаги, чтобы заставить код сверху работать:

1. устанавливаем библиотеки gtest и gmock (если не используете cmake, нужно установить `libgtest-dev` и `libgmock-dev`)
2. Компилируем этот чудесный код командой `g++ -std=c++17 -o test main.cpp -lgtest -lgmock -lpthread`
3. Запускаем при помощи команды `./test`

Если с С++ проблемы, можете почитать статью: https://learn.microsoft.com/en-us/visualstudio/test/writing-unit-tests-for-c-cpp?view=vs-2022

Если она не поможет - пишите мне
