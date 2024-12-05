# Работа с базами данных - ORM *

**Напоминание**: звёздочками выделено то, что будет на защите и экзамене.

------

## Что такое ORM

Почти в любом проекте необходимо иметь базу данных,
иногда даже в десктопных и консольных приложениях (например, sqlite),
а для разработки веб-приложений она просто необходима.

Есть несколько вариантов работы с базой данных: напрямую подключаться к ним, создавать сессии,
управлять полностью всеми процессами, джоинами и прочим. Такой код можно сделать эффективным,
но это очень сложно и требует, соответственно, больших усилий.

Чтобы не тратить попусту силы и сконцентрироваться больше на логике работы самого приложения,
используют ORM (Object-Relational Mapping) - специальные библиотеки, которые позволяют
разработчикам взаимодействовать с базами данных через объектно-ориентированный код.

ORM избавляют от необходимости писать прямые SQL-запросы для большинства операций, делая код
более читаемым, переносимым и удобным в поддержке.

В этой лабе будем использовать postgres, скачать его можно здесь: https://www.postgresql.org/download

Для работы с ним будет необходим клиент, можно скачать любой из списка: https://wiki.postgresql.org/wiki/PostgreSQL_Clients

Советую утилиту `psql`

На маке скачать можно командой
```bash
brew install libpq
```

На линуксе - командой
```bash
sudo apt-get install -y postgresql-client
```

На виндовсе - через установщик: https://www.postgresql.org/download/windows/?ref=timescale.com

Теперь создадим базу данных.

Запустим утилиту psql:

```bash
psql --dbname=postgres
```

(в линуксе используйте `sudo -i -u postgres`)

Откроется командная строка postgres:
`postgres=#`

Теперь создадим нашу базу данных и перейдём в неё:

```sql
CREATE DATABASE mydb;
\c mydb
```

Нам надо создать пользователя, от которого дальше будем взаимодействовать с базой через код

```sql
CREATE ROLE username WITH PASSWORD 'password' SUPERUSER LOGIN;
```

Создадим таблицы

```sql
-- Создание таблицы users
CREATE TABLE users (
    id SERIAL PRIMARY KEY, -- Уникальный идентификатор пользователя (с автоинкрементом)
    name VARCHAR NOT NULL  -- Имя пользователя (обязательное поле)
);

-- Создание таблицы posts
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,        -- Уникальный идентификатор поста (с автоинкрементом)
    title VARCHAR NOT NULL,       -- Заголовок поста (обязательное поле)
    content VARCHAR,              -- Содержимое поста (может быть NULL)
    user_id INTEGER REFERENCES users(id) -- Внешний ключ на таблицу users
);
```

## Примеры

Для примеров мы будем использовать следующую модель:

`User`: пользователи системы.

`Post`: записи, созданные пользователями.

Связи:
Один пользователь (User) может создавать много записей (Post) — связь 1:N (один ко многим)

### Python

Зависимости: `psycopg2-binary` `sqlalchemy`

```python
from sqlalchemy import create_engine, Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship, declarative_base, sessionmaker

Base = declarative_base()

# Модель User
class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    posts = relationship("Post", back_populates="user")

# Модель Post
class Post(Base):
    __tablename__ = "posts"
    id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    content = Column(String)
    user_id = Column(Integer, ForeignKey("users.id"))
    user = relationship("User", back_populates="posts")

# Подключение к базе и создание таблиц
engine = create_engine("postgresql+psycopg2://username:password@localhost/mydb")
Base.metadata.create_all(engine)

# CRUD-операции
Session = sessionmaker(bind=engine)
session = Session()

# Create
user = User(name="Alice")
post = Post(title="Hello, World!", content="My first post.", user=user)
session.add(user)
session.add(post)
session.commit()

# Read
user_from_db = session.query(User).filter_by(name="Alice").first()
print(user_from_db.name, user_from_db.posts[0].title)

# Update
user_from_db.name = "Alice Updated"
session.commit()

# Delete
session.delete(user_from_db)
session.commit()
```

### Java

В Java как правило используют JPA, но для него требуется дополнительная
конфигурация.

Для начала нужны библиотеки `org.postgresql:postgresql:42.7.4`
и `org.springframework.boot:spring-boot-starter-data-jpa:3.4.0`

Весь проект необходимо завернуть в пакет (в моём случае - `main`)

Файл `User.java`:
```java
package main;

import jakarta.persistence.*;

import java.util.List;

@Entity(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Post> posts;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List<Post> getPosts() {
        return posts;
    }

    public void setPosts(List<Post> posts) {
        this.posts = posts;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", posts=" + posts +
                '}';
    }
}
```

Файл `Post.java`:
```java
package main;

import jakarta.persistence.*;

@Entity(name = "posts")
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String content;

    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }

    @Override
    public String toString() {
        return "Post{" +
            "id=" + id +
            ", title='" + title + '\'' +
            ", content='" + content + '\'' +
            '}';
    }
}
```

Файл `UserRepository.java`:
```java
package main;

import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {}
```

