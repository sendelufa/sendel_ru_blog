---
title: "Hibernate Faq"
date: 2022-07-26T00:08:27+05:00
description: 
author: "Konstantin Shibkov"
tags: []
categories: []
draft: true
---

## FAQ по Hibernate

### Конфигурация

#### Как указать ассоциировать класс с таблицей

Использовать аннотацию `@Entity` над классом и зарегистрировать в
метаданных Hibernate.

Класс должен иметь пустой конструктор, поле с первичным ключом
и все поля должны иметь геттеры и сеттеры.

```java
@Entity
public class Person{

    // обязательно должен быть пустой конструктор
    public Person(){}

    // иметь первичный ключ
    @Id
    private long id;

    // ни один из полей не должны быть final
    private String name;

    // сеттеры и геттеры
}
```

#### Как зарегистрировать @Entity класс в проекте?

Добавить через java конфиг:

```java
Configuration conf = new Configuration();
conf.addAnnotatedClass(Person.class);
conf.configure();
```

или через XML конфиг:

```xml
<session-factory>
     <mapping class="ru.sendel.model.Person"/>
</session-factory>
```

#### Показать в логе форматированный SQL запросы hibernate

```xml
<session-factory>
     <property name="show_sql">true</property>
     <property name="format_sql">true</property>
</session-factory>
```

#### Стратегия маппинга имен переменных Java в SQL

для автоматического нейминга (CamelCase -> SnakeCase)

```java
Configuration conf = new Configuration();
conf.setPhysicalNamingStrategy(new CamelCaseToUnderscoresNamingStrategy());
conf.configure();
```

### Entity

#### Важные аннотации

- `@Entity` - класс является проекцией на таблицу SQL.
- `@Table(name = "users", schema = "public")` -
над классом, указанием имени таблицы и схемы.
- `@Id` - надо полем с primary key.
- `@Column(name = "update_date")` - над полем, для указания колонки с данными, возможны параметры:
  - boolean `nullable` - может быть значение NULL.
  - int `length` - длина, например VARCHAR.
  - boolean `unique` - уникальное значение во всей таблице.

#### Конвертер кастомных классов полей @Entity в известные SQL типы

Например, нам надо хранить класс даты дня рождения и
он должен в SQL представлять тип `java.sql.Date`.

Создать класс и имплементировать
`AttributeConverter<Birthday, Date>`:

```java
public class BirthdayConverter implements AttributeConverter<Birthday, Date> {
    @Override
    public Date convertToDatabaseColumn(Birthday attribute){
        return Optional.ofNullable(attribute)
            .map(Birthday::birthday) // LocalDate поле
            .map(Date::valueOf)
            .orElse(null);
    }

    @Override
    public Birthday convertToEntityAttribute(Date dbData) {
        return Optional.ofNullable(dbData)
            .map(Date::toLocalDate)
            .map(Birthday::new) // конструктор с LocalDate
            .orElse(null);
    }
}
```

Добавить использование конвертера для выбранного поля

- Вариант 1, установить конвертер для каждого поля отдельно

Над полем в классе `@Entity`

```java
@Converter(converter = BirthdayConverter.class)
private Birthday birthday;
```

- Вариант 2, для всех полей с типов в конвертере

```java
Configuration conf = new Configuration();
// второй аргумент true -> авто применение конвертера
conf.addAttributeConverter(new BirthdayConverter(), true);
conf.configure();
```

Авто применение (auto apply), можно также установить
с помощью аннотации, в конфигурации остается только объект
конвертера:

```java
Configuration conf = new Configuration();
// второй аргумент true -> авто применение конвертера
conf.addAttributeConverter(new BirthdayConverter());
conf.configure();
```

а над классом конвертера ставится аннотация:

```java
import javax.persistence.Converter;

@Converter(autoApply = true)
public class BirthdayConverter implements AttributeConverter<Birthday, Date> {
```

#### Создание кастомных SQL типов

Создайте класс и имплементируйте `UserType`, переопределите
все его методы.

Для облегчения работы по использованию кастомных типов для
PostgreSQL, на помощь придет библиотека с большим набором
готовых типов [Hibernate Types 52](https://mvnrepository.com/artifact/com.vladmihalcea/hibernate-types-52)

Например, для хранения JSON как бинарного набора, используйте:

```java
@Type(type = "com.vladmihalcea.hibernate.type.json.JsonBinaryType")
private String data;
```

Необходимо указать полный путь до класса или использовать
значение name, которое возвращает класс типа, для `JsonBinaryType` это будет:

```java
@Type(type = "jsonb")
private String data;
```

Можно определять своем имя для типа, смотри [@TypeDef](https://docs.jboss.org/hibernate/orm/5.4/javadocs/org/hibernate/annotations/TypeDef.html)

И далее зарегистрировать в конфиге hibernate новый тип:

```java
Configuration conf = new Configuration();
conf.registerTypeOverride(new JsonBinaryType());
conf.configure();
```

#### Маппинг Enum типов

Для маппинга поля Enum значения в ячейку колонки используется
две стратегии Ordinal и String. Ordinal записывает значение индекса INT Enum значения в порядке объявления в Java, а String берет строковое значение, например для
VARCHAR.

> По умолчанию используется ORDINAL.

Для использования строкового значения:

```java
@Enumerated(EnumType.STRING)
private OrderType type;
```

### Жизненный цикл Entity

- **Transient** - объект не связан ни с одной сессией

Такое состояние принимает при создании нового объекта @Entity через конструктор или после удаления
объекта из бд. Объект не связан с Session. Изменение
объекта не изменяет состояние БД.

- **Persistent** - состояние получает объект как только ассоциируется с сессией.

Ассоциация с сессией происходит,например, когда
выполняются методы `save(T)` или `persist(T)`.
В таком состоянии изменения объекта изменяют запись
в бд. При этом фиксация изменений в бд происходит
только в случаи коммита транзакции или выполнения
очистки (flush) сессии.

Методы Session, которые создают или переводят объект в
Persistent состояние:

- `save(T)` - сохранить новое Entity в бд, возвращает
сгенерированный ключ объекта. Если объект был в
состоянии Transient или Detached, то save переведет
в Persistence и получит новый ключ
и создать новую запись в БД.
- `persist(T)` - этот метод частично похож на save.
Переводит из состояние Transient в Persistent.
Ничего не возвращает. Если попытаться применить метод
к объекту в состоянии Detached - будет выброшено
исключение, то есть дублирующего сохранения не получится.
- `update(T)` - обновляет объект, измененный в состоянии
Detached.
- `saveOrUpdate(T)` - если состояние объекта Transient,
то получает id и создает новый объект. Если состояние
объекта Detached, то обновляет объект в бд.
- `merge(T)` - создает Persistence объект из
существующего объекта. Получает объект по id из Persist
Context или, если там не найден, берет объект из бд.
Копирует все поля с новый объект и возвращает объект.

- **Detached** - состояние при котором объект отключен
(отсоединен) от сессии. Объект может перейти в такое
состояние только из Persistent, а из состояние Detached
возможно вернуться в Persistent.

Методы для перехода в Detached:

- `detach(T)` - 

