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