Файл `UserService.java`
```java
package main;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public User create(String name, String title, String content) {
        User user = new User();
        user.setName(name);
        Post post = new Post();
        post.setTitle(title);
        post.setContent(content);
        post.setUser(user);
        user.setPosts(List.of(post));
        return userRepository.save(user);
    }

    public User findById(long id) {
        return userRepository.findById(id).orElse(null);
    }

    public User updateName(long id, String name) {
        User userFromDb = userRepository.findById(id).orElseThrow();
        userFromDb.setName(name);
        return userRepository.save(userFromDb);
    }

    public void delete(long id) {
        User userFromDb = userRepository.findById(id).orElseThrow();
        userRepository.delete(userFromDb);
    }
}
```

Файл `AppConfiguration.java`:
```java
package main;

import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class AppConfiguration {
    @Bean
    public DataSource dataSource() {
        return DataSourceBuilder.create().url("jdbc:postgresql://localhost:5432/mydb?user=username&password=password").build();
    }
}
```

Файл `Main.java`:
```java
package main;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Main implements CommandLineRunner {
    private final UserService userService;

    public Main(UserService userService) {
        this.userService = userService;
    }

    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }

    @Override
    public void run(String... args) throws Exception {
        User user = userService.create("Alex", "Title", "Test");
        System.out.println(user);
    }
}
```

### GO
Скачаем драйвер постгрес:
```bash
go get -u gorm.io/gorm gorm.io/driver/postgres
```

```go
package main

import (
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

type User struct {
	ID    uint `gorm:"primaryKey"`
	Name  string
	Posts []Post `gorm:"foreignKey:UserID"`
}

type Post struct {
	ID      uint `gorm:"primaryKey"`
	Title   string
	Content string
	UserID  uint
}

func main() {
	dsn := "host=localhost user=username password=password dbname=mydb port=5432 sslmode=disable"
	db, _ := gorm.Open(postgres.Open(dsn), &gorm.Config{})

	db.AutoMigrate(&User{}, &Post{})

	// Create
	user := User{Name: "Alice", Posts: []Post{
		{Title: "Hello, World!", Content: "My first post."},
	}}
	db.Create(&user)

	// Read
	var userFromDb User
	db.Preload("Posts").First(&userFromDb, "name = ?", "Alice")
	println(userFromDb.Name)

	// Update
	db.Model(&userFromDb).Update("Name", "Alice Updated")

	// Delete
	db.Delete(&userFromDb.Posts)
	db.Delete(&userFromDb)
}
```

(как же хорошо после джавы на го писать, просто вау)

### C#

Необходимо установить `Npgsql.EntityFrameworkCore.PostgreSQL` и `Microsoft.EntityFrameworkCore`

```csharp
using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;

namespace ConsoleApp1;

[Table("users")]
public class User {
    [Column("id")]
    public int Id { get; set; }
    [Column("name")]
    public string Name { get; set; }
    [Column("password")]
    public List<Post> Posts { get; set; } = new();
}

[Table("posts")]
public class Post {
    [Column("id")]
    public int Id { get; set; }
    [Column("title")]
    public string Title { get; set; }
    [Column("content")]
    public string Content { get; set; }
    [Column("user_id")]
    public int UserId { get; set; }
    public User User { get; set; }
}

// Контекст базы данных
public class AppDbContext : DbContext {
    public DbSet<User> Users { get; set; }
    public DbSet<Post> Posts { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder) {
        // Укажите свои данные для подключения
        optionsBuilder.UseNpgsql("Host=localhost;Database=mydb;Username=username;Password=password");
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder) {
        // Дополнительная настройка сущностей
        modelBuilder.Entity<Post>()
            .HasOne(p => p.User)
            .WithMany(u => u.Posts)
            .HasForeignKey(p => p.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}

public class Program {
    public static void Main(string[] args)
    {
        // Инициализация базы данных
        using var context = new AppDbContext();
        // Создаём базу данных, если её нет
        context.Database.EnsureCreated();

        // --- CRUD Операции ---
        // 1. Create (Добавление данных)
        Console.WriteLine("=== CREATE ===");
        var user = new User { Name = "Alice" };
        var post1 = new Post { Title = "Hello World", Content = "My first post!", User = user };
        var post2 = new Post { Title = "Another Post", Content = "This is another post.", User = user };
        context.Users.Add(user);
        context.Posts.AddRange(post1, post2);
        context.SaveChanges();
        Console.WriteLine("User and posts created!");

        // 2. Read (Чтение данных)
        Console.WriteLine("\n=== READ ===");
        var users = context.Users.Include(u => u.Posts).ToList();
        foreach (var u in users) {
            Console.WriteLine($"User: {u.Name}");
            foreach (var p in u.Posts) {
                Console.WriteLine($"  Post: {p.Title} - {p.Content}");
            }
        }

        // 3. Update (Обновление данных)
        Console.WriteLine("\n=== UPDATE ===");
        var userToUpdate = context.Users.FirstOrDefault(u => u.Name == "Alice");
        if (userToUpdate != null) {
            userToUpdate.Name = "Alice Updated";
            context.SaveChanges();
            Console.WriteLine("User updated!");
        }

        // 4. Delete (Удаление данных)
        Console.WriteLine("\n=== DELETE ===");
        var userToDelete = context.Users.Include(u => u.Posts).FirstOrDefault(u => u.Name == "Alice Updated");
        if (userToDelete != null) {
            context.Users.Remove(userToDelete);
            context.SaveChanges();
            Console.WriteLine("User and their posts deleted!");
        }
    }
}
```
